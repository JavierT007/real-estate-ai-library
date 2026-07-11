---
id: SOL-001
tipo: lessons
---

# Lecciones aprendidas — G Inversiones

## Errores y soluciones
- Make + Google Calendar: createAnEvent requiere objetos Date nativos de Make, no strings de Supabase
- Supabase upsertARecord: requiere TODAS las columnas NOT NULL
- Slack en Make: mensajes de bot necesitan conexión con bot token separada, no OAuth estándar
- Make WatchMessages: el cursor solo avanza en ejecuciones programadas, no con Run once
- Gemini free tier: rate limit 20 req/min si dos escenarios corren simultáneos
- useMemo sin dependencia properties provocaba catálogo vacío
- RLS bloqueaba lecturas anon: crear política explícita para SELECT público
- Cambios de schema Supabase: siempre SQL manual con add column if not exists
- URL de redirect de reset password debe estar whitelisted en Supabase Auth

## Qué reutilizar
- Conversión de unidades de terreno (Manzanas 6988.96 m2, Acres 4046.86 m2)
- Flujo de foto principal reordenada a índice 0
- resetPasswordForEmail() con ruta dedicada

## Qué NO volver a hacer
- Asumir que gemini-2.0-flash sigue disponible (descontinuado, usar 2.5-flash)
- Correr escenarios Make con IA free tier en paralelo
