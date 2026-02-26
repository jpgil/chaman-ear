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
