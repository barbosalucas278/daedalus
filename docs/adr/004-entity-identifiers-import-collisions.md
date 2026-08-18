# ADR-004: Entity identifiers and import collision policy

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Daedalus creates entities offline, preserves references through JSON/YAML round trips, and stores workspaces independently. Importing a snapshot whose workspace is already registered must never overwrite or merge data silently.

## Decision

User-created workspaces and entities use UUIDv7 generated locally. UUIDs are immutable and opaque to domain behavior and are persisted and exchanged as canonical lowercase hyphenated strings. SQLite initially stores them as `TEXT` for transparent validation and inspection.

Built-in component types use UUIDv5 identifiers derived from the fixed Daedalus namespace `6fc5279d-2e0f-4acc-8b6c-a977e7cb164c` and the name `component-type/<CANONICAL_KEY>`. Custom component types use UUIDv7. The fixed namespace and canonical English keys make built-in references deterministic across installations.

Entity identity is scoped to a workspace as `(workspaceId, entityId)`. Entity IDs are unique across entity kinds within a workspace. MVP 1 does not permit cross-workspace references.

Import validates all identifiers, references, and workspace scopes before persistence:

- If the `workspaceId` is not registered locally, import creates an isolated workspace and preserves every ID.
- If the `workspaceId` already exists, import makes no change until the user explicitly chooses **Replace**, **Clone**, or **Cancel**.
- **Replace** builds and validates temporary storage before atomically replacing the existing workspace while preserving every ID.
- **Clone** generates a new UUIDv7 `workspaceId`, updates the workspace scope on all records, and preserves entity IDs so corresponding entities remain comparable across copies.
- **Cancel** leaves local state unchanged.

Automatic merge and silent identifier remapping are outside MVP 1.

## Alternatives considered

- **Sequential integer IDs:** compact and index-friendly, but require coordination and are unsafe across independently created portable workspaces.
- **UUIDv4:** sufficiently unique, but lacks the insertion locality of UUIDv7.
- **Semantic IDs derived from names:** human-readable, but couple identity to mutable, localized data.
- **Regenerate every entity ID when cloning:** guarantees globally distinct raw IDs, but destroys stable correspondence and requires rewriting every reference.
- **Automatically replace or merge on collision:** reduces user interaction, but risks data loss and requires an undefined conflict model.

## Consequences

### Positive

- IDs can be generated offline without coordination and retain stable round-trip identity.
- Clones remain independent while corresponding entities can still be compared.
- Import collisions are visible and require user intent.
- Deterministic built-in IDs prevent installation-specific references.

### Negative

- UUIDv7 reveals an approximate creation time and is not a substitute for an explicit timestamp.
- Every lookup and reference must respect workspace scope.
- Replace requires temporary storage and an atomic lifecycle operation.
- Future synchronization or merge needs a separate lineage, version, and conflict model.

## Traceability

- Product source: [PRD §§3, 5, 8, and 10](../../PRD.md)
- Architecture source: [ARD §§4, 5, and 8](../../ARD.md)
- Follow-up: [Issue #16 — Core domain model](https://github.com/barbosalucas278/daedalus/issues/16), [Issue #18 — JSON/YAML interchange](https://github.com/barbosalucas278/daedalus/issues/18)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
