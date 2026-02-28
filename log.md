# Registro de Cambios (Changelog)

**Sugerencia de escritura:** Priorizar la acción y el valor aportado sobre el nombre del archivo para una lectura más fluida del progreso. Sigue los ejemplos anteriores.

## 2026-02-25 - Inicio del Proyecto

- Sesión inicial con Claude para definir la primera versión del Brief.
- Creación del documento principal con descripción, arquitectura y reglas del Agente.
- Integración y traducción del Brief v0.1 (visión y especificaciones).
- Primera sesión de planificación de especificaciones técnicas y ejecución.
- Definición de arquitectura, flujo de datos y capas del sistema.
- Sustitución de diagramas ASCII por Mermaid en arquitectura.
- Inclusión de sección "Completitud Inteligente" con ejemplos prácticos.
- Actualización de arquitectura con justificación de WebSockets y versionado.
- Definición de especificaciones técnicas: interfaces, payloads y stack.
- Refinamiento de especificaciones: configuración extensible, prompts externos y TDD.
- Diseño del Roadmap en 6 fases con enfoque TDD y criterios de aceptación.
- Cierre de sesión con registro de tiempos e introspección.
- Migración del stack de Backend a Python/FastAPI.
- Actualización de reglas de trabajo y estructura general.

## 2026-02-26 - Reglas MaC

- Reformulación de reglas de agente IA, ahora son reglas [MaC](MaC.md) (Management as Code) pensadas para ser genéricas.
- Diseño de sistema de dos modos de operación (Consultivo/Proactivo) para resolver contradicción en las reglas originales.
- Radar de contexto: mecanismo para que el agente señale información relevante que se pasa por alto.
- Regla de coherencia documental para propagar cambios entre documentos.
- Separación de reglas técnicas en `docs/technical-guidelines.md`, fuera del MaC.
- Creados `AGENTS.md`, `session-TEMPLATE.md`, `MaC.md`.
- Sección Contributing agregada al README.

## 2026-02-27 - Configuración de Agente e Investigación

- Configuración de carga automática de `AGENTS.md` mediante `.agent/instructions.md`.
- Investigación de la API de Gemini Live Streaming en `docs/investigacion/Arquitectura de Streaming Speech-to-Text Gemini.md`
- Revisión documental profunda de Arquitectura e EETT (v2): adoptado `google-genai` sobre Python, Web Audio API remuestreando a PCM 16kHz Base64, e iteración a una interfaz de UX híbrida (callbacks asíncronos para Feedback Acústico y Contenido Inteligente).
- Guía técnica `docs/technical-guidelines.md` actualizada con advertencias estrictas sobre manejo de credenciales Git y resolución de layouts Mermaid/Dagre.
- Cierre de sesión y reflexión de PM: el valor de "deep research" para descartar supuestas arquitecturas previo a planificar (y usar consultas abiertas de IA).