## Context

ARD §13 names architectural decisions that should be recorded before or during implementation. The repository does not yet have a durable ADR location or format.

## Goals / Non-Goals

**Goals:**

- Establish a lightweight, numbered ADR format in version control.
- Capture the decisions that constrain the first MVP work.

**Non-Goals:**

- Rewriting the PRD or ARD.
- Finalizing decisions that need implementation evidence; those remain proposed until resolved.

## Decisions

- **Use Markdown ADRs under `docs/adr/` with immutable numbers.** Markdown is reviewable in PRs and numbers provide stable references.
- **Use status values Proposed, Accepted, Superseded, and Deprecated.** This preserves decision history instead of editing old reasoning away.
- **Create only decisions supported by PRD/ARD.** New product scope is not introduced through ADRs.
- **Use UUIDv7 for user-created workspaces and entities.** UUIDv7 is generated locally, remains opaque to the domain, and provides better insertion locality than UUIDv4. Persist and interchange UUIDs in canonical lowercase hyphenated form. Sequential and semantic identifiers were rejected because they require coordination or couple identity to mutable data.
- **Give built-in component types deterministic UUIDv5 identifiers.** Derive them from a fixed Daedalus namespace and their canonical English keys so every installation and interchange file resolves the same built-ins. Custom component types use UUIDv7.
- **Scope entity identity to its workspace.** The logical identity is `(workspaceId, entityId)`. Entity IDs are immutable and unique within a workspace; MVP 1 does not permit cross-workspace references.
- **Preserve identity during normal import and require an explicit collision choice.** When the imported `workspaceId` is not registered locally, create an isolated workspace and preserve every ID. When it already exists, make no changes until the user chooses Replace, Clone, or Cancel:
  - **Replace** validates the snapshot in temporary storage before atomically replacing the existing workspace while preserving every ID.
  - **Clone** generates a new `workspaceId`, updates that workspace scope throughout the snapshot, and preserves entity IDs so corresponding entities remain comparable across copies.
  - **Cancel** leaves local storage unchanged.
  Automatic merge and silent identifier remapping are excluded from MVP 1. Merge requires a separate conflict and version model; remapping every entity during clone would break stable correspondence without solving a current requirement.

## Risks / Trade-offs

- [Prematurely accepted decisions constrain implementation] → Mark unresolved choices Proposed and link their follow-up issue/change.
- [ADRs duplicate source documents] → ADRs cite PRD/ARD and focus on the decision, alternatives, and consequences.
- [UUIDv7 exposes approximate creation time] → Treat IDs as opaque and store explicit timestamps only when product behavior needs them; the local-first portability benefit and database locality outweigh this limited disclosure.
- [Clones can diverge while retaining corresponding entity IDs] → Always scope lookups and references by workspace and present clones as independent workspaces; future synchronization must define its own lineage and conflict policy.

## Migration Plan

1. Create the ADR template and index.
2. Add initial records for workspace persistence, interchange, identifiers and import collisions, UI state, plugins, and impact strategy seams.
3. Reference ADRs from later OpenSpec designs and PRs.
