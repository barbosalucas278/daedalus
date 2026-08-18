# ADR-009: Canonical and transient UI state boundaries

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

React and Cytoscape need interactive state, but neither may become a second source of truth for the landscape. Portable graph positions are intentional workspace data while selection, open panels, and in-progress form state are transient.

## Decision

SQLite-backed domain data is canonical and reaches the UI through application queries/read DTOs. UI components mutate it only through typed application commands. The graph is a one-way projection of read DTOs; Cytoscape element data and runtime layout state are never authoritative domain records.

Keep transient presentation state at the narrowest practical UI scope: selection, filters, search text, dialogs, draft forms, viewport, and loading/error state. A shared UI state library may coordinate cross-screen presentation concerns, but it cannot own canonical entities or persistence rules.

Graph positions are the explicit exception: load persisted positions before rendering, update the local projection during drag, and persist the final coordinates through a debounced application command. Interchange includes positions but excludes other local UI-only state unless a later ADR designates a portable preference.

## Alternatives considered

- **Mirror the entire landscape in a global React store:** makes rendering convenient, but creates synchronization and invalidation problems with SQLite.
- **Treat Cytoscape as the topology source:** couples domain semantics to a visualization library and bypasses application validation.
- **Persist all UI state:** restores exact sessions, but pollutes portable workspaces with device-specific and ephemeral details.
- **Persist no UI state:** keeps storage simple, but violates the requirement to retain arranged graph positions.

## Consequences

### Positive

- There is one canonical landscape and one mutation path.
- React and Cytoscape remain replaceable adapters.
- Tests can distinguish domain behavior from transient interaction behavior.

### Negative

- Queries and commands need explicit DTOs and invalidation behavior.
- Dragging positions requires local responsiveness plus debounced persistence.
- A future offline undo/session feature needs a separate state and history design.

## Traceability

- Product source: [PRD §§7, 8, and 10 — MVP 1 experience](../../PRD.md)
- Architecture source: [ARD §§2, 3, 5, 8, 9, and 11](../../ARD.md)
- Follow-up: [Issue #5 — Desktop platform](https://github.com/barbosalucas278/daedalus/issues/5), [Issue #7 — Navigable graph](https://github.com/barbosalucas278/daedalus/issues/7)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
