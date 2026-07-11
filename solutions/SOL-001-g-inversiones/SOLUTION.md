---
id: SOL-001
nombre: G Inversiones
tipo: website
industria: inmobiliaria
stack: [react, vite, tailwind, supabase, vercel, make, pipedream]
estado: activo
usado-en: []
recursos: []
---

# G Inversiones

## Problema
Inmobiliaria en San Miguel, El Salvador, necesitaba plataforma web para publicar propiedades, captar leads y automatizar agendamiento de citas.

## Solución
Web app React con catálogo filtrable, portal admin, i18n ES/EN, captura de leads a Supabase, y automatización Make (Gmail/Slack a Gemini a Google Calendar).

## Arquitectura
- Frontend: React + Vite + Tailwind, routing por pathname (sin React Router)
- Backend: Supabase (auth, RLS, tabla leads, storage de imágenes)
- Deploy: Vercel vía GitHub
- Automatización: Make (2 escenarios) + Pipedream (leads a Slack)
- IA: gemini-2.5-flash para agendamiento

## Componentes reutilizables
- PropertyCard.jsx (diseño Dribbble, colores de marca)
- TranslatedText (traducción automática vía MyMemory API)
- Location picker con Leaflet + Nominatim
- Compresión de imágenes con browser-image-compression
- Sistema i18n con localStorage
