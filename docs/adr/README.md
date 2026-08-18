# Architecture Decision Records

This directory contains the durable architecture decisions for Daedalus. ADR numbers are assigned once and never reused, even when a decision is later replaced.

## Status lifecycle

- **Proposed:** the direction is documented but still needs evidence or a decision from its owning feature.
- **Accepted:** the decision constrains implementation.
- **Superseded:** a newer ADR replaces the decision. The old ADR remains unchanged except for its status and a link to the replacement.
- **Deprecated:** the decision no longer applies and has no direct replacement. The ADR remains as historical context.

Changing a decision requires a new ADR. Do not rewrite the reasoning of an accepted ADR. Update its status, add a `Superseded by` link when applicable, and describe the new reasoning in the new record.

## Index

| ADR | Status | Decision |
| --- | --- | --- |
| [ADR-001](001-workspace-storage-boundary.md) | Accepted | Workspace storage boundary and portable package layout |
| [ADR-002](002-sqlite-access-ownership.md) | Accepted | SQLite access ownership |
| [ADR-003](003-interchange-schema-versioning.md) | Proposed | Canonical JSON/YAML interchange schema and version policy |
| [ADR-004](004-entity-identifiers-import-collisions.md) | Accepted | Entity identifiers and import collision policy |
| [ADR-005](005-impact-propagation.md) | Accepted | Default impact propagation and status ordering |
| [ADR-006](006-impact-scoring.md) | Proposed | Impact scoring strategy and versioning |
| [ADR-007](007-internationalization.md) | Proposed | Internationalization library and fallback policy |
| [ADR-008](008-component-deletion.md) | Proposed | Component deletion and reassignment policy |
| [ADR-009](009-ui-state-boundaries.md) | Accepted | Canonical and transient UI state boundaries |
| [ADR-010](010-extension-boundaries.md) | Accepted | Extension and integration boundaries |

Use [the template](000-template.md) for new decisions.
