# ADR-006: Impact scoring strategy and versioning

- Status: Proposed
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

Impact scoring must evolve independently from graph propagation and must never overwrite user-authored Business Criticality. The default formula still needs implementation evidence and product calibration.

## Decision

Expose scoring as a pure domain/application strategy with a stable `id`, a version, and a `score(input)` operation. Input is read-only and contains the completed analysis result plus permitted component metadata. Output contains the score when meaningful, contributing factors, strategy identity/version, and a user-facing explanation payload.

A strategy cannot mutate the landscape, write Business Criticality, access persistence, perform network calls, or depend on UI frameworks. Strategy selection is dependency injection through an application port, not a dynamic runtime plugin API.

Issue #19 will define the initial formula and its versioning rules. This ADR becomes Accepted when deterministic fixtures demonstrate the selected formula, explanation factors, and behavior across formula versions.

## Alternatives considered

- **Embed scoring in propagation:** simpler execution, but couples topology semantics to a replaceable ranking policy.
- **Use Business Criticality as the impact score:** conflates intrinsic importance with scenario-specific effects.
- **Allow strategies to query integrations:** enables richer signals, but destroys determinism and makes offline explanations dependent on external state.
- **Expose a dynamic third-party plugin runtime now:** premature for MVP 2 and expands security, compatibility, and packaging scope.

## Consequences

### Positive

- Propagation remains stable while scoring evolves independently.
- Results identify the exact strategy and version that produced them.
- Pure strategies are deterministic and straightforward to test.

### Negative

- The application must carry strategy identity and explanation data through result contracts.
- Formula changes require explicit versions and regression fixtures.
- Runtime third-party scoring plugins remain unsupported until separately designed.

## Traceability

- Product source: [PRD §§3, 6, 8, and 10 — MVP 2](../../PRD.md)
- Architecture source: [ARD §7 — Extensible scoring](../../ARD.md)
- Follow-up: [Issue #19 — Impact scoring strategies](https://github.com/barbosalucas278/daedalus/issues/19)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
