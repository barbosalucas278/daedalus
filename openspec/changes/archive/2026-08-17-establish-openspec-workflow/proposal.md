## Why

Daedalus necesita un modo único y auditable de convertir el backlog de GitHub en cambios implementables. Sin una convención explícita, las issues, las especificaciones y los PRs pueden perder trazabilidad.

## What Changes

- Establecer OpenSpec como el registro versionado de propuesta, diseño, tareas y decisiones de cada cambio.
- Definir el vínculo obligatorio entre una issue de GitHub, una change de OpenSpec, una rama y un PR hacia `develop`.
- Exigir TDD para cada comportamiento: escribir y ejecutar una prueba que falle antes de implementar, hacerla pasar con el mínimo cambio y refactorizar con la suite verde.
- Definir la entrega mediante un PR a `develop` que referencia las issues resueltas, sin gate de aprobación de QA.

## Capabilities

### New Capabilities

Ninguna. Este cambio define proceso y tooling, no comportamiento del producto.

### Modified Capabilities

Ninguna.

## Impact

- `openspec/config.yaml` y documentación de contribución que se introducirá con la implementación.
- Las issues #14 y las futuras features de Daedalus.
- Flujo de ramas, PRs y pruebas del repositorio.
