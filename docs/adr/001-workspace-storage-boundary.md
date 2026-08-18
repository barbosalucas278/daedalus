# ADR-001: Workspace storage boundary and portable package layout

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Daedalus is local-first, must isolate multiple ecosystems, and must provide a portable interchange format. The internal storage boundary must not make the application database an accidental public format.

## Decision

Each workspace has one isolated SQLite database. SQLite is the canonical source of truth while that workspace is open. Foreign keys, migrations, and transactions protect its invariants.

The application owns the database location and any adjacent internal manifest. This internal layout is private and may evolve through migrations. Portability is guaranteed only by the versioned JSON/YAML interchange contract, not by copying an internal SQLite file or workspace directory.

Workspace creation, opening, replacement, and removal cross an application service and a native desktop boundary. UI code does not receive unrestricted filesystem paths or manage database connections.

## Alternatives considered

- **One SQLite database containing every workspace:** simpler discovery, but weakens failure isolation, backup boundaries, and deliberate workspace replacement.
- **JSON or YAML as live canonical storage:** human-readable, but poorly suited to transactional mutations, referential integrity, and indexed graph queries.
- **Treat the SQLite file as the portable public package:** preserves exact storage, but freezes implementation details and makes future schema or storage changes part of the external contract.

## Consequences

### Positive

- Workspace lifecycle, backup, and replacement have a clear isolation boundary.
- SQLite can enforce transactional and referential invariants without constraining the interchange schema.
- Internal storage can evolve independently from portable exports.

### Negative

- The desktop layer must maintain a local workspace registry and connection lifecycle.
- Export/import requires an explicit mapping between SQLite records and the canonical snapshot.
- Moving an internal database file manually is unsupported as an interchange workflow.

## Traceability

- Product source: [PRD §§3, 6, 8, and 9](../../PRD.md)
- Architecture source: [ARD §§1, 5, and 8](../../ARD.md)
- Follow-up: [Issue #6 — Workspace persistence](https://github.com/barbosalucas278/daedalus/issues/6)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
