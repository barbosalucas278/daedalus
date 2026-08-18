# Daedalus — Architecture Requirements Document

## 1. Purpose and architectural boundary

This document translates the Daedalus product definition into an implementation-oriented architecture. It governs the desktop application for MVP 1 while reserving explicit seams for the Impact Engine (MVP 2) and Flows (MVP 3).

**Stack:** Tauri, React, TypeScript, SQLite, Cytoscape.js. Target platforms are macOS and Windows.

Daedalus is local-first. SQLite is the canonical source of truth for an open workspace. YAML and JSON are validated import/export formats. A future architecture-as-code/Git mode may change the configured source-of-truth policy, but must not be assumed in MVP 1.

## 2. High-level architecture

```text
React UI (localized) ── commands/queries ──> Application services
     │                                           │
     └── Cytoscape graph adapter                 ├── Domain model + validation
                                                 ├── SQLite repositories
                                                 ├── Import/export adapters (YAML/JSON)
                                                 └── Future: Impact Engine / Flow services
Tauri shell ── filesystem, SQLite lifecycle, native packaging
```

The UI never owns canonical domain state. It reads view models and invokes typed commands. Application services enforce invariants and transaction boundaries. The domain model must not depend on React, Cytoscape, Tauri, SQL, or translation libraries.

## 3. Modules

| Module | Responsibility |
| --- | --- |
| `domain` | Entities, value objects, enums, invariants, dependency semantics, pure impact interfaces. |
| `application` | Use cases, command/query contracts, validation orchestration, transactions, DTO mapping. |
| `persistence` | SQLite schema, migrations, repository implementations, workspace lifecycle. |
| `interchange` | Versioned YAML/JSON schema, parsing, validation, import/export mapping. |
| `graph` | Convert domain/query data to Cytoscape elements; layout and persisted-position handling. |
| `ui` | React screens, forms, state, localization, accessibility. |
| `desktop` | Tauri commands, filesystem boundaries, packaging, native error translation. |
| `impact` (MVP 2) | Traversal, propagation, explanation paths, strategies. |
| `flows` (MVP 3) | Flow definitions, ordered steps, projection and impact linkage. |

## 4. Core domain model

All identifiers, enum values, serialized keys, and APIs are English. Display labels are localized only at presentation boundaries.

```text
Workspace
  id, name, description?, metadata
  Products[] (optional grouping)
  Projects[] (may have productId = null)

Product
  id, workspaceId, name, description?, metadata

Project
  id, workspaceId, productId?, name, description?, metadata

Component
  id, workspaceId, projectId, typeId, name, description?
  businessCriticality?, repositoryMetadata?, metadata, tags

ComponentType
  id, workspaceId? (null = built-in), key, displayName?, kind = BUILT_IN | CUSTOM

Dependency
  id, workspaceId, sourceComponentId, targetComponentId
  relationType, severity, label?, description?, metadata, tags

DocumentationReference
  id, workspaceId, entityType, entityId, title, url, description?

GraphPosition
  workspaceId, componentId, x, y
```

`RepositoryMetadata` is an embedded value or separate one-to-one record (for example URL, provider, default branch, path, notes). It must never be represented as a `Component`. Documentation references can attach to workspace, product, project, component, dependency, and later flow entities.

`BusinessCriticality` is optional user-authored metadata. Its scale must be explicit and stable (recommended initial enum: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`). It expresses business importance only; it is not computed by the Impact Engine.

### Built-in types

Built-in `ComponentType.key` values:

`APPLICATION`, `FRONTEND`, `BACKEND`, `API`, `DATABASE`, `QUEUE`, `TOPIC`, `JOB`, `LIBRARY`, `EXTERNAL_SERVICE`, `INFRASTRUCTURE`.

Custom types have workspace scope and a stable generated id/key. They cannot overwrite built-ins.

## 5. Persistence and workspace lifecycle

Each workspace is a portable unit for one company/ecosystem. It must have a unique identity and isolated SQLite database. The initial implementation may use a workspace directory containing a SQLite database and optional manifest, but its external interchange contract—not an internal file layout—is the portability guarantee.

SQLite requirements:

- Foreign keys enabled; use migrations and transactions.
- `projects.product_id` nullable; all other hierarchy keys required as appropriate.
- Component and dependency workspace IDs are retained or derivable and checked to prevent cross-workspace links.
- Uniqueness constraints prevent ambiguous names within a chosen parent scope where product design requires it.
- Graph positions are persisted independently of Cytoscape runtime state.
- Deletes must be deliberate: reject deletion with dependents or offer an application-service operation that removes/reassigns all affected references atomically.

Recommended tables: `workspaces`, `products`, `projects`, `component_types`, `components`, `dependencies`, `documentation_references`, `graph_positions`, plus migration metadata. Add `repository_metadata` only if embedded JSON/value columns are insufficient for queries.

## 6. Dependency semantics and direction

Every edge has the form:

```text
sourceComponent --relationType--> targetComponent
```

The source is the component performing the named action or relying on the target. The source therefore has a dependency on the target, whether direct (`depends-on`) or implied by a more specific relationship.

| Relation | Directional statement | Example |
| --- | --- | --- |
| `depends-on` | source depends on target | API → Library |
| `consumes` | source consumes target | Backend → External Service |
| `reads-from` | source reads from target | API → Database |
| `writes-to` | source writes to target | API → Topic |
| `publishes-to` | source publishes to target | Backend → Queue |
| `subscribes-to` | source subscribes to target | Job → Topic |
| `authenticates-with` | source authenticates with target | Frontend → API |
| `deployed-on` | source is deployed on target | API → Infrastructure |

The system must not infer reverse edges as stored records. Query projections may create a reverse adjacency index for impact traversal. UI arrows and inspectors must preserve this meaning.

`DependencySeverity`:

- `REQUIRED`: inability of target normally makes the source unavailable.
- `DEGRADED`: target failure normally leaves source functional with degraded behavior.
- `OPTIONAL`: target failure normally leaves source available; reduced optional capability may be reported separately.

Dependency metadata is extensible JSON plus indexed common fields (`label`, `description`, `tags`). Future standard fields can include protocol, operation, timeout, owner, and SLA, without changing the relationship identity.

## 7. Impact Engine (MVP 2)

### Inputs and outputs

Input: workspace snapshot, initiating `componentId`, scenario (`UNAVAILABLE` or `DEGRADED`), and an `ImpactScoringStrategy`.

Output is an ephemeral `ImpactAnalysisResult` containing per-component status, causal explanation path(s), contributing dependencies, score data, strategy identity/version, and diagnostic warnings. Do not mutate the landscape or Business Criticality.

Recommended component result states: `UNAVAILABLE`, `DEGRADED`, `UNAFFECTED`; optionally add `OPTIONAL_IMPACT` only when UX needs it.

### Propagation rule

Impact propagates from a failed target to components that point to it, therefore traversal follows the reverse adjacency of stored dependency directions. Use a work queue and retain predecessors for explanation paths. Detect cycles and process a component only when a newly found path produces a stronger impact status.

Initial default rule matrix:

| Target scenario / dependency severity | Result on source |
| --- | --- |
| `UNAVAILABLE` + `REQUIRED` | `UNAVAILABLE` |
| `UNAVAILABLE` + `DEGRADED` | `DEGRADED` |
| `UNAVAILABLE` + `OPTIONAL` | `UNAFFECTED` (record optional effect if desired) |
| `DEGRADED` + `REQUIRED` | `DEGRADED` |
| `DEGRADED` + `DEGRADED` | `DEGRADED` |
| `DEGRADED` + `OPTIONAL` | `UNAFFECTED` |

Relation type may refine this default in a future policy. For MVP 2, all supported relations use the shared matrix unless an explicit, tested relation policy is introduced. Never silently encode semantics in UI code.

When multiple paths reach a component, take the strongest status (`UNAVAILABLE` > `DEGRADED` > `UNAFFECTED`) and retain enough paths to explain the winning result. Traversal limits and warnings should protect against unexpectedly huge graphs.

### Extensible scoring

Define an application/domain contract such as:

```ts
interface ImpactScoringStrategy {
  readonly id: string;
  readonly version: string;
  score(input: ImpactScoringInput): ImpactScore;
}
```

`ImpactScoringInput` includes the analysis graph/result and read-only component metadata (including Business Criticality). `ImpactScore` includes numeric value only when appropriate, contributing factors, and an explanation suitable for the UI. The MVP default strategy may score based on affected status, reach, and business criticality, but must keep its formula documented and replaceable. A strategy cannot write entity metadata or perform network calls.

## 8. Import and export

Export is a versioned snapshot of all portable workspace data: workspace metadata, product/project hierarchy, component types, components, dependencies and metadata, repository metadata, documentation references, and graph positions. JSON and YAML must map to the same canonical interchange schema.

Rules:

- Include `schemaVersion`; reject unsupported major versions with a clear error.
- Validate structure, IDs, references, type keys, relation types, enums, and workspace-boundary invariants before changing SQLite.
- Parse and validate fully, then import in one transaction; failure leaves the current workspace unchanged.
- Preserve stable IDs on round-trip. Define collision behavior explicitly (recommended: create-new workspace by default; future merge is separate).
- Do not serialize local UI-only state beyond intentional portable settings and graph positions.

## 9. UI and i18n

Use `i18next` (or equivalent) with `en` and `es` catalogs from MVP 1. UI components reference translation keys only; visible strings, validation messages, labels, relation descriptions, and enum displays all come from catalogs. Enforce this with code review and, if practical, lint/test checks.

Persisted and internal values remain English, for example `REQUIRED`, `UNAVAILABLE`, `reads-from`. Translation maps them to labels such as *Required/Requerida*. User-entered entity names/descriptions are not translated.

The graph adapter should expose semantic element data (`entityId`, kind, relationType, severity, source/target) and apply UI localization only to labels/tooltips. Graph positions are loaded before render and written through a debounced application command after user movement.

## 10. Suggested repository structure

```text
daedalus/
├── src/
│   ├── domain/
│   ├── application/
│   ├── persistence/
│   ├── interchange/
│   ├── graph/
│   ├── impact/              # MVP 2 implementation; interfaces may exist earlier
│   ├── flows/               # MVP 3
│   ├── ui/
│   │   ├── features/
│   │   └── components/
│   ├── i18n/
│   │   ├── en.json
│   │   └── es.json
│   └── shared/
├── src-tauri/
│   ├── src/
│   └── migrations/
├── docs/
│   └── adr/
├── tests/
│   ├── domain/
│   ├── application/
│   ├── interchange/
│   └── e2e/
└── PRD.md
```

Exact language ownership between TypeScript and Rust should be decided early and documented. A pragmatic boundary is TypeScript domain/application logic initially, with Tauri/Rust responsible for native lifecycle, filesystem mediation, and SQLite access exposed through narrow typed commands. Do not expose raw SQL or filesystem paths directly to the UI.

## 11. Internal contracts

- **Commands:** typed, mutation-oriented requests returning an entity/read DTO; all mutations validate and transact.
- **Queries:** read-only DTO projections shaped for catalog, inspector, and graph; they avoid leaking persistence rows.
- **Repositories:** operate on domain entities/value objects and accept a transaction context where needed.
- **Tauri boundary:** explicit allowlisted commands with serializable request/response contracts; map errors to stable application error codes.
- **Interchange adapter:** `parse → validate → canonical snapshot`; never bypass application/domain validation.
- **Graph adapter:** one-way projection from query DTOs to Cytoscape elements; graph events become typed UI/application commands.

## 12. Testing and quality gates

- Unit-test domain invariants, relationship semantics, propagation matrix, scoring strategies, and cycle handling.
- Test application services for transactions, invalid cross-workspace references, and deletion behavior.
- Contract-test JSON/YAML round trips, schema versioning, malformed files, and atomic import failure.
- Integration-test SQLite migrations and repositories against a real temporary database.
- Component-test forms and localization; verify translation coverage for supported locales.
- End-to-end test the essential MVP 1 path: create workspace → model landscape → arrange graph → close/reopen → export/import.
- For MVP 2, use deterministic fixture graphs proving paths, competing statuses, cycles, and score explanations.

## 13. ADRs to create before or during implementation

1. **ADR-001: Workspace storage boundary and portable package layout.**
2. **ADR-002: SQLite access ownership (Rust vs TypeScript abstraction).**
3. **ADR-003: Canonical JSON/YAML interchange schema and version policy.**
4. **ADR-004: Entity ID format and import collision policy.**
5. **ADR-005: Default impact propagation matrix and status ordering.**
6. **ADR-006: Default ImpactScoringStrategy formula and versioning.**
7. **ADR-007: i18n library, fallback locale, and translation-key conventions.**
8. **ADR-008: Component deletion/reassignment policy.**

## 14. Extensibility constraints

Future work must extend the core model rather than fork it. Architecture-as-code/Git may provide an alternate synchronized source of truth; repository discovery may propose components/dependencies but must pass the same validation workflow; Datadog and other observability integrations may attach runtime signals without redefining topology; natural-language queries should operate through safe, typed queries over the canonical model; and integrations should remain adapters outside the domain core.

New relation types, component types, scoring strategies, metadata fields, and flow step kinds require explicit semantic documentation and migration/interchange consideration. The key architectural promise is that a portable Daedalus workspace remains intelligible without any external integration.
