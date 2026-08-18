# ADR-010: Extension and integration boundaries

- Status: Accepted
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Daedalus must later support scoring strategies, repository discovery, observability, architecture-as-code, and natural-language queries without letting vendor or framework concerns redefine the core model.

## Decision

Extensions integrate through explicit domain/application ports and adapter modules. The domain never imports an integration SDK, performs network calls, or accepts vendor-specific records as canonical topology.

External discovery and observability adapters may produce typed proposals or signals. Proposed topology changes pass through the same application validation and transaction workflow as UI edits. Runtime signals remain annotations or analysis inputs and do not silently redefine components or dependency direction.

Impact scoring strategies are pure injected implementations of the contract recorded in ADR-006. Interchange, persistence, graph, desktop, and localization remain adapters around the same core model.

MVP 1 does not provide a dynamic third-party plugin runtime, executable plugin packages, or a public plugin ABI. Those capabilities require a future ADR covering trust, permissions, compatibility, discovery, and packaging.

## Alternatives considered

- **Add integration-specific fields and services to domain entities:** expedient initially, but couples the model to vendors and forks semantics.
- **Allow integrations to write directly to SQLite:** bypasses validation, transactions, and user intent.
- **Build a dynamic plugin runtime before integrations exist:** maximizes theoretical flexibility, but introduces security and compatibility costs without validated use cases.
- **Fork the core model for architecture-as-code:** creates competing sources of truth and incompatible behavior.

## Consequences

### Positive

- Core semantics remain stable, local, and testable.
- Integrations can evolve or be removed without migrations to the domain model solely for vendor concerns.
- Every topology mutation follows the same validation path.

### Negative

- Adapters require explicit mapping and may not expose every vendor capability.
- Dynamic third-party plugins are deferred.
- New extension types still require semantic documentation and interchange/migration review.

## Traceability

- Product source: [PRD §§3, 6, and 11 — progressive sophistication and future scope](../../PRD.md)
- Architecture source: [ARD §§3, 7, 11, and 14](../../ARD.md)
- Follow-up: [Issue #19 — Impact scoring](https://github.com/barbosalucas278/daedalus/issues/19), future integration issues
- OpenSpec: `openspec/changes/document-foundational-adrs/`
