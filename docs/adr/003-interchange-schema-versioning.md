# ADR-003: Canonical JSON/YAML interchange schema and version policy

- Status: Proposed
- Date: 2026-08-18
- Owners: Issue #20 / `document-foundational-adrs`
- Supersedes: None
- Superseded by: None

## Context

JSON and YAML must round-trip the same complete workspace while SQLite remains canonical active storage. Version handling must prevent a newer or incompatible file from partially changing a workspace.

## Decision

Define one language-neutral canonical snapshot model. JSON and YAML are two encodings of that same model; neither format has format-specific domain fields or semantics. YAML is restricted to values representable by the JSON data model, without custom tags.

Every snapshot carries a `schemaVersion` in `MAJOR.MINOR` form. A major version denotes an incompatible shape or semantic change. A minor version denotes a backward-compatible addition. Export writes the current version. Import rejects unsupported major versions and newer unsupported minor versions, migrates explicitly supported older versions in memory, then validates the resulting canonical snapshot before opening a transaction.

The exact schema document, validation library, canonical field ordering, and supported-version table remain owned by Issue #18. This ADR becomes Accepted when that feature supplies contract tests proving equivalent JSON/YAML parsing and faithful round trips.

## Alternatives considered

- **Independent JSON and YAML schemas:** allows format-specific features, but creates drift and makes fidelity impossible to reason about.
- **No explicit version:** simplifies initial files, but makes later compatibility and actionable errors ambiguous.
- **Accept unknown versions and ignore fields:** appears forward-compatible, but can silently lose data when re-exporting.
- **Use interchange files as active storage:** conflicts with the accepted SQLite canonical storage boundary.

## Consequences

### Positive

- Both formats share validation, migrations, and domain mapping.
- Unsupported inputs fail before persistence changes.
- Version evolution is explicit and testable.

### Negative

- The project must maintain version migrations and fixtures.
- YAML features outside the JSON data model are intentionally unavailable.
- Newer minor versions are rejected until their additions are understood, favoring fidelity over permissive parsing.

## Traceability

- Product source: [PRD §§3, 6, 8, and 10](../../PRD.md)
- Architecture source: [ARD §§1, 8, 11, and 12](../../ARD.md)
- Follow-up: [Issue #18 — JSON/YAML interchange](https://github.com/barbosalucas278/daedalus/issues/18)
- OpenSpec: `openspec/changes/document-foundational-adrs/`
