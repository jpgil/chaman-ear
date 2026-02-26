# Chaman-ear

> Voz a texto en streaming con interpretación de intención.

## Descripción del Proyecto

**Chaman-ear** es un SDK de código abierto que habilita el "dictado inteligente" en tiempo real: el usuario habla de forma natural —con dudas y muletillas— y obtiene un párrafo limpio y bien estructurado que preserva su intención original, sin transcripciones literales ruidosas.

## Arquitectura Esperada

- **API de Backend (Python/FastAPI)**: Utiliza la API de **Google Gemini Live** para procesar audio nativo y devolver eventos estructurados en streaming (`partial` y `final`) evaluando la semántica.
- **SDK JavaScript (NPM)**: Biblioteca cliente que maneja la grabación web (`MediaRecorder`) y los WebSockets, exponiendo métodos simples (`onPartial`, `onFinal`, `start`, `stop`).
- **Aplicación de Demostración**: Un cliente web minimalista que muestra la experiencia implementando el motor y su sistema de contexto a medida (Instrucción de formato + Contexto de Background).

---

## Reglas para el Agente de IA

Este documento define las directrices y convenciones de desarrollo para este repositorio:

1. **Registro de Cambios**: Documentar cualquier modificación significativa en el archivo `log.md`.
2. **Actualización del Stack**: Mantener `technical_stack.md` al día, registrando inmediatamente cualquier nueva tecnología o herramienta incorporada.
3. **Autonomía y Velocidad**: 
   * El agente tiene autonomía total para tomar decisiones de diseño sin consulta previa, priorizando la agilidad y el avance rápido del proyecto.
   * **Implementación en dos pasos**: Si el usuario pide un cambio o mejora, el agente primero explica la estrategia y la registra en el archivo de sesión (`planning/`). Solo tras la confirmación del usuario o en un prompt posterior, el agente procede a implementar lo que esté marcado como incompleto.
4. **Ritual de Inicio de Sesión**: Al comenzar cualquier sesión, el agente DEBE seguir este orden antes de tomar cualquier acción:
   1. Leer `log.md` para obtener contexto del historial de cambios del proyecto.
   2. Leer las **Deudas abiertas** del archivo `session-*.md` más reciente y usarlas como punto de partida. Si una deuda ya fue resuelta, marcarla como completada.
   3. Leer `planning/roadmap.md` para entender el estado actual de las fases y la planificación a alto nivel.
5. **Planificación**:
   * Cualquier implementación requiere que el plan o mejora ya esté descrita en la sesión activa.
   * El agente debe revisar si hay un plan activo en `planning/` y continuar desde el último checklist no completado. Si no existe, debe crearlo con nombre `planning/session-YYYY-MM-DD.md`. Buscar en los últimos tres archivos de planificación (orden cronológico).
6. **Unit Tests**: Todo código Python del backend debe tener tests unitarios con `pytest` en el directorio `tests/`. Los tests deben ejecutarse y pasar antes de dar una fase por completada.
7. **Documentación**: Mantener documentación actualizada en `docs/`. Agregar docstrings a funciones Python y JSDoc a funciones JavaScript. Cada componente nuevo debe documentarse antes de pasar a la siguiente fase.
8. **Bugfixes y Mejoras**: Todo bugfix o feature que surja fuera del plan original debe registrarse en el `session-YYYY-MM-DD.md` activo bajo las secciones `## Bugfixes` y `## Mejoras`, con síntoma, causa, fix e ítems completados.
9. **Cambios de Documentación y Reglas**: Las actualizaciones de documentación (`docs/`, `README.md`, `technical_stack.md`, etc.) y de reglas del agente **no** requieren entrada en `planning/`. Solo deben registrarse en `log.md`.
10. **Cierre de Sesión con Introspección**: Cuando el usuario indique explícitamente el cierre de la sesión, se gatilla la escritura de la sección `## Introspección de la Sesión` en el archivo `session-*.md` activo. Este proceso sigue un flujo interactivo estricto:
    - El agente genera de inmediato la estructura base (TL;DR, decisiones, sorpresas, aprendizajes, métricas, reflexiones PM, deudas abiertas).
    - Una vez que la base está lista, el agente da paso al usuario (PM) para que ingrese sus reflexiones personales.
    - **IMPORTANTE:** El usuario puede enviar múltiples prompts con reflexiones. Estas reflexiones **no deben gatillar acciones técnicas ni escrituras prematuras**. El agente simplemente acusará recibo y esperará a que el usuario indique que ha terminado.
    - Solo cuando el usuario devuelva formalmente el control para finalizar, el agente tomará todas esas reflexiones, las redactará en la sección `**Reflexiones del PM**` de la bitácora, y a partir de todo ese contexto final, inferirá y escribirá la sección de `**Deudas abiertas**` cerrando definitivamente el documento.

    Las subsecciones de la introspección son:
    - **TL;DR**: Métricas duras y la idea central de la sesión en 2-3 líneas.
    - **Cadena de decisiones**: Diagrama o lista de las macro-decisiones y sus derivaciones (bugs, mejoras, meta-mejoras).
    - **Micro-decisiones clave**: Tabla con las decisiones pequeñas que resultaron determinantes (contexto → impacto).
    - **Sorpresas**: Qué supuestos se invalidaron y por qué, con contexto suficiente para entender la causa.
    - **Aprendizajes**: 1-3 lecciones con *implicancia* explícita (qué cambiar en el futuro).
    - **Métricas**: Tabla con dimensiones clave (archivos, líneas, tests, bugs, mejoras, reglas).
    - **Reflexiones del PM**: Observaciones aportadas por el humano de forma interactiva.
    - **Deudas abiertas**: Checkboxes con problemas identificados pero no resueltos. **Esta subsección DEBE ir invariablemente al final absoluto de la introspección**, alimentándose de las reflexiones del PM.
    
    **Tono**: compacto pero con desarrollo suficiente para que un humano o IA pueda extrapolar las ideas rápidamente y sin dificultad. Evitar tanto la verbosidad excesiva como la compresión críptica.
11. **Nomenclatura Estándar**: Bugfixes se identifican como `BF-XX` y mejoras como `M-XX` (numeración secuencial por sesión). Esta nomenclatura debe usarse consistentemente en `session-*.md`, `log.md` y commits.
12. **Horario de Sesión**: Todo archivo `session-*.md` debe registrar en su encabezado la **hora de inicio** y la **hora de término** de la sesión (formato `HH:MM`, zona horaria local). La hora de inicio se registra al crear el archivo; la hora de término se actualiza al cerrar la sesión (antes de la introspección).