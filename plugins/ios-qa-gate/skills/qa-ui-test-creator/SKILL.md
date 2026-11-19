---
name: qa-ui-test-creator
description: Use to write or change iOS view controller and view tests in XCTest: every state rendered from a ViewModel fake, what each control asks the ViewModel to do, accessibility and Dynamic Type. Dispatch the qa-test-creator agent (Sonnet) with this skill named.
---

# QA UI test creator

Dispatch `qa-test-creator` with: "Follow `qa-ui-test-creator`", the ACs, the view controller or view file, and the design frame links.

## What to test
- Load the real view controller (from its storyboard or initialiser), give it a ViewModel fake in a known state, put it in a `UIWindow` and call `loadViewIfNeeded()` / `layoutIfNeeded()`. No backend.
- One `XCTestCase` class per state, named `<Screen>Given<State>Tests`: `ReceiptVCGivenNoReceiptYetTests`, `…GivenTheReceiptIsLoadingTests`, `…GivenLoadFailedTests`, `…GivenAReceiptIsLoadedTests`.
- Every control asks the ViewModel for the right thing: send the control's action (`sendActions(for: .touchUpInside)`) and assert the calls the fake recorded.
- Disabled controls ask for nothing.
- Accessibility: icon-only buttons have an `accessibilityLabel`; touch targets are at least 44×44 pt.
- Dynamic Type: override the trait collection to `.accessibilityExtraExtraExtraLarge` and feed the longest localised string; nothing truncated (`numberOfLines`, label fits its bounds).
- Find views by `accessibilityIdentifier` constants, never by visible text. Reuse the identifiers your E2E flows already use.
- The UI must not call the network or a singleton service. If it does, stop and report the seam needed (the repo's MVVM rules).

## Screenshots
Without a snapshot-testing library there are no image baselines. For a visual check, return the steps to show the state on a simulator and capture it with `xcrun simctl io <udid> screenshot`; the main session shows it to a person beside the design frame.

## Return
Files changed (each added to the test target); per test its name and tickets; the commands from `qa-test-case-runner` with `-only-testing:<Target>/<Class>`.
