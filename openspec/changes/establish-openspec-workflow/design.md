## Context

The repository has a GitHub Project and issue hierarchy, but no checked-in delivery convention. OpenSpec is initialized locally; its active changes must become the technical record linked from the backlog.

## Goals / Non-Goals

**Goals:**

- Make each implementable change traceable from GitHub issue to OpenSpec artifacts, branch, PR, and merge to `develop`.
- Make TDD the mandatory implementation loop for behavior changes.
- Make `develop` the integration target for completed work.

**Non-Goals:**

- Automating GitHub state transitions or proving the temporal order of an agent's edits from CI.
- Changing the product behavior.

## Decisions

- **One feature issue maps to one OpenSpec change.** The change name is recorded in the issue and its proposal references the issue number. This keeps scope reviewable and avoids multi-feature PRs.
- **Changes remain active until their PR is merged into `develop`.** The archive is committed in that same PR, so the main specs and implementation stay synchronized.
- **Each behavior change follows red-green-refactor.** The agent writes a focused automated test, executes it to observe the expected failure, implements the minimal production change until it passes, then refactors only while the suite remains green.
- **PR description includes `Closes #<issue>`, the change directory, and concise TDD evidence.** Evidence records the tests added or modified and the final test command; it excludes internal prompts, discarded attempts, and incidental errors.
- **Merge to `develop` is the delivery event.** No QA approval or QA Project status is required for this workflow.
- **Create `develop` from the initial repository baseline before the first feature PR.** The current repository has no commits, so the integration branch must exist before it can be a PR target.

## Risks / Trade-offs

- [Manual cross-linking can drift] → Require the links in the change template and PR checklist.
- [Large features can outgrow one change] → Split the feature before implementation rather than use an oversized PR.
- [CI cannot prove a test failed before implementation] → Make the red step a required agent practice and request concise PR evidence rather than an exhaustive execution log.

## Migration Plan

1. Add the workflow convention and PR template.
2. Apply the convention to all new changes.
3. Reconcile any active change before it is implemented.
