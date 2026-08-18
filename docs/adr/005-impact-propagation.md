# ADR-005: Default impact propagation and status ordering

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Impact analysis must produce deterministic, explainable results over directed dependencies, including graphs with multiple paths and cycles. The semantics cannot be inferred from UI arrows or hidden in presentation code.

## Decision

Stored dependencies point from the component that depends on or acts upon another component to its target. Impact therefore traverses a reverse adjacency projection from a failed target to its dependants; it does not create stored reverse edges.

The default propagation matrix is:

| Target status | Dependency severity | Source result |
| --- | --- | --- |
| `UNAVAILABLE` | `REQUIRED` | `UNAVAILABLE` |
| `UNAVAILABLE` | `DEGRADED` | `DEGRADED` |
| `UNAVAILABLE` | `OPTIONAL` | `UNAFFECTED`, with an optional diagnostic |
| `DEGRADED` | `REQUIRED` | `DEGRADED` |
| `DEGRADED` | `DEGRADED` | `DEGRADED` |
| `DEGRADED` | `OPTIONAL` | `UNAFFECTED` |

Status strength is `UNAVAILABLE > DEGRADED > UNAFFECTED`. When multiple paths reach a component, retain the strongest result and enough predecessor information to explain the winning path. A component is reprocessed only when a stronger state is discovered, which terminates cycle traversal over the finite ordering.

All MVP 2 relation types use this shared matrix. Relation-specific behavior requires a new tested policy and ADR.

## Alternatives considered

- **Traverse stored edge direction:** identifies the dependencies of the failed component rather than components affected by it.
- **Store reverse edges:** duplicates facts and risks inconsistent topology.
- **First path wins:** depends on traversal order and can hide a stronger later path.
- **Encode rules in the graph UI:** makes headless testing and reuse impossible and violates the domain boundary.

## Consequences

### Positive

- Results are deterministic, cycle-safe, and explainable.
- Dependency storage retains one unambiguous direction.
- The engine can be tested as a pure domain/application service.

### Negative

- Analysis must build or maintain a reverse adjacency projection.
- Explanation data increases result size.
- Relation-specific nuances are intentionally deferred and require explicit evolution.

## Traceability

- Product source: [PRD §§6, 8, and 10 — MVP 2](../../PRD.md)
- Architecture source: [ARD §§6, 7, and 12](../../ARD.md)
- Follow-up: [Issue #9 — Impact propagation](https://github.com/barbosalucas278/daedalus/issues/9), [Issue #10 — Explainable results](https://github.com/barbosalucas278/daedalus/issues/10)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
