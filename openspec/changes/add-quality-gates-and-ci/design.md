## Context

Daedalus has no application scaffold or CI yet. The first CI design must support incremental adoption: it cannot require tools that the repository has not introduced, but it must define the checks every implementation change will eventually satisfy.

## Goals / Non-Goals

**Goals:**

- Run deterministic checks on pull requests targeting `develop`.
- Validate OpenSpec planning artifacts together with code quality and tests.
- Publish a clear pass/fail signal before merge to `develop`.
- Support the mandatory TDD loop with fast, focused automated tests.

**Non-Goals:**

- Deploying the desktop application.
- Proving the red step mechanically after the final implementation is present.

## Decisions

- **Use GitHub Actions as the CI runner.** It is co-located with the repository and directly reports PR checks.
- **Make validation layered:** OpenSpec validation first, then dependency install, formatting/lint, typecheck, tests, and build as each script becomes available.
- **Target `develop` PRs.** Feature branches remain integration candidates until checks pass and the PR is merged.
- **Keep the workflow strict but bootstrappable.** The first implementation change adds the actual package scripts; CI then makes them required.
- **Treat TDD evidence as a PR convention, not a CI assertion.** CI verifies final tests are green; the agent records the focused tests that drove the change and confirms the expected initial failure without retaining an internal trial log.

## Risks / Trade-offs

- [A workflow added before scripts exist will fail] → Stage its implementation after the desktop scaffold introduces the scripts.
- [Platform-specific desktop builds are costly] → Start with a portable verification job; add macOS/Windows packaging checks when the Tauri build exists.

## Migration Plan

1. Add project scripts and a CI workflow with the desktop scaffold.
2. Require CI success for PRs to `develop`.
3. Add platform packaging jobs when supported by the application build.
