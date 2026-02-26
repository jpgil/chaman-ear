# Chaman-ear

> Voz a texto en streaming con interpretación de intención.

## Descripción del Proyecto

**Chaman-ear** es un SDK de código abierto que habilita el "dictado inteligente" en tiempo real: el usuario habla de forma natural —con dudas y muletillas— y obtiene un párrafo limpio y bien estructurado que preserva su intención original, sin transcripciones literales ruidosas.

## Arquitectura Esperada

- **API de Backend (Python/FastAPI)**: Utiliza la API de **Google AI Studio Live** para procesar audio nativo y devolver eventos estructurados en streaming (`partial` y `final`) evaluando la semántica.
- **SDK JavaScript (NPM)**: Biblioteca cliente que maneja la grabación web (`MediaRecorder`) y los WebSockets, exponiendo métodos simples (`onPartial`, `onFinal`, `start`, `stop`).
- **Aplicación de Demostración**: Un cliente web minimalista que muestra la experiencia implementando el motor y su sistema de contexto a medida (Instrucción de formato + Contexto de Background).

---

## Reglas de Gestión (MaC)

> **Agentes de IA**: Antes de tomar cualquier acción en este repositorio, **lee y aplica** el archivo [`MaC.md`](MaC.md) (Management as Code). Contiene las reglas operativas, los modos de trabajo, el ritual de inicio de sesión y los estándares que rigen toda interacción.

## Contributing

- **Gestión del proyecto**: [`MaC.md`](MaC.md) — Modos de operación, sesiones, registros y cierre.
- **Guía técnica**: [`docs/technical-guidelines.md`](docs/technical-guidelines.md) — Tests, documentación de código, stack y convenciones.