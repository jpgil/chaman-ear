# Chaman-ear

> Voz a texto en streaming con interpretación de intención.

**SDK Inteligente de Voz a Texto en Tiempo Real**  
**Documento de Definición de Producto — v0.1**  
**Fecha:** Febrero 2025  
**Autores:** Juan P. Gil & Claude (Anthropic)  

---

## 1. Resumen

**Chaman-ear** es un SDK de código abierto que habilita el "dictado inteligente" en tiempo real: el usuario habla de forma natural —con dudas, muletillas y frases incompletas— y lo que aparece en pantalla es un párrafo limpio y bien estructurado que preserva la intención original. No es una transcripción literal, sino **el texto que el usuario habría escrito si se hubiera tomado su tiempo**.

El proyecto fue concebido y definido de manera colaborativa entre Juan P. Gil y Claude (Anthropic) mediante una conversación de descubrimiento de producto en febrero de 2025. La idea surgió de la necesidad de acelerar la creación de textos bien redactados —prompts, documentos técnicos, contenido literario, correos electrónicos— mediante la voz, sin la fricción de la edición manual.

El nombre **Chaman-ear** evoca la intuición, la transformación y la idea de escuchar más allá de las palabras literales: capturar el significado, no el ruido.

---

## 2. Definición del Problema

Las herramientas de voz a texto actuales producen transcripciones literales. Capturan cada "uhm", cada frase repetida y cada inicio en falso. Luego, se exige al usuario editar manualmente el resultado en bruto, lo que elimina la mayor parte de la ventaja de productividad que supone hablar en lugar de escribir.

Los desarrolladores que desean integrar la voz en sus aplicaciones enfrentan un desafío adicional: deben ensamblar una capa de captura de audio, un servicio ASR y un modelo de lenguaje por sí mismos, sin contar con una abstracción estandarizada con la cual trabajar.

**Chaman-ear** resuelve ambos problemas al:
*   **Abstraer** toda la tubería de audio a texto refinado en un solo SDK.
*   **Entregar** salida en streaming —no un bloque final de texto tras una larga pausa—.
*   **Permitir** al desarrollador definir el formato de salida y el contexto, haciendo el sistema configurable para cualquier caso de uso de escritura.

---

## 3. Visión del Producto

Chaman-ear tiene una filosofía **API-first**. La inteligencia central reside en una API centralizada bajo el control del autor. El SDK es un cliente ligero que cualquier desarrollador instala en minutos e integra en su propia aplicación web. Una aplicación de demostración —construida con el propio SDK— sirve como el escaparate público del producto.

La ambición a largo plazo es ofrecer esto como un servicio API de pago. La fase actual es una prueba de concepto: validar la experiencia principal, abrir el código fuente del SDK y absorber internamente los costos de la API externa mientras el producto madura.

---

## 4. Público Objetivo

**Principal (v0.1 — prueba de concepto):**
*   **Desarrolladores** que construyen aplicaciones web y desean agregar entrada de voz inteligente sin ensamblar la tubería por sí mismos.
*   **Usuarios técnicos** (por ejemplo, desarrolladores que utilizan asistentes de código de IA) que desean dictar prompts, documentación o texto estructurado con salida de alta calidad.

**Públicos futuros:**
*   **Escritores** (guiones, novelas, documentación técnica).
*   **Trabajadores del conocimiento** que redactan correos electrónicos, informes o notas estructuradas mediante la voz.

---

## 5. Descripción de la Arquitectura

El sistema está compuesto por tres capas distintas:

### 5.1 La API (lado del servidor, controlada por el autor)

La API es el núcleo de inteligencia de Chaman-ear. Recibe un flujo de audio del SDK del cliente y devuelve un flujo de eventos de texto. Internamente, utiliza la **API de Google Gemini Live**, la cual maneja tanto el reconocimiento de voz como la reformulación del idioma en un solo paso —eliminando la necesidad de una capa ASR separada—.

La API emite dos tipos de eventos a través de una conexión en streaming:
*   `partial`: una hipótesis inestable. El texto aún se está formando. El cliente puede renderizar esto en un estilo visualmente distinto (por ejemplo, atenuado, en cursiva). Es probable que sea reemplazado.
*   `final`: un bloque de texto validado. El modelo ha determinado que la idea está semánticamente completa. El cliente renderiza esto como texto estable.

La decisión de cuándo emitir un evento final es tomada por el modelo de lenguaje en base a la completitud semántica, no simplemente en base a pausas en el audio. Esta es una decisión de diseño deliberada: un usuario puede hacer una pausa a mitad de un pensamiento y reanudarlo, y el sistema no debe cerrar prematuramente un bloque. La única excepción es cuando el usuario suelta el botón del micrófono —esto siempre desencadena un evento final forzado para el bloque actual—.

La API es **sin estado (stateless)** por diseño. El cliente envía todo el contexto necesario con cada solicitud. Esto simplifica el escalado, elimina la gestión de sesiones y hace que el sistema sea más fácil de razonar. La compensación —cargas útiles ligeramente mayores— es aceptable dados los tamaños de contexto esperados (5–10 páginas, aproximadamente 3,000–6,000 tokens).

### 5.2 El SDK de JavaScript (lado del cliente, paquete npm)

El SDK es una biblioteca de JavaScript distribuida a través de `npm`. Se ejecuta en el navegador y maneja tres responsabilidades: capturar el audio del micrófono utilizando la API nativa `MediaRecorder` del navegador, mantener el WebSocket o la conexión de flujo con la API de Chaman-ear, y exponer una interfaz de eventos limpia al desarrollador.

La interfaz pública del SDK es mínima y orientada a eventos:
*   `onPartial(text)`: se dispara cuando llega una hipótesis parcial.
*   `onFinal(text)`: se dispara cuando un bloque validado está listo.
*   `start(context, config)`: inicia una sesión con el contexto y formato de salida definidos por el usuario.
*   `stop()`: finaliza la sesión y activa un evento final forzado.

### 5.3 La Aplicación de Demostración

Una aplicación web de una sola página construida con el propio SDK de Chaman-ear. Sirve tanto como demostración funcional del producto como documentación viva para los desarrolladores que evalúan el SDK.

El diseño es intencionalmente minimalista: un fondo oscuro, un único botón de micrófono parpadeante en el centro y una zona de texto encima donde aparece el resultado. El texto parcial se muestra en un tono gris apagado, solidificándose a blanco cuando llega un evento final. Un panel lateral colapsable permite al usuario configurar su contexto antes de empezar a hablar.

---

## 6. El Sistema de Contexto

La característica más distintiva de Chaman-ear es su sistema de contexto configurable. Antes de iniciar una sesión, el usuario proporciona dos entradas:

*   **Contexto:** información de fondo que el modelo debe conocer. Esto puede ser un extracto de texto, una lista de personajes, un hilo de correos electrónicos anterior, un glosario técnico, o cualquier otro material que deba informar la salida. Hasta aproximadamente 10 páginas de texto.
*   **Instrucción de formato:** una descripción en lenguaje natural del estilo de salida deseado. Esta es una cadena de formato libre sin opciones predefinidas.

**Ejemplos de instrucciones de formato:**
*   "Escribe como una escena para un guion de teatro negro de los años 1940, respetando las voces de los personajes establecidas en el contexto."
*   "Genera la salida como un correo electrónico profesional bien redactado con mi propia voz, utilizando el tono de los correos de ejemplo en el contexto."
*   "Produce un prompt de IA conciso para un modelo de código. Elimina las muletillas, combina ideas relacionadas y usa los acrónimos técnicos del contexto."
*   "Formatea la salida como una especificación técnica estructurada en Markdown."

Este contexto se envía en **cada llamada a la API** (diseño sin estado). El SDK del cliente es responsable de almacenarlo y transmitirlo. Este diseño mantiene la API simple y garantiza que ningún dato del usuario se persista en el servidor.

---

## 7. Decisiones Tecnológicas

### 7.1 Google Gemini Live API
Seleccionado como el único proveedor de IA para la prueba de concepto. Gemini Live acepta la entrada de audio sin procesar y devuelve texto en tiempo real, colapsando la tubería ASR + LLM en una sola llamada API. Esto reduce la latencia, simplifica la arquitectura del lado del servidor y mantiene al mínimo el número de dependencias externas. El nivel gratuito de Google AI Studio lo hace viable para una prueba de concepto sin costo inicial de infraestructura.

### 7.2 JavaScript (Navegador) para Captura de Audio
La API `MediaRecorder` del navegador es la forma más universal y multiplataforma de capturar audio del micrófono. Al colocar la lógica de captura en JavaScript ejecutándose en el navegador, el SDK evita la complejidad de las bibliotecas de audio específicas de plataforma (por ejemplo, PyAudio en Linux/macOS/Windows). El SDK tiene como objetivo principal las aplicaciones web como entorno de despliegue.

### 7.3 Diseño de API sin estado (stateless)
Elegido frente a un enfoque basado en sesiones por su simplicidad y escalabilidad. El cliente mantiene todo el estado. Cada solicitud es independiente. Este es el valor predeterminado correcto para una prueba de concepto y se puede evolucionar hacia un modelo de almacenamiento en caché híbrido en una versión futura si los tamaños de contexto o la frecuencia de llamadas lo hacen necesario.

---

## 8. Completitud Inteligente

Este es el principio de diseño central de Chaman-ear y lo que lo diferencia de cualquier transcriptor convencional: **la salida parcial no es una transcripción literal del audio recibido hasta el momento.** Es la versión que el usuario habría escrito si hubiera tenido tiempo de pensar, corregir y completar su idea.

Cada evento `partial` debe cumplir estas propiedades:
*   **Gramaticalmente correcto**: sin muletillas, sin repeticiones, sin pausas transcritas.
*   **Semánticamente completo**: si el usuario cortó la frase a la mitad, el modelo infiere el final más probable y lo incluye.
*   **Fiel a la intención**: no inventa contenido nuevo; completa lo que el usuario claramente iba a decir.
*   **Respeta el formato solicitado**: si la instrucción de formato pide un estilo específico, el parcial ya lo refleja.

### Ejemplos

**Ejemplo 1 — Prompt para modelo de código**

| | Contenido |
|---|---|
| **Entrada (voz, cortada)** | *"quiero que eh... mires el repo y me digas tres... tres cosas que se pueden refactorizar, bueno, que reduzcan el acoplamiento entre mó..."* |
| **Salida parcial** | Quiero que analices el repositorio y me propongas tres cosas que se puedan refactorizar para reducir el acoplamiento entre módulos. |

**Ejemplo 2 — Guion de teatro**

| | Contenido |
|---|---|
| **Entrada (voz, cortada)** | *"marlowe le dice a carmen que sabe que estuvo en el almacén esa noche, que tiene un testigo, aunque en realidad no tie..."* |
| **Salida parcial** | Marlowe le dice a Carmen que sabe que estuvo en el almacén esa noche y que tiene un testigo, aunque en realidad no tiene ninguna prueba. |

**Ejemplo 3 — Email profesional**

| | Contenido |
|---|---|
| **Entrada (voz, cortada)** | *"quiero escribirle al cliente de Acme, que lleva dos semanas sin contestar la propuesta, preguntarle si puede..."* |
| **Salida parcial** | Quiero escribirle al cliente de Acme, que lleva dos semanas sin responder la propuesta, para preguntarle si puede reunirse esta semana. |

**Ejemplo 4 — Documentación técnica**

| | Contenido |
|---|---|
| **Entrada (voz, cortada)** | *"el método start recibe el contexto y la configuración, devuelve una promesa, y si falla porque no hay micrófono lanza un..."* |
| **Salida parcial** | El método start recibe el contexto y la configuración, devuelve una promesa, y si falla porque no hay acceso al micrófono lanza un error de tipo MediaError. |

> **Implicación para la arquitectura:** Toda la "inteligencia" de completitud sucede dentro de Gemini, guiada por el system prompt que el servidor construye a partir del contexto y la instrucción de formato del usuario. El servidor no realiza ningún post-procesamiento de texto.

---

## 9. Contrato de Streaming

El comportamiento de Chaman-ear está inspirado en la renderización en tiempo real basada en diferencias (diffs) en herramientas como Cursor. **Se espera que el texto en la pantalla cambie:** los bloques anteriores pueden ser reescritos a medida que llega nuevo contexto. Esto es una característica, no un error.

El contrato de streaming entre la API y el SDK es:
*   Mientras el micrófono está activo, la API emite continuamente eventos `partial` a medida que Gemini procesa el audio entrante.
*   Cuando el modelo determina que una unidad semántica está completa (una idea ha sido totalmente expresada), emite un evento `final` con el texto validado.
*   Cuando el usuario suelta el botón del micrófono, el SDK llama a `stop()`, lo cual fuerza un evento `final` para cualquier texto en curso pendiente y cierra el flujo.

---

## 10. Alcance para la v0.1 — Prueba de Concepto

**Dentro del alcance:**
*   SDK de JavaScript (npm) con captura de micrófono e interfaz de eventos en streaming.
*   API del lado del servidor que integra Gemini Live para streaming de audio a texto refinado.
*   Sistema de contexto: documento de contexto de formato libre + instrucción de formato de forma libre, enviados de forma independiente por llamada.
*   Aplicación de demostración presentando el SDK con la interfaz de usuario mínima descrita en la sección 5.3.
*   Repositorio público en GitHub con documentación y guía de inicio rápido.

**Fuera de alcance para la v0.1:**
*   Autenticación, limitación de velocidad, o cuentas de usuario en la API.
*   Envoltorio (wrapper) en Python o soporte nativo de escritorio.
*   Monetización o niveles de API de pago.
*   Caché de contexto basada en sesiones.

---

## 11. Consideraciones Futuras
*   Paquete Python (pip) que envuelva el SDK de JavaScript para desarrolladores backend.
*   Caché de contexto basada en sesiones para reducir el tamaño de las cargas útiles en sesiones largas.
*   Soporte para proveedores alternativos de IA (OpenAI, Deepgram) mediante una interfaz adaptadora.
*   Nivel de API paga con precios basados en el uso (por minuto de audio procesado o por token de salida).
*   Integraciones nativas con herramientas para desarrolladores (por ejemplo, extensión para VS Code, panel de Cursor).
