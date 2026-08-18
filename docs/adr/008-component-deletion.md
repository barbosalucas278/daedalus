# ADR-008: Component deletion and reassignment policy

- Status: Proposed
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Deleting hierarchy or graph entities can invalidate dependencies, documentation references, positions, and later flow steps. SQLite cascading alone cannot express the required user intent or present the impact before data is removed.

## Decision

Reject destructive deletion by default when dependants or references exist. An application service first returns a deletion-impact preview grouped by affected entity and reference type. The user may then cancel or invoke an explicit atomic operation.

For a component, explicit cascade removes the component, its incoming and outgoing dependencies, documentation references, graph position, and other owned records in one transaction. Dependencies are not reassigned automatically because a replacement component changes their semantics. Moving a component to another project is a separate validated update, not part of deletion.

For product or project deletion, the explicit operation may either cascade the contained hierarchy or reassign children to a valid destination selected by the user. Cross-workspace reassignment is never valid. Future flow references must follow the same preview-and-explicit-action contract.

Issue #17 must validate the preview shape and confirmation UX before this ADR becomes Accepted.

## Alternatives considered

- **Unconditional database cascade:** preserves referential integrity but can remove substantial user data without explaining the effect.
- **Always reject deletion with references:** safest, but prevents legitimate cleanup and makes users manually remove every edge.
- **Automatically reconnect dependencies to a replacement:** changes topology semantics without enough information to remain trustworthy.
- **Soft-delete every entity:** preserves history, but complicates all queries, exports, uniqueness constraints, and graph behavior without an MVP requirement.

## Consequences

### Positive

- Destructive effects are visible before they occur.
- Successful deletion remains atomic and referentially valid.
- Topology is never silently reinterpreted.

### Negative

- Deletion requires preview and confirmation contracts in addition to repository operations.
- UI flows must handle cascade and reassignment choices.
- Undo/history is not provided; recovery depends on cancellation, export, or future backup behavior.

## Traceability

- Product source: [PRD §§6 and 8 — MVP 1 CRUD](../../PRD.md)
- Architecture source: [ARD §§5 and 12](../../ARD.md)
- Follow-up: [Issue #17 — Landscape catalog and editing](https://github.com/barbosalucas278/daedalus/issues/17)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
