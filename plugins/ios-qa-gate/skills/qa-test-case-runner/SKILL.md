---
name: qa-test-case-runner
description: Use to run iOS tests that already exist (XCTest unit and UI, and E2E flows such as Maestro) and get a raw report. Dispatch the qa-test-runner agent (Haiku) with the commands; never run the suite on a stronger model to save a round trip.
---

# QA test case runner

Dispatch the `qa-test-runner` agent (Haiku) with:
- the exact commands, grouped by level;
- the simulator UDID;
- the preconditions for each E2E flow, copied from its header comment.

The agent runs them, saves evidence and reports. It never edits files or diagnoses.

## Commands by level

Use the app's own scheme, test target, configuration and bundle id; match what CI runs.

```bash
# Build once per code change
xcodebuild -workspace <App>.xcworkspace -scheme <Scheme> -configuration <Configuration> \
  -destination "id=<udid>" -derivedDataPath build/dd \
  CODE_SIGNING_ALLOWED=NO \
  build-for-testing

# If the app asks for permissions (microphone, camera...) at launch, an unanswered alert breaks tests.
# A grant made before the first install is dropped, so install first.
xcrun simctl install <udid> build/dd/Build/Products/<Configuration>-iphonesimulator/<App>.app
xcrun simctl privacy <udid> grant microphone <bundle-id>
xcrun simctl privacy <udid> grant camera <bundle-id>

# Unit and UI: one class or one method per -only-testing
xcodebuild -workspace <App>.xcworkspace -scheme <Scheme> -configuration <Configuration> \
  -destination "id=<udid>" -derivedDataPath build/dd \
  CODE_SIGNING_ALLOWED=NO \
  -only-testing:<TestTarget>/<Class>[/<testMethod>] \
  -resultBundlePath build/qa-runs/<yyyyMMdd-HHmmss>/<label>.xcresult \
  test-without-building
```

## E2E (for example Maestro)

Run each flow against a build of the code under test, with the device pinned:

```bash
MAESTRO_DEVICE=<udid> maestro test <flow>.yaml
```

- Install the build under test before every Red and every Green run, so the flow tests the right code.
- If a flow needs secrets or test accounts that are not set up, it is **not run**; say so.

## Simulator rules
- Use the UDID you were given. With none given, or one that does not exist, stop and ask. Never pick one yourself: other sessions run their own simulators, and a booted one you were not given belongs to someone else.
- One simulator per run. Always pass the device to the E2E tool and `id=` to `xcodebuild`.
- Boot only when asked: `xcrun simctl boot <udid>`, then `xcrun simctl bootstatus <udid> -b`. Never a fixed sleep.
- Apply only the preconditions you were given. Never plant state with `simctl spawn … defaults write`.

## Running
- Run independent levels in parallel only when they use different simulators; one `xcodebuild` per simulator at a time.
- A suite that can outlast the Bash timeout runs in the background writing to a log; wait with an `until` loop on the process.
- If a command fails, run it once more. Fail then pass is **flaky**. Fail twice is **fail**. Never a third run.
- A run where `xcodebuild` reports no executed tests (for example "Unable to find a destination") is **not run**, never pass.

## Evidence
Copy into `build/qa-runs/<yyyyMMdd-HHmmss>/` (check that `build/` is git-ignored, and never stage it):
- the `.xcresult` bundles and the `xcodebuild` log of each command
- the E2E artifacts
- `xcrun simctl spawn <udid> log show --last 15m --style compact > simulator.log` after each simulator run

## Report
For every command:
- the command and its exit code;
- pass / fail / flaky / not run counts (`xcrun xcresulttool get test-results summary --path <bundle>`);
- each failing or flaky test as `Class › testMethod`, with the first 10 lines of its failure, quoted;
- the evidence folder.

End with **Proposed memory entries**: commands that needed a flag, simulator quirks, flaky tests and what they did.
