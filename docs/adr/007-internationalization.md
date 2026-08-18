# ADR-007: Internationalization library and fallback policy

- Status: Proposed
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

MVP 1 requires Spanish and English UI while persisted keys, enums, and APIs remain stable English values. Localization must stay at presentation boundaries and cover validation and graph labels as well as React components.

## Decision

Use `i18next` with `en` and `es` catalogs. Detect the operating-system locale for the initial selection, persist the user's explicit preference as local UI state, and use English as the fallback locale because internal identifiers and canonical terminology are English.

All user-visible application strings use translation keys, including labels, errors, enum displays, relation descriptions, and graph tooltips. User-authored names and descriptions are never translated. Domain, persistence, interchange, and Tauri error contracts return stable codes or English identifiers; the UI maps them to localized messages.

Issue #8 must confirm the React binding, catalog loading strategy, pluralization conventions, and automated translation-coverage check before this ADR becomes Accepted.

## Alternatives considered

- **Hardcode visible strings and translate later:** accelerates scaffolding, but makes complete extraction and validation unreliable.
- **Persist localized enum values:** produces locale-dependent data and breaks interchange stability.
- **Use Spanish as fallback:** may suit an initial audience, but conflicts with the canonical English vocabulary and open-source extension surface.
- **Build a custom translation runtime:** avoids a dependency, but recreates fallback, interpolation, and pluralization behavior.

## Consequences

### Positive

- Persisted and interchange data remain locale-independent.
- UI locale can change without rebuilding domain state.
- Translation completeness can be tested against stable keys.

### Negative

- Every visible message needs a maintained key in both catalogs.
- Error contracts need stable codes rather than display-ready prose.
- The exact integration remains pending until the desktop UI exists.

## Traceability

- Product source: [PRD §§3, 6, 8, and 10 — localization](../../PRD.md)
- Architecture source: [ARD §§4 and 9](../../ARD.md)
- Follow-up: [Issue #8 — Localization and accessibility](https://github.com/barbosalucas278/daedalus/issues/8)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
