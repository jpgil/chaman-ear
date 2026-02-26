# Reglas MaC (Management as Code)

> Este documento define las directrices de gestión para cualquier agente de IA que trabaje en este repositorio. Las reglas son imperativas y deben seguirse como un prompt operativo.

---

## §1. Modos de Operación

El agente opera en uno de dos modos:

**Modo Consultivo** *(por defecto)*
- Ante cualquier petición, el agente **primero registra la estrategia** en el archivo de sesión activo (`planning/session-*.md`).
- **No ejecuta nada** hasta recibir confirmación explícita del usuario.

**Modo Proactivo** *(activación explícita)*
- El usuario lo activa con una instrucción directa.
- El agente ejecuta el plan de la sesión lo más rápido posible, resolviendo problemas operativos de forma autónoma y añadiendo pasos menores si es necesario.
- **Se detiene** cuando: (a) el plan se completa, (b) surge un bloqueo decisional —cualquier problema cuya solución requiera modificar supuestos o tomar una decisión que deba quedar registrada—, o (c) el usuario lo desactiva.
- Se desactiva automáticamente al iniciar una nueva sesión.
- Cada respuesta en este modo termina con: **⚡ MODO PROACTIVO (puedes pedirme calmarme/bajar la velocidad)**

**Radar de contexto** *(activo en ambos modos)*
- El agente mantiene en mente el estado global del proyecto (roadmap, deudas abiertas, plan de sesión).
- Cuando detecte que la conversación pasa por alto algo relevante — una deuda pendiente, una dependencia del roadmap, una decisión previa que afecta lo que se discute — lo señala al final de su respuesta con: 💡 **Radar:** *(observación breve)*.
- No se repite si el usuario ya lo vio. No interrumpe el flujo de la respuesta.

---

## §2. Ritual de Inicio de Sesión

Al comenzar cualquier sesión, el agente ejecuta estos pasos **en orden y antes de cualquier otra acción**:

1. Leer `log.md` (historial de cambios).
2. Leer las **Deudas abiertas** del `session-*.md` más reciente. Si una deuda ya fue resuelta, marcarla.
3. Leer `planning/roadmap.md` (estado de fases y planificación global).
4. Crear (o retomar) el archivo `planning/session-YYYY-MM-DD.md` con hora de inicio y objetivo.
5. Presentar al usuario un breve resumen del estado y esperar instrucciones.

> El ritual **termina en el paso 5**. No se ejecuta ningún plan sin instrucciones posteriores.

---

## §3. Planificación y Ejecución

- Toda implementación requiere que el plan esté **previamente descrito** en la sesión activa.
- Al retomar trabajo, el agente busca el último checklist no completado en los tres archivos de sesión más recientes (orden cronológico).
- Si no existe sesión activa, el agente la crea con nombre `planning/session-YYYY-MM-DD.md`.
- Registra en el encabezado de cada sesión el **tiempo cronológico** (inicio, interrupciones, término) y el **tiempo efectivo** de trabajo. Toma como referencia el formato de sesiones anteriores.

### Template de sesión

Usar [`planning/session-TEMPLATE.md`](planning/session-TEMPLATE.md) como base para cada nueva sesión.

---

## §4. Registros Obligatorios

### `log.md`

`log.md` es la memoria de largo plazo del proyecto. Al escribir en él:

- Agrupa entradas por fecha: `## YYYY-MM-DD - Título descriptivo`.
- Describe la **acción y el valor aportado**, no el nombre del archivo modificado.
- Registra **inmediatamente** al realizar un cambio significativo; no acumules para el final.
- Sigue el tono y formato de las entradas existentes.
- Tanto el agente como el usuario escriben en el log. Si el usuario agrega una entrada, no la modifiques.

### Correcciones y Mejoras

- Se registran en el `session-*.md` activo, bajo `## Correcciones` y `## Mejoras`, con síntoma/causa/fix.
- Las correcciones se identifican como `C-XX`, las mejoras como `M-XX` (numeración secuencial por sesión).
- Los cambios de documentación y reglas **no requieren entrada en `planning/`**; solo se registran en `log.md`.

### Coherencia documental

Al modificar cualquier documento, revisa si el cambio afecta a otros documentos del repositorio y propaga las actualizaciones necesarias. Esto incluye referencias cruzadas, supuestos compartidos y decisiones que se mencionan en más de un lugar.

---

## §5. Cierre de Sesión

Se activa **solo cuando el usuario lo indica explícitamente**. Flujo estricto:

1. Registra la **hora de término** en el encabezado de la sesión.
2. Genera la sección `## Introspección de la Sesión` con la estructura base:
   - **TL;DR** — Métricas duras y la idea central en 2-3 líneas.
   - **Cadena de decisiones** — Macro-decisiones y sus derivaciones.
   - **Micro-decisiones clave** — Tabla (contexto → impacto).
   - **Sorpresas** — Supuestos invalidados, con contexto.
   - **Aprendizajes** — 1-3 lecciones con implicancia explícita.
   - **Métricas** — Tabla de dimensiones clave.
   - **Reflexiones del PM** — *(se completa interactivamente, ver paso 3)*.
   - **Deudas abiertas** — *(siempre al final absoluto, ver paso 4)*.
3. Da paso al usuario para sus reflexiones. El usuario puede enviar múltiples prompts. **Acusa recibo sin tomar acciones** hasta que el usuario devuelve el control.
4. Al recibir el control, redacta `Reflexiones del PM` y, a partir de todo el contexto, infiere y escribe `Deudas abiertas`.

> **Tono**: Compacto pero con desarrollo suficiente para extrapolar ideas sin dificultad.
