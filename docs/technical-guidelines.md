# Guía Técnica

> Convenciones técnicas, estándares de código y buenas prácticas para este repositorio.

---

## Tests

- Todo código de backend (Python) debe tener tests con `pytest` en `tests/`.
- Los tests deben pasar antes de dar una fase por completada.
- Metodología TDD: RED → GREEN → REFACTOR.

## Documentación de Código

- Mantener `docs/` actualizado.
- Docstrings en funciones Python.
- JSDoc en funciones JavaScript.
- Cada componente nuevo se documenta antes de avanzar a la siguiente fase.

## Stack Tecnológico

Las decisiones de stack están documentadas en las [Especificaciones Técnicas](technical_specs.md) (sección 5). Cualquier cambio de tecnología debe registrarse en `log.md`.

## Documentación y Diagramas

- **Mermaid JS (Prevención de roturas de layout):** En diagramas `graph TD` que utilicen subgrafos, el motor interno (Dagre) asume un flujo "arriba hacia abajo". Si hubiese una flecha de retorno cruzando "hacia arriba" entre dos subgrafos internamente, Dagre intentará resolver el "ciclo" desplazando los subgrafos de manera horizontal, rompiendo la experiencia visual. 
  - **Convención:** En caso de flujos de retorno (loops cruzados), anclar siempre las flechas explícitamente en el nodo principal (borde) pertinente, convirtiendo las conexiones de vuelta a flujos intra-nodo (internos), permitiendo que la comunicación inter-nodo entre capas siempre conserve la misma dirección fundamental.

## Seguridad

- **Manejo de Credenciales:** Bajo ninguna circunstancia se debe comitear al repositorio ningún tipo de secreto o credencial (API Keys, contraseñas, tokens de acceso). Las claves (como `GEMINI_API_KEY`) deben proveerse siempre mediante variables de entorno local o uso de archivos `.env`, los cuales deben estar obligatoriamente excluidos mediante `.gitignore`. El agente **jamás** incluirá un API key real o funcional en un commit.
