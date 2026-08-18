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

## Risks / Trade-offs

- [Prematurely accepted decisions constrain implementation] → Mark unresolved choices Proposed and link their follow-up issue/change.
- [ADRs duplicate source documents] → ADRs cite PRD/ARD and focus on the decision, alternatives, and consequences.

## Migration Plan

1. Create the ADR template and index.
2. Add initial records for workspace persistence, interchange, IDs, UI state, plugins, and impact strategy seams.
3. Reference ADRs from later OpenSpec designs and PRs.
