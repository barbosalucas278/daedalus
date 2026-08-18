## 1. Quality contract

- [ ] 1.1 Define required local scripts for format, lint, typecheck, test, and build once the desktop scaffold exists.
- [ ] 1.2 Define the OpenSpec validation command and expected validation evidence for each PR.
- [ ] 1.3 Define concise TDD evidence: tests added or modified, expected red-step confirmation, and final passing test command.

## 2. CI workflow

- [ ] 2.1 Add a GitHub Actions workflow that runs on pull requests targeting `develop`.
- [ ] 2.2 Implement ordered execution of OpenSpec validation and available project checks.
- [ ] 2.3 Make unsupported future checks explicit rather than silently skipping them.

## 3. Delivery evidence

- [ ] 3.1 Add PR checklist entries for checks, issue closure, OpenSpec archive, and concise TDD evidence.
- [ ] 3.2 Verify a failing check blocks merge to `develop`.
