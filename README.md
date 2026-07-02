# MIAFlow

Repositorio del MVP de automatización de procesos docentes. Ver `CLAUDE.md` para contexto completo (arquitectura, reglas de negocio, estado del proyecto).

## Estructura

- `n8n-workflows/` — exports JSON de cada workflow de n8n.
- `flowise-flows/` — exports JSON de los flujos del chatbot (Missi).
- `supabase/migrations/` — migraciones SQL, esquema, políticas RLS.
- `prompts/` — prompts del LLM (corrección, quiz, reportes), versionados aparte del JSON del workflow.
- `.env.example` — variables de entorno necesarias (sin valores reales).

## Documentación

El avance del proyecto se documenta en Notion (espacio "MIAFlow" → "Roadmap Maestro").
