# Contributing to Daedalus

## Delivery workflow

Each feature issue is implemented through one OpenSpec change:

1. Create or link the change under `openspec/changes/<change-name>/` from its GitHub issue.
2. Review the proposal, design, and tasks before implementation.
3. Work on a feature branch created from `develop`.
4. For behavior changes, use TDD: write and execute a focused failing test, make the smallest production change that passes it, then refactor with the suite green.
5. Open a pull request to `develop`. The PR must include `Closes #<issue>`, the OpenSpec change path, test evidence, and validation results.
6. Archive the OpenSpec change in the PR that merges its implementation into `develop`.

GitHub Project statuses are `Todo`, `In Progress`, `In Review`, and `Done`. A merged PR closes its linked feature issue; no QA approval status is required.

## TDD evidence

Keep evidence concise. State the test added or updated, confirm that its first execution failed for the intended missing behavior, and provide the final passing test command. Do not record internal prompts, discarded tests, or incidental errors.
