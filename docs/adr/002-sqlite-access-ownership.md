# ADR-002: SQLite access ownership

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Daedalus uses a React/TypeScript frontend inside Tauri while SQLite and filesystem lifecycle are native concerns. The ownership boundary must preserve a pure, testable domain and prevent persistence details from leaking into UI code.

## Decision

Domain entities, application use cases, commands, queries, and persistence ports are defined in TypeScript without dependencies on React, Tauri, SQL, or the filesystem.

Rust/Tauri owns workspace file selection, database connection lifecycle, migrations, transactions, and concrete SQLite access. It exposes narrow, allowlisted commands with serializable request/response contracts and stable application error codes. TypeScript persistence adapters implement application ports by invoking those commands.

No React component, UI state module, or graph adapter may issue raw SQL or receive an unrestricted filesystem path. A Tauri SQL plugin may be used internally by the native persistence implementation, but its generic JavaScript query API is not an application boundary.

## Alternatives considered

- **Execute SQL directly from frontend TypeScript:** reduces native code, but exposes schema and transaction details to the UI and permits bypassing application invariants.
- **Implement domain and application services entirely in Rust:** creates a strong native boundary, but duplicates TypeScript contracts and makes pure UI-adjacent use cases harder to test and evolve.
- **Keep SQLite in TypeScript through a sidecar runtime:** adds packaging and lifecycle complexity without improving the architectural boundary.

## Consequences

### Positive

- Domain and application logic remain portable and fast to unit test.
- Native capabilities stay behind explicit Tauri permissions and typed commands.
- SQLite migrations and transactions have one owner.

### Negative

- TypeScript and Rust need synchronized DTO and error contracts.
- Repository calls cross the Tauri IPC boundary and should be coarse enough to avoid chatty access.
- Integration tests must cover both persistence behavior and serialization across the boundary.

## Traceability

- Product source: [PRD §9 — Maintainability and privacy](../../PRD.md)
- Architecture source: [ARD §§2, 3, 10, and 11](../../ARD.md)
- Follow-up: [Issue #5 — Desktop platform](https://github.com/barbosalucas278/daedalus/issues/5), [Issue #6 — Workspace persistence](https://github.com/barbosalucas278/daedalus/issues/6)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
