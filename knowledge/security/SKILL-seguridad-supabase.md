---
name: seguridad-supabase
description: Seguridad en apps con Supabase + frontend. Usar al auditar RLS, roles, auth, o antes de lanzar cualquier sitio con Supabase a producción. Previene fuga de datos y escalación de privilegios.
usado-en: [SOL-001]
---

# Seguridad Supabase

## Principio central (el error que todos cometen)
El control de acceso en el FRONTEND (ej. `if (user.role === 'admin')`) solo
esconde botones. NO protege datos. La anon key es pública y cualquiera puede
hablar directo con Supabase desde la consola del navegador.

**La única defensa real está en RLS + configuración de Auth.**

## Trampa crítica: "authenticated" no significa "staff"
El registro de Supabase es PÚBLICO por defecto. Cualquiera puede crearse una
cuenta y volverse `authenticated`. Por eso una política `to authenticated`
con `using (true)` NO es segura: la ve cualquier desconocido registrado.

### Síntomas de este problema
- Políticas RLS con rol `authenticated` y `qual: true` / `with_check: true`
- Roles de usuario (admin/staff) definidos solo en el código React
- Creación de usuarios vía `signUp` con el rol elegido en un formulario

### Riesgos concretos (caso SOL-001)
1. Leer TODOS los leads (datos privados de clientes) → fuga de datos
2. Insert/update en `profiles` → cualquiera se hace admin (escalación de privilegios)
3. DELETE abierto en tablas → borran todo el catálogo desde consola

## Solución probada (2 capas)

### Capa 1 — Cerrar el registro público (el candado principal)
Supabase → Authentication → Sign In / Providers → Email →
desactivar "Allow new users to sign up".
Efecto: "authenticated" pasa a significar "solo staff que YO creé".
Contra: el botón de crear usuario del portal deja de servir (ver abajo).

### Capa 2 — Políticas RLS por rol
```sql
-- Helper sin recursión para leer el rol del usuario actual
create or replace function public.mi_rol()
returns text language sql security definer stable
set search_path = public as $$
  select role from public.profiles where id = auth.uid();
$$;

-- Lectura pública SOLO donde debe (catálogo). Escritura/borrado por rol.
-- Ejemplo properties:
create policy "staff inserta propiedades" on properties for insert to authenticated
with check (public.mi_rol() in ('admin','ejecutivo'));
create policy "solo admin borra propiedades" on properties for delete to authenticated
using (public.mi_rol() = 'admin');

-- profiles: cada quien lee lo suyo, solo admin crea/edita
create policy "leer perfil propio o admin" on profiles for select to authenticated
using (id = auth.uid() or public.mi_rol() = 'admin');
create policy "solo admin crea perfiles" on profiles for insert to authenticated
with check (public.mi_rol() = 'admin');
```

## Crear staff sin el botón (con registro cerrado)
1. Authentication → Users → Add user (email + password, confirmar)
2. Copiar el UUID
3. `insert into profiles (id, name, email, role) values ('UUID','Nombre','correo','ejecutivo');`

## Verificar políticas actuales (query de auditoría)
```sql
select tablename, policyname, cmd, roles, qual, with_check
from pg_policies where schemaname = 'public'
order by tablename, cmd;
```
Bandera roja: rol `{authenticated}` con `qual: true` en tablas sensibles
(leads, profiles) o en operaciones DELETE/UPDATE.

## Otros pendientes de seguridad
- **Security headers** en vercel.json: X-Frame-Options, X-Content-Type-Options,
  HSTS, Referrer-Policy, Permissions-Policy (ver plantilla en la bóveda)
- **Storage**: validar MIME real y tamaño en el bucket, no solo la extensión
  del nombre de archivo en el cliente
- **No dejar archivos de prueba** (ej. test-supabase.js) en producción
- La anon/publishable key SÍ va en el frontend (es su diseño). La `service_role`
  key NUNCA toca el frontend.

## Relacionado
- SKILL supabase (patrones generales de RLS y storage)
- SOL-001 G Inversiones (auditoría de origen)
