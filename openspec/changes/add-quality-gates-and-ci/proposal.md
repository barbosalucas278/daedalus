## Why

Daedalus requiere garantías automatizadas desde el inicio para preservar invariantes del dominio, intercambios válidos y una entrega reproducible. Sin quality gates y práctica TDD, los cambios SDD no tendrían una verificación consistente antes del merge.

## What Changes

- Definir los checks mínimos de formato, lint, typecheck, pruebas y validación de OpenSpec.
- Diseñar un workflow de CI que se ejecute en PRs a `develop`.
- Establecer evidencia de TDD y criterios de bloqueo para el merge.

## Capabilities

### New Capabilities

Ninguna. Este cambio configura tooling y procesos de entrega, no comportamiento observable de Daedalus.

### Modified Capabilities

Ninguna.

## Impact

- Scripts de proyecto, configuración de CI y convenciones de PR.
- Issue #15 y todos los cambios posteriores.
