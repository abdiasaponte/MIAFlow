# CLAUDE.md — MIAFlow

> Este documento es la fuente de verdad para Claude Code en este repositorio. Es la versión equivalente a las instrucciones del Proyecto "MIAFlow" en Claude (Projects/Cowork). Antes de proponer arquitectura nueva, revisar la sección 4 (estado actual) y preguntar si algo cambió.

## 1. Contexto

MIAFlow es un MVP de automatización de procesos docentes. Conecta aulas virtuales (Google Classroom, Microsoft Teams, Moodle, WhatsApp/Telegram) con un flujo construido en **n8n** que recibe tareas entregadas por estudiantes, las corrige con un LLM, genera calificaciones y reportes de retroalimentación, los almacena en Drive/OneDrive, y produce reportes de rendimiento periódicos. Todo el desarrollo se documenta en **Notion**.

## 2. Objetivo del MVP

Automatizar el ciclo completo de una tarea escolar: entrega → corrección → calificación → retroalimentación → almacenamiento → reporte periódico, reduciendo trabajo manual del docente y dando trazabilidad al aprendizaje real del estudiante.

## 3. Stack técnico

- **Orquestación:** n8n self-hosted local, corriendo en **Docker** (`docker-compose.yml` en la raíz del repo) desde el 2026-07-02. Antes corría instalado directo vía `npm` — se migró a Docker para evitar problemas de permisos/caché de npm y aislar el entorno. Los datos (workflows, credenciales) viven en `~/.n8n`, montado como volumen dentro del contenedor — no se perdió nada en la migración. Levantar con `docker compose up -d` desde la raíz del repo; editor accesible en `http://localhost:5678`.
- **LLM de corrección/generación:** ChatGPT u otro LLM configurable vía API (diseño agnóstico al proveedor cuando sea posible).
- **Almacenamiento:** Google Drive u OneDrive (carpeta por estudiante, dentro de cada curso).
- **Calificaciones:** XLSX (nombre de la tarea + columnas: Nombre completo, Calificación).
- **Retroalimentación:** PDF o Docx, máx. 1 página, formato fijo (ver sección 5).
- **Documentación:** Notion.
- **Entorno de desarrollo:** VS Code + Claude Code, con MCP hacia n8n, Supabase, Notion (y Drive/OneDrive/Flowise si exponen MCP).
- **Flowise ("Missi"):** chatbot/agente para docentes y/o estudiantes. Canal de interacción, no reemplaza la lógica de corrección en n8n. Alcance exacto aún por confirmar — no asumir funciones sin verificar en Notion primero.
- **Supabase:** base de datos relacional + vector store/RAG + autenticación, a la vez. Antes de proponer tablas/esquema nuevo, inspeccionar el esquema real vía MCP de Supabase.

## 4. Estado actual del proyecto

> ⚠️ Esta sección puede estar desactualizada respecto a Notion — Notion es la fuente viva. Verificar "Roadmap Maestro" antes de asumir nada.

**Última verificación en Notion: 2026-07-02.**

- **Infraestructura:** repo git inicializado (2026-07-02), `CLAUDE.md` en la raíz, Claude Code instalado y funcionando sobre este repo, MCP conectados (Notion, n8n, Supabase). n8n migrado de instalación npm a Docker el mismo día (ver sección 3). Workflow "Flujo 1.4 (Repor. Retroalim.)" exportado y versionado en `n8n-workflows/`.
- **n8n:** el Roadmap Maestro muestra la etapa **"1. Motor de Automatización (MVP)"** en estado **En Proceso** desde 2026-06-18 (fin estimado 2026-08-02), prioridad Alta. Resultado esperado: "Automatizar evaluación, feedback, PDF y calificación". Según el checklist "Cerrar bien el MVP actual" en Notion: completado (entrega recibida, extracción de Classroom, cálculo del 60% de trabajo entregado); en proceso (separar la nota en dos partes, prueba diagnóstica del 40%, crear carpeta del estudiante en su curso); pendiente (XLSX para IDoceo, carpeta de reporte de calificaciones, reporte de retroalimentación en HTML). Esto contradice el estado "apenas etapa inicial" registrado antes en las instrucciones del Proyecto — el MVP está considerablemente más avanzado de lo que ese documento sugería.
- **Notion:** estructura ya definida. Página raíz "MIAFlow" → "Gestión del Proyecto" → "Roadmap Maestro" (database), más páginas "N8N", "Flowise/Missi", "Frontend", "DashBoard". Bases relacionadas: "Módulos del Proyecto", "Registro de Sesiones N8N", "Registro de Sesiones Flowise/Missi", "Registro de Sesiones Documental", "Estructura Drive", "Configuración Nodos N8N". No crear páginas/bases nuevas sin revisar esta estructura primero.
- **Flowise/Missi:** implementado como chatbot/agente. Función específica (qué consultas resuelve, con qué fuentes, si tiene RAG conectado) aún no completamente definida en las instrucciones del proyecto — tratar como componente aparte del flujo de corrección de n8n.
- **Supabase:** en uso como BD relacional + RAG + auth simultáneamente. No se ha detallado el esquema exacto — preguntar/inspeccionar antes de proponer tablas o políticas RLS.

## 5. Rol de Claude Code en este repo

- Actuar como arquitecto/desarrollador de automatizaciones: workflows de n8n (nodos, lógica condicional, webhooks, credenciales) y prompts de LLM robustos.
- La lógica de negocio de este documento es la fuente de verdad; si una petición la contradice, señalarlo antes de implementar.
- Priorizar soluciones gratuitas o de bajo costo.
- Generar/exportar workflows de n8n como JSON versionado en `n8n-workflows/`.
- Prompts de corrección auditables: rúbrica explícita, criterios claros, salida estructurada en JSON para que n8n la parsee.
- Registrar avances en Notion respetando la estructura ya existente (sección 4) — leer antes de escribir.
- Ser objetivo, ir al grano, no alucinar estado del proyecto — verificar en Notion o preguntar.

## 6. Reglas de negocio clave

### 6.1 Fechas de entrega y penalización por atraso
- 1ra entrega: hasta 3 días después de asignada. Rúbrica sobre 100 puntos.
- 2da entrega: hasta 7 días después de vencida la 1ra. Máximo 90 puntos.
- 3ra entrega: hasta 7 días después de vencida la 2da. Máximo 80 puntos.
- Fuera de cada ventana, el flujo no procesa nuevas entregas de esa tarea hasta nuevo aviso del docente.

### 6.2 Prueba diagnóstica automática (40% de la nota final)
- Quiz de opción múltiple (5–10 preguntas) generado al entregar, basado en el contenido investigado por el estudiante.
- Tiempo límite: 10 min totales (1 min/pregunta máx.), un solo intento.
- Ponderación: 60% evaluación del trabajo + 40% resultado del quiz.

### 6.3 Formatos de salida por tarea corregida
1. **XLSX de calificación:** nombre = nombre de la tarea. Columnas: Nombre completo del estudiante | Calificación.
2. **Reporte de Retroalimentación (PDF/Docx, máx. 1 página):** logo institución centrado; nombre institución (negro, negrita, 15, centrado); título "Reporte de Retroalimentación" (negro, negrita, subrayado, 14, centrado); estudiante + curso (negrita, 12); tarea + materia (negrita, 12); docente (negrita, 12); Calificación / Fortalezas / Aspectos a Mejorar / Retroalimentación (etiqueta negrita + texto normal); todo tamaño 12 salvo lo indicado.

### 6.4 Almacenamiento
- Al generar el reporte, crear (si no existe) carpeta con el nombre del estudiante dentro del curso en Drive/OneDrive. Idempotente — no duplicar carpetas.

### 6.5 Reportes periódicos
- Trimestral: LLM combina reportes acumulados → "Reporte de Rendimiento de Período" por estudiante.
- Asistencia: generada a partir de datos cargados por el docente (IDoceo/Additio u otra plataforma).
- Reporte de Período: calificaciones + asistencia + rendimiento consolidados.

### 6.6 Planificación docente (fase futura, prioritaria)
- Automatizar planificación diaria y por proyectos; generación de prácticas, exámenes, cuestionarios y actividades alineadas a las planificaciones.
- Base de datos futura: plantilla oficial MINERD vigente, malla curricular, Bases del Currículo Dominicano, calendario escolar.

## 7. Convenciones de trabajo

- Workflows de n8n como JSON versionado en `n8n-workflows/`.
- Prompts de LLM en `prompts/` (subcarpetas: `correccion/`, `quiz/`, `reportes/`), no hardcodeados en el JSON del workflow cuando sea evitable.
- Credenciales y llaves de API: nunca en código ni en archivos versionados — usar `.env` (no versionado, ver `.env.example`) o el gestor de credenciales de n8n/Supabase.
- Antes de modificar un workflow o esquema existente: explicar qué nodo(s)/tabla(s) afecta y por qué.
- Operaciones idempotentes vía MCP (no duplicar carpetas, tablas o páginas de Notion — "solo crear si no existe").

## 8. Servidores MCP

Conectados o en plan: **Supabase, n8n, Notion** (posiblemente Drive/OneDrive o Flowise si exponen MCP).

- **Supabase:** inspeccionar esquema real antes de proponer tablas; generar/aplicar migraciones SQL en `supabase/migrations/`; escribir políticas RLS.
- **n8n:** leer/exportar workflows existentes antes de modificarlos; validar que los cambios no rompan las reglas de negocio (sección 6) antes de desplegar.
- **Notion:** registrar avances en la estructura ya existente (sección 4) — leer antes de escribir, nunca asumir jerarquía propia.

---
*Generado a partir de las instrucciones del Proyecto MIAFlow (Claude/Cowork) el 2026-07-02. Mantener sincronizado con ese documento — si uno cambia, actualizar el otro.*
