# Daedalus — Product Requirements Document

**Tagline (provisional):** *Navigate your system landscape.*

## 1. Product summary

Daedalus is an open-source, desktop-first application for making a software ecosystem understandable. It lets teams model their system landscape—products, projects, components, and their dependencies—in a portable workspace, then progressively use that model to analyze operational impact and communicate business and technical flows.

The product is deliberately staged:

| Release | Outcome |
| --- | --- |
| MVP 1 — System Landscape | A reliable, navigable, editable map of the ecosystem. |
| MVP 2 — Impact Analysis | Explain what is affected when a component degrades or becomes unavailable. |
| MVP 3 — Business / Technical Flows | Connect system behavior to named user and business flows. |
| Future | Automation, integrations, architecture-as-code, and natural-language access. |

## 2. Problem

Teams commonly understand individual services but lack a trustworthy, current view of the system as a whole. Architecture knowledge ends up scattered across diagrams, repositories, documents, and people’s memory. During change planning or an incident, answering questions such as “what depends on this database?” or “which customer capability is affected?” is slow and unreliable.

Existing documentation tools often either produce static diagrams or require heavyweight enterprise adoption. Daedalus should provide a lightweight, local-first source of shared architectural understanding that starts with manual modelling and can later be automated.

## 3. Vision and principles

Daedalus helps a team navigate system complexity instead of merely drawing it.

- **Landscape first.** A correct, useful map is the prerequisite for intelligent analysis.
- **Local and portable.** One workspace represents one company or ecosystem and can be moved, backed up, and shared as a unit.
- **Explicit semantics.** Components and dependencies have typed, directed, inspectable meaning.
- **Human-editable interchange.** Users can work through UI forms or YAML/JSON without losing fidelity.
- **Progressive sophistication.** The core model must support later impact analysis, flows, automation, and integrations without making MVP 1 complex.
- **Open-source friendly.** Internal concepts are stable, English identifiers; visible UI is localized.

## 4. Target users

| User | Need |
| --- | --- |
| Engineering manager / tech lead | Maintain an understandable view of ownership and system boundaries. |
| Software architect | Model dependencies, communicate design, and identify architectural risk. |
| Developer | Quickly understand where a component fits and what it uses or serves. |
| SRE / incident responder | Trace expected blast radius during an outage (MVP 2). |
| Product / operations stakeholder | Understand which business capabilities rely on technical systems (MVP 3). |

## 5. Domain model

The organizational hierarchy is:

```text
Workspace
└── Product (optional)
    └── Project
        └── Component
```

A workspace represents a company or ecosystem. A project may belong directly to a workspace or to a product. A component belongs to exactly one project. Dependencies may cross project and product boundaries within the workspace.

Built-in component types for MVP 1:

`Application`, `Frontend`, `Backend`, `API`, `Database`, `Queue`, `Topic`, `Job`, `Library`, `External Service`, `Infrastructure`.

Users may define **custom component types**, which remain components in the core model. A repository is not a component: it is metadata attached to a component. Documentation is likewise an external reference attached to relevant domain entities, not a component type.

## 6. Scope by release

### MVP 1 — System Landscape

The first release provides an accurate, persistent landscape.

Prioritized use cases:

1. Create or open a portable workspace for one ecosystem.
2. Create, edit, and delete products, projects, components, component types, and dependencies using forms.
3. Attach metadata, repository links, and documentation references.
4. Visualize the system as an interactive graph; filter and inspect entities and relations.
5. Persist user-arranged graph positions.
6. Import and export the complete workspace in YAML and JSON.
7. Switch all UI between Spanish and English.

**Dependency relationship types:** `depends-on`, `consumes`, `reads-from`, `writes-to`, `publishes-to`, `subscribes-to`, `authenticates-with`, `deployed-on`.

Each dependency is directed and stores a severity of `REQUIRED`, `DEGRADED`, or `OPTIONAL`, plus extensible metadata such as a label, description, protocol or integration detail, and tags.

### MVP 2 — Impact Analysis

MVP 2 introduces an Impact Engine on top of the MVP 1 landscape. A user selects a component and an incident scenario—`UNAVAILABLE` or `DEGRADED`—and sees direct and transitive affected components, paths, explanations, and an impact score.

Business Criticality is user-maintained metadata expressing intrinsic business importance. It is not an Impact Score. The Impact Score is a calculated, scenario-specific result produced by an extensible `ImpactScoringStrategy`.

Prioritized use cases:

1. Simulate a component becoming unavailable or degraded.
2. Explain why each downstream component is affected, including dependency path and relation semantics.
3. Distinguish unavailable, degraded, and unaffected results.
4. Rank or summarize impact using a selected scoring strategy without overwriting business criticality.

### MVP 3 — Business / Technical Flows

MVP 3 lets users define named flows that connect business intent and technical execution. A flow can reference components and ordered interactions, then be displayed over the landscape and used as additional context for impact analysis.

Prioritized use cases:

1. Define a business or technical flow with a name, description, owner, and ordered steps.
2. Link each step to components and/or dependencies.
3. Visualize and inspect a flow as a focused path over the system landscape.
4. Identify flows affected by an MVP 2 incident simulation.

### Future

- Architecture-as-code and Git as an optional source of truth.
- Repository discovery and metadata enrichment.
- Observability integrations, initially including Datadog.
- Natural-language questions over the model.
- Integrations with adjacent engineering and documentation systems.
- Collaboration, conflict handling, and richer governance where justified.

## 7. MVP 1 experience

The primary view is a graph canvas with a clear hierarchy and directed dependency edges. The user can pan, zoom, select, filter, and reposition nodes. Selection opens an inspector with full metadata and actions. Forms provide a guided alternative to YAML/JSON editing; neither mode is subordinate.

Suggested navigation:

- **Landscape:** graph, filters, search, inspectors.
- **Catalog:** hierarchical list of products, projects, components, and custom types.
- **Import / Export:** validated YAML and JSON interchange.
- **Workspace settings:** workspace metadata, language, and other local preferences.

The product should make the origin, destination, relation type, severity, and meaning of each dependency legible without requiring users to infer direction from layout alone.

## 8. Functional requirements

### MVP 1

- Create, open, rename, and locally store multiple workspaces; workspace data is isolated by ecosystem.
- CRUD for products (optional grouping), projects, components, custom component types, dependencies, repository metadata, and documentation references.
- Enforce hierarchy ownership and prevent references to entities outside the active workspace.
- Provide all built-in component types and allow custom type creation.
- Require a source, target, relationship type, and severity for every dependency.
- Render a directed graph and persist node positions per workspace.
- Provide search and filters at least by hierarchy, component type, and dependency severity.
- Import/export all domain data and graph positions in JSON and YAML, with validation errors that identify the source location where possible.
- Store canonical data in SQLite; import files are interchange formats, not the active source of truth.
- Provide Spanish and English UI from day one; no user-visible UI text is hardcoded in UI components.
- Run on macOS and Windows as a desktop application.

### MVP 2

- Run an impact analysis using an entity, scenario, and configured/default scoring strategy.
- Return an explainable impact result for every traversed component and dependency path.
- Respect dependency severity and relation-specific propagation rules.
- Preserve simulation output separately from the landscape model unless a future feature explicitly saves it.

### MVP 3

- CRUD for business and technical flows and their ordered steps.
- Associate steps with existing model entities without duplicating component data.
- Show the flow and its affected status in the graph and detail views.

## 9. Non-functional requirements

- **Correctness:** domain invariants and import validation are enforced before persistence.
- **Portability:** a workspace can be exported, transferred, and imported without requiring a cloud account.
- **Usability:** common landscape editing is accessible via UI forms; text import/export remains first-class.
- **Performance:** a typical ecosystem graph should remain responsive while navigating, filtering, and editing. Define quantitative limits through implementation benchmarks before release.
- **Privacy:** MVP 1 has no required hosted backend or telemetry dependency.
- **Accessibility:** keyboard navigation, visible focus, and readable contrast should be considered in all primary workflows.
- **Maintainability:** a typed internal domain model, localized presentation layer, and testable pure application services.

## 10. Acceptance criteria

### MVP 1

1. A user can build the hierarchy Workspace → optional Product → Project → Component and view it in the catalog and graph.
2. A component can use each built-in type or a custom type; a repository and documentation link can be attached without creating false components.
3. A user can create a directed dependency with every supported relation type and one of the three severities.
4. The inspector clearly identifies source, target, direction, relation, severity, and metadata.
5. Moving nodes and reopening the workspace retains their graph positions.
6. A complete workspace exports to JSON and YAML, and importing either format reproduces the same valid landscape.
7. Invalid imports do not partially persist data and give actionable validation feedback.
8. Switching Spanish/English updates all visible application text; internal persisted enum values remain English.
9. The application runs as a supported desktop build on macOS and Windows.

### MVP 2

1. Simulating `UNAVAILABLE` and `DEGRADED` produces distinct, explainable results when relation/severity rules differ.
2. Each affected component reports a causal path from the simulated component.
3. Business Criticality remains unchanged by analysis; Impact Score is calculated by a strategy.

### MVP 3

1. A user can create a named flow with ordered, linked steps and view it over the landscape.
2. An impact simulation identifies affected flows through their referenced components or dependencies.

## 11. Explicitly out of scope

| Release | Out of scope |
| --- | --- |
| MVP 1 | Automated repository discovery, Git synchronization, production telemetry, incident computation, flows, hosted collaboration, natural-language querying. |
| MVP 2 | Flow authoring, observability ingestion, architecture-as-code as source of truth, automated dependency discovery. |
| MVP 3 | Mandatory cloud synchronization, real-time multi-user collaboration, vendor-specific operational automation. |

