---
name: qa-unit-test-creator
description: Use to write or change behavioural XCTest unit tests for iOS ViewModels, use cases, services and mappers, test first. Dispatch the qa-test-creator agent (Sonnet) with this skill named; never write tests on Haiku.
---

# QA unit test creator

Dispatch `qa-test-creator` with: "Follow `qa-unit-test-creator`", the ACs (with ticket keys) and the production files.

## The test is the spec
- One test file per production type, in the unit test target's folder: `ReceiptViewModel.swift` → `ReceiptViewModelTests.swift`. If the file exists, extend it; never start a parallel one.
- One `final class … : XCTestCase` per situation, all in that file, named `<Subject>Given<Situation>Tests`: `ReceiptViewModelGivenReceiptIsCachedTests`. Top-level classes only, so `-only-testing:<Target>/<Class>` finds them.
- One test per When/Then, named `test_<KEY without the dash>_<sentence in camelCase>`: `test_PROJ1234_openingItOnlineShowsItWithoutLoading`. Keep existing keys and append new ones (`test_PROJ1234_PROJ1300_…`).
- A test whose behaviour a later ticket changes is edited in place.

```swift
import XCTest
@testable import MyApp

final class ReceiptViewModelGivenReceiptIsCachedTests: XCTestCase {
    private let service = FakeReceiptService(cached: .sample)

    func test_PROJ1234_openingItOnlineShowsItWithoutLoading() {
        let viewModel = ReceiptViewModel(service: service, network: FakeNetworkStatus(isOnline: true))

        viewModel.open(purchaseId: "purchase-1")

        XCTAssertEqual(viewModel.state, .loaded(.sample))
    }
}
```

## Rules
- Write the test first. It must fail on today's code for the AC's reason.
- Cover the happy path, each edge, each failure, and, for anything holding user data in memory (singletons, preference managers, caches), that logout clears it.
- Hand-written fakes first, injected through the initialiser. If the type reaches a singleton with no seam, stop and report the seam needed; do not test through the network.
- Async: `XCTestExpectation` with a short timeout, or `async` test methods. No `sleep`.
- Global state (`UserDefaults`, shared managers) is saved in `setUp` and restored in `tearDown`.
- User-visible text is asserted through the same lookup the code uses (the key's localised string), never a literal copied from the design.
- New files go into the unit test target of the Xcode project: added lines only, no reordering.

## Return
Files changed; per test its name and tickets, and the commands from `qa-test-case-runner` with
`-only-testing:<Target>/<Class>`.
