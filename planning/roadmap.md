# Roadmap de Ejecución

> Documento derivado de la [Arquitectura](architecture.md) y las [Especificaciones Técnicas](technical_specs.md). Define las fases de trabajo, sus entregables y criterios de completitud para Chaman-ear v0.1.

> **Metodología:** Cada fase sigue TDD estricto (RED -> GREEN -> REFACTOR). Ninguna fase se considera completa hasta que todos sus tests pasen.

---

## Fase 0: Investigación y Resolución de Incógnitas

**Objetivo:** Responder las preguntas abiertas de las [specs (sección 6)](technical_specs.md#6-puntos-abiertos-para-investigación) que impactan decisiones de arquitectura. Sin esto, las fases siguientes se construyen sobre supuestos no validados.

### Tareas

- [x] Investigar la API de Google AI Studio Live:
  - [x] ¿Acepta conexión WebSocket nativa (vía Python/FastAPI), o requiere el SDK oficial (`google-genai`)?
  - [x] ¿Acepta audio en formato WebM/Opus, o requiere PCM/WAV?
  - [x] ¿Emite hipótesis parciales nativamente, o solo texto cuando considera una unidad completa?
  - [ ] ¿Cuál es el límite real de tokens del system prompt?
- [ ] Crear un script de prueba mínimo en Python que envíe audio a Google AI Studio Live y reciba texto, para validar la viabilidad técnica.
- [ ] Documentar hallazgos en `docs/research/gemini-live-api.md`.

### Criterios de "done"

- Las preguntas abiertas de las specs tienen respuesta documentada.
- Existe un script funcional en Python que demuestra la comunicación con Google AI Studio Live.

---

## Fase 1: Setup del Repositorio

**Objetivo:** Tener un monorepo funcional con las herramientas de desarrollo configuradas, listo para escribir código con TDD desde el primer minuto.

### Tareas

- [ ] Inicializar monorepo/estructura:
  - [ ] `packages/sdk/` — SDK de JavaScript (TypeScript, npm).
  - [ ] `backend/` — API del servidor en **Python (FastAPI)**.
  - [ ] `packages/demo/` — Aplicación de demostración.
- [ ] Configurar herramientas de desarrollo:
  - [ ] JS/TS: `vitest`, `prettier`, `eslint`, `tsup`.
  - [ ] Python: Entorno virtual (`uv` o `venv`), `pytest`, dependencias principales (`fastapi`, `uvicorn`, `websockets`).
- [ ] Crear `config/system-prompt.txt` con la versión inicial del prompt (de las specs, sección 3.3).
- [ ] Verificar que los scripts base de testing y linting funcionan.

### Criterios de "done"

- Dependencias instaladas en ambos lenguajes.
- Los tests "vacíos" de JS y Python se ejecutan exitosamente.
- La estructura de directorios refleja la división cliente(JS)/servidor(Python).

---

## Fase 2: SDK de JavaScript

**Objetivo:** Implementar la clase `ChamanEar` completa con captura de audio y comunicación WebSocket, testeable con mocks del servidor.

### Tareas

- [ ] **Clase `ChamanEar`** (interfaz pública según specs sección 1.1):
  - [ ] Test: constructor acepta `ChamanEarConfig` con valores por defecto.
  - [ ] Test: `start()` solicita permiso de micrófono y abre WebSocket.
  - [ ] Test: `start()` lanza `MediaError` si no hay acceso al micrófono.
  - [ ] Test: `start()` lanza `ConnectionError` si el servidor no responde.
  - [ ] Test: `stop()` envía mensaje `stop` y cierra conexión.
  - [ ] Test: `onAcousticFeedback` se dispara al recibir evento de transcripción pura (stt) del servidor.
  - [ ] Test: `onSmartContent` se dispara al recibir evento generativo del modelo, tras pausa VAD.
  - [ ] Test: `onError` se dispara al recibir evento `error` del servidor.
  - [ ] Test: `onStateChange` refleja transiciones de estado correctas.
- [ ] **Captura de audio:**
  - [ ] Test: `AudioContext` remuestrea audio del micro a PCM 16kHz mono.
  - [ ] Test: los chunks se empaquetan en Base64 dentro de un payload JSON cada `chunkInterval` ms.
- [ ] **Mensajes WebSocket:**
  - [ ] Test: el mensaje inicial contiene `context` y `formatInstruction`.
  - [ ] Test: `formatInstruction` usa el valor por defecto si se omite.
  - [ ] Test: los frames de audio se envían como JSON con mimeType explícito.
- [ ] Configurar build con `tsup` o `esbuild` (ESM + CJS).
- [ ] Escribir `README.md` del paquete SDK con el ejemplo de uso de las specs (sección 1.2).

### Criterios de "done"

- Todos los tests unitarios pasan con servidor mockeado.
- El paquete se construye sin errores (`pnpm build` en `packages/sdk/`).
- El `README.md` del SDK incluye ejemplo de uso funcional.

> **Nota:** La validación real contra un servidor funcional ocurre en la Fase 4.

---

## Fase 3: Servidor (API)

**Objetivo:** Implementar el proxy WebSocket-a-WebSocket que conecta el SDK con Google AI Studio Live, incluyendo el system prompt como artefacto configurable.

### Tareas

- [ ] **WebSocket server:**
  - [ ] Test: acepta conexiones WebSocket en `/ws`.
  - [ ] Test: parsea correctamente el mensaje `start` (extrae `context` + `formatInstruction`).
  - [ ] Test: rechaza conexiones sin mensaje `start` válido.
- [ ] **Construcción del system prompt:**
  - [ ] Test: lee `config/system-prompt.txt` al iniciar.
  - [ ] Test: sustituye `{context}` y `{formatInstruction}` con los valores de la sesión.
  - [ ] Test: usa el `formatInstruction` por defecto si el cliente no lo envía.
- [ ] **Proxy a Gemini Live (vía google-genai):**
  - [ ] Test: inicia sesión de agente asíncrono con Google AI Studio Live al recibir `start`.
  - [ ] Test: reenvía payloads JSON con frames PCM Base64 a Gemini.
  - [ ] Test: intercepta eventos `inputTranscription` y los reenvía como eventos provisionales (acústicos).
  - [ ] Test: intercepta eventos `serverContent` y los reenvía como eventos definitivos (generativos, completitud inteligente).
- [ ] **Manejo de cierres:**
  - [ ] Test: al detener el micro, cierra conexión con Gemini de forma grácil.
  - [ ] Test: cierra conexión con Gemini y con el cliente.
- [ ] **Manejo de errores:**
  - [ ] Test: errores de Gemini se traducen en eventos `error` para el cliente.
  - [ ] Test: desconexión inesperada del cliente cierra la sesión de Gemini.

### Criterios de "done"

- Todos los tests unitarios pasan (con Google AI Studio Live mockeado donde sea necesario).
- El servidor arranca con `pnpm dev` y acepta conexiones WebSocket.
- El system prompt se carga desde el archivo externo y se sustituyen las variables.

---

## Fase 4: Aplicación de Demostración

**Objetivo:** Construir la webapp mínima descrita en el brief (sección 5.3) usando el SDK como un desarrollador externo lo haría (dogfooding). **El despliegue es exclusivamente en localhost.**

### Tareas

- [ ] **Estructura HTML:**
  - [ ] Layout de pantalla completa con fondo oscuro.
  - [ ] Zona de texto para la salida (feedback provisional en gris, generación inteligente validada en blanco).
  - [ ] Botón de micrófono central con animación de pulsación.
  - [ ] Panel lateral colapsable para configurar contexto e instrucción de formato.
- [ ] **Integración con el SDK:**
  - [ ] Importar `chaman-ear` como dependencia.
  - [ ] Conectar `onAcousticFeedback` -> renderizado en gris/cursiva.
  - [ ] Conectar `onSmartContent` -> descartar gris y concatenar bloque definitivo en blanco.
  - [ ] Conectar `onError` -> mostrar mensaje de error al usuario.
  - [ ] Conectar `onStateChange` -> feedback visual (botón activo/inactivo, indicador de conexión).
- [ ] **UI del contexto:**
  - [ ] Textarea para el campo `context`.
  - [ ] Input para el campo `formatInstruction` (con placeholder del default).
  - [ ] Los valores se persisten en `localStorage`.

### Criterios de "done"

- La demo arranca con `pnpm dev` en `packages/demo/`.
- Se puede hablar, ver feedback interino y bloques del LLM en pantalla.
- El panel de contexto funciona y persiste entre recargas.
- La experiencia visual coincide con la descrita en el brief (fondo oscuro, texto que solidifica).

---

## Fase 5: Integración End-to-End y Pruebas

**Objetivo:** Validar que las tres capas (SDK + Servidor + Google AI Studio Live) funcionan juntas en un flujo real completo.

### Tareas

- [ ] **Tests de integración:**
  - [ ] Test E2E: flujo completo start -> audio -> provisional -> generativo -> stop.
  - [ ] Test E2E: manejo de errores (servidor caído, Gemini no disponible).
  - [ ] Test E2E: reconexión o fallo elegante ante pérdida de conexión.
- [ ] **Validación de completitud inteligente:**
  - [ ] Probar con los 4 ejemplos del brief (sección 8) como casos de referencia.
  - [ ] Iterar el system prompt (`config/system-prompt.txt`) hasta que los 4 ejemplos produzcan resultados satisfactorios.
  - [ ] Documentar iteraciones del prompt en el log.
- [ ] **Validación de rendimiento:**
  - [ ] Medir latencia percibida: tiempo entre habla y aparición del primer feedback provisional.
  - [ ] Medir latencia de bloque: tiempo entre pausa de habla y generación del contenido inteligente.
- [ ] **Documentación:**
  - [ ] `README.md` principal del repo con quick-start guide.
  - [ ] Verificar que la documentación en `docs/` refleja el estado final.
- [ ] **Publicación:**
  - [ ] Preparar el paquete SDK para publicación en npm (pendiente de decisión).
  - [ ] Repositorio público en GitHub.

### Criterios de "done"

- Un usuario puede clonar el repo, ejecutar `pnpm install && pnpm dev`, y dictar texto que aparece refinado en la demo.
- Los 4 ejemplos del brief producen resultados cualitativamente aceptables.
- La latencia percibida es inferior a 1 segundo para el primer feedback acústico.
- Todos los tests (unitarios + integración) pasan.
