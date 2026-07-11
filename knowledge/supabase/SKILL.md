---
name: supabase
description: Backend con Supabase - auth, RLS, storage, tablas. Usar cuando el proyecto necesite base de datos, autenticacion, o falle acceso a datos (RLS, anon key, policies).
---

# Supabase

## Patrones probados
- Captura de leads: tabla `leads` + politica RLS que permite INSERT anon pero no SELECT
- Reset de password: `resetPasswordForEmail()` + ruta dedicada; la URL de redirect DEBE estar whitelisted en Auth settings
- Storage de imagenes: comprimir en cliente antes de subir (browser-image-compression)

## Errores comunes y soluciones
- "No aparecen datos" con anon key: falta politica RLS explicita para SELECT publico
- upsert falla: faltan columnas NOT NULL en el payload
- timestamptz con fracciones de segundo rompe validadores externos (ej. Make): convertir a Date nativo en destino
- Cambios de schema: siempre SQL manual con `alter table ... add column if not exists`, nunca confiar en el editor visual

## Snippets clave
-- Politica de lectura publica
create policy "public read" on propiedades for select using (true);

-- Politica insert anon para leads
create policy "anon insert" on leads for insert with check (true);

## Recursos relacionados
- Proyecto de referencia: SOL-001 (G Inversiones)
