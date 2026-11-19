---
name: qa-gate
description: Use before calling any iOS change done, fixed or ready, and for any bug fix or new behaviour. Runs static checks, has tests created and seen failing, then passing, and reports each acceptance criterion. Nothing is done until this gate is green.
---

# QA gate

Runs in the main session. No green, no done.

## Steps
1. **Static checks.** Run the linter on the changed files (for example `swiftlint lint --config .swiftlint.yml <changed .swift files>`) and a `build-for-testing` through `qa-test-case-runner`. Fix what they report in the files you changed before going on. Warnings that CI lets pass are not blockers.
2. **Pick levels** from the change:
   - ViewModel, use case, service, mapper → unit
   - view controller, view or cell → UI
   - screen data or an API call → unit, with the service faked
   - a user journey across screens → E2E (for example Maestro flows)
3. **Create.** Invoke `qa-test-case-creator` with the ticket key and the changed files.
4. **Red.** Run the returned commands through `qa-test-case-runner` on the code before the fix. Every new test must fail, and its failure must match the acceptance criterion (AC). A test that passes already, or fails for another reason, goes back to its creator.
5. **Fix.** Make the production change. Do not edit tests in this step.
6. **Green.** Run the same commands again through `qa-test-case-runner`, independent levels in parallel.
7. **Report.** For each AC of the ticket, find its tests by ticket key: `grep -rnE '<KEY>|<KEY without the dash>' <test folders>`. XCTest names carry the key without the dash (`test_PROJ123_…`); E2E flows name it in their header comment. Print:

| AC | Tests | Result |
|---|---|---|

The change is **not done** if any AC has no passing test, any test failed, any test was flaky, or a test only compiled and never ran. A suite the runner could not run is reported as **not run**, never as passing.

## Bugs
Order of evidence: a failing automated test first; then simulator logs; then a screenshot; then asking the developer.

## Scope
This gate is a skill. CI does not enforce the AC report, and a PR opened by a person does not pass through it.

## Memory
The runner and creators return "Proposed memory entries". Show them to the user and add the ones they accept to the repo's own notes (for example `CLAUDE.md`): short, dated, deduplicated, no secrets.
