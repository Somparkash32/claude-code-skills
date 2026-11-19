---
name: mobile-pr-review
description: Use when reviewing a pull request, branch or diff in a mobile app repo (iOS Swift/UIKit/MVVM or Android Kotlin/Compose/MVI), or when asked to "review this PR". Detects the platform, applies the repo's own rules files, checks the change against its ticket or spec, and runs platform risk checks, a code smell pass and an over-engineering pass. Read-only, posts nothing unless told to.
---

# Mobile PR review

Review a PR, branch or diff. Read-only: never commit, push, or post PR comments unless the user says so.

## Platform
Detect from the repo root: `*.xcodeproj` / `*.xcworkspace` = **iOS**, `build.gradle.kts` = **Android**. Read that repo's `CLAUDE.md`: its numbered rules are the project rules for this review. Apply only the matching platform section below. For any other stack, skip the platform sections and use the rest.

## Rules files
`CLAUDE.md` is not the only one. Also read, when present: `AGENTS.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`, and every doc a `CLAUDE.md` rule points to. A break of any of them is a project-rule finding; name the file and the rule.

Read the lint config too (`.swiftlint.yml`, ktlint/detekt config). Do not report what the linter or CI already blocks as an error: CI catches it. Lint warnings that still pass CI are fair game.

## Steps
1. Get the target: a PR number (`gh pr view <n>`, `gh pr diff <n>`) or `git merge-base origin/main HEAD` to `HEAD` (plus uncommitted and untracked files when reviewing a working tree). Check it before anything else: the ref resolves (`git rev-parse <ref>`) and the diff is not empty. If either fails, stop and tell the user. Write down the head commit and the commit list (`git log --oneline <base>..HEAD`). A review only counts for that commit: if code changes after it, review the new changes again.
2. Find the spec, in this order, and stop at the first hit:
   1. A ticket ID (for example `ABC-123`) in the PR title, branch name or commit messages. Fetch it from your tracker (Jira, Linear, GitHub issue...), with its parent when it is a sub-task.
   2. A spec path or text the user gave.
   3. A spec file under `docs/` that matches the branch or feature name.
   4. Ask the user. If they say there is none, skip the spec pass and write "No spec available" in the report. Never guess the spec from the diff.
3. Run **two passes**. If you can dispatch subagents, give each pass its own fresh subagent and send both in one message so they run in parallel. A subagent does not see this file, so paste what it needs into its prompt: the diff command, the commit list, and its own brief below. If you cannot dispatch subagents, run both passes one after the other.
   - **Spec pass** (tag findings `[Spec]`): paste the ticket text and the "Severity" section. Answer three questions, and quote the ticket line for each finding:
     1. What did the ticket ask for that is missing or only half done?
     2. What did the PR add that nobody asked for (scope creep)?
     3. What looks done but is done wrong?
   - **Rules pass** (tag findings `[Rules]`): paste the whole "Severity" and "Rules for the reviewer" sections, the repo's `CLAUDE.md` rules, and the list of rules files found above. Apply every rule and smell, strictly. Use only the diff and the rules. Do not use the spec, earlier review notes or the author's conclusions, so you cannot copy their severity or their "deferred" decisions.
4. Run the "Repo-wide checks" below. A diff alone cannot show copies that live in other files.
5. Over-engineering pass: the "Over-engineering lens" below. Report under Code Smells.
6. If the change touches auth, tokens, networking, storage, logging, deep links, WebViews, enums or server data, apply the "Data-safety checklist" below.
7. Confirm every finding yourself before reporting it, and keep the proof (see Evidence). Do the same for bot claims (CodeRabbit, other agents). A claim with no proof is not a finding: drop it, or ask it as a polite question and say what you did not verify.
8. Report in the output format below. Ask before posting anything to GitHub.

## Severity (do not soften)
- A broken project rule (repo `CLAUDE.md`) in code the PR adds, rewrites or copies is at least **Medium**. It is **blocking** when the PR adds a new copy of existing knowledge, a new raw-string match or hardcoded string the rules forbid, or new logic or a fix without a test on the real path.
- Never lower a finding because it is a "pre-existing pattern", "deferred", "by design", "out of scope" or "the spec says so". Report it at its real severity and let the owner rule on it. Old code the PR did not touch is out of scope. Code the PR rewrites or copies is in scope.
- Overall risk is **Low** only when the rules pass found no rule break, the spec pass found nothing missing or wrong, and every fix has a test that would fail without it. A clean result on one pass never hides a failure on the other.
- A missing or wrong ticket requirement is at least **Medium** and blocking. Scope creep is at least **Low**, and **Medium** when it changes user-visible behaviour.
- A clean bot result (CodeRabbit, Semgrep, Trivy) or a green CI run does not lower the risk.
- Anything the spec, a ledger or an earlier pass marked deferred must show up under "Deferred", with its reason.

## Repo-wide checks (run yourself, list what you find)
- **Copies:** for every new mapping, table, switch or helper (wire values, spellings, icons, titles, aliases), pick 3-4 of its literals and search the whole repo (`git grep -n`). List every copy as `file:line`. Two or more copies is a finding. A new "shared" helper that leaves the old copies in place is one more copy.
- **Same data, many screens:** when the PR makes something depend on a flag or variant, search for every place that shows or sends it. List which screens follow the flag and which do not.
- **Missing keys:** iOS: a new localisation key must exist in the bundled catalogue. Android: user-visible text must be in `res/values/strings*.xml`.
- **Flag reads:** say how each flag is read (live, or latched once) and when a change applies.

## Evidence (required for every finding)
- Each finding carries proof you checked yourself: the exact line (quote it, short), the call path (A calls B at `file:line`), or a test or log result.
- Read the real code at the PR's commit. Do not trust a summary, the PR text, or a bot's claim.
- Claims about behaviour (crash, freeze, stale data, slow) need a way to see it. If you only guess, use "Potential issue" and say so.
- Say plainly what you did not verify. You may claim only what you ran or read in this task: "should pass" and "probably" are not results.

## Posting to GitHub (only when the user says to post)
- Post findings as **inline comments**, one per finding, on the exact file and line of the PR head commit. One review call: `gh api repos/<owner/repo>/pulls/<n>/reviews --input <json>` with `commit_id`, `event` (`REQUEST_CHANGES` or `COMMENT`), a one-line `body`, and `comments: [{path, line, side: "RIGHT", body}]`. Never post the whole report as one big review body.
- Each comment is **2-3 short lines** in simple everyday words. No fancy words, no labels like "Evidence:" or "Architecture note". Say what is wrong, what could happen, what to do. Put `Blocking:` at the start only when it blocks.
- A finding with no single line (missing test, repo-wide copy): comment on the closest changed line, or put it as one short bullet in the review body.
- Code smells go inline too, started with `Small thing:`.
- The review body is one or two lines (verdict and count). Do not paste the Missing Test or Not Verified lists.
- If a long review was already posted by mistake, shorten its body with `gh api -X PUT repos/<owner/repo>/pulls/<n>/reviews/<id> -f body=...` instead of leaving a duplicate.

## Writing style
- Polite, professional, positive. Talk about the code, never the person. Suggest instead of ordering: "Could we...", "It would be safer to...".
- Start the summary with one honest, specific good point if there is one. Never invent praise.
- Every finding says what to do next.
- Simple everyday words. One or two lines per finding: what and where, what to do, the evidence.
- Do not repeat the same point in the summary and in the findings.

## Rules for the reviewer

### Project rules
Every rule in the repo's `CLAUDE.md` is a finding when broken. Good defaults for both platforms, when the repo has no stricter rule: no WHAT comments (only a non-obvious WHY); user-data caches reset on logout; no logic without edge/failure tests and no fix without a regression test; simplest correct solution and no duplicated logic.

### iOS risks
- MVVM: networking, persistence, parsing or business rules in a ViewController; UIKit inside a ViewModel.
- Crashes: force unwrap, force cast, index out of range, UI update off the main thread.
- Memory: strong `self` in closures, timers, observers and Combine subscriptions not cleaned up, Tasks that outlive the screen.
- Concurrency: stale async results updating a newer state, missing cancellation, duplicate requests, `performAndWait` on the main thread, main-actor values read from nonisolated closures.
- Lifecycle: bindings added again on each `viewWillAppear`, setup and cleanup not balanced.
- Data: dates and time zones, pagination, stale cache, Core Data context on the wrong thread.
- Security: tokens, keys or personal data in logs.
- CI: tests that use the real Keychain-backed session store fail on CI (no Keychain). They need an in-memory store.
- Noise: `project.pbxproj`, generated files or `.xcscheme` changes that are not part of the task.

### Android risks
- MVI: business logic, networking or persistence in a Composable or Activity/Fragment; state mutated outside the ViewModel; one-shot events sent through `State` instead of an `Effect`.
- Crashes: `!!`, unchecked casts, `lateinit` read before set, index out of range, work on the main thread (disk, network, Room).
- Lifecycle and memory: `Context`/`Activity`/`View` held in a ViewModel or singleton; coroutines launched in `GlobalScope` or a scope that outlives the screen; flows collected without `repeatWhenStarted`/`collectAsStateWithLifecycle`; listeners not removed.
- Concurrency: stale results overwriting newer state, missing cancellation, duplicate requests, shared mutable state across coroutines.
- Compose: unstable parameters or new lambdas causing recomposition storms; side effects outside `LaunchedEffect`/`DisposableEffect`; `remember` missing keys.
- Data: dates and time zones, pagination, stale cache, Room migrations.
- Security: tokens, keys or personal data in logs; secrets in code or resources; exported components without need.
- Noise: unrelated Gradle, lockfile or formatting-only churn.

### Tests (the most common miss)
- **Real path:** does the test go through the real entry point (the view model, use case, repository, payload builder)? Flag a test that builds its own copy of the production logic.
- **Would it fail?** For every fix and every new rule: if the old code comes back, does a test fail? If not, name the line to revert and the test that should catch it.
- **Both variants:** for a flag or variant, both sides must be tested at the real send or save site.
- **Tests that cannot fail:** production output compared with production output; a value assigned then read back; a guard tested with input that can never reach it.
- **A test edited to pass:** if the PR changed a test so it would pass, ask why.
- **E2E:** if the change alters a user flow, check that an end-to-end flow (Maestro, XCUITest, Espresso...) covers it, existing or added. A new user-visible flow with no e2e coverage is a Missing Test Coverage item.

### Data-safety checklist (step 6)
- **Races:** read-check-write and find-or-create without an atomic step; status changes that two callbacks can apply twice or skip.
- **Enum and value completeness:** a new enum case, status string or type constant is traced through every consumer (read each file that switches on, filters by or shows sibling values). A consumer that falls through to a wrong `default` is a finding.
- **LLM / chat output trust:** model output written to storage, opened as a URL, or rendered as HTML without a shape check.
- **Completeness gaps:** partial enum handling, missing error paths or missing negative tests that would cost little to add.
- **Time windows:** "today" keys that assume a full day, mixed UTC and local buckets for the same data.
- **Type coercion at boundaries:** numbers that arrive as strings (or the reverse) from the server, keys or hashes built from them.
- **CI and scripts** when touched: tool versions, artifact paths, secrets through `${{ secrets.X }}`, no untrusted input in `run:`.

### Code smells
Report a smell only when it makes this change harder to read, test or change safely. Name it, point to the line, say what to do.

A smell is always a judgement call, never a hard rule break: write "possible Feature Envy", not "Feature Envy". A project rule always wins: if a rules file allows a pattern, do not flag it as a smell.
- God type, long function, deep nesting (3+ levels), long parameter list or Boolean flag parameters.
- Duplicated logic across screens or view models (say where the shared one should live).
- Primitive obsession / stringly typed values; magic numbers and strings.
- Feature envy, shotgun surgery, speculative generality (one-implementation protocol/interface, unused parameter).
- Test hooks open to all production callers; dead code; hidden shared state (singletons read deep in code).
- Inconsistent naming, misleading names or comments, parallel switches, silent `default`/`else` over a closed set.
- Data clumps: the same few values always passed together. Fix: one small type that holds them.
- Divergent change: one file edited in this PR for several unrelated reasons. Fix: split it so each part changes for one reason.
- Message chains: long `a.b.c.d` walks the caller should not know about. Fix: one method on the first object.
- Middle man: a type or function that only passes calls on. Fix: remove it and call the real target.
- Refused bequest: a subclass or conformer that ignores or overrides most of what it gets. Fix: use composition instead.

### Over-engineering lens (step 5)
Find what to delete. One line per finding: `file:line: <tag> <what>. <replacement>.`
- `delete:` dead code, unused flexibility, speculative feature.
- `stdlib:` hand-rolled thing the standard library ships (name it).
- `native:` code or dependency doing what the platform does (name the feature).
- `yagni:` abstraction with one implementation, config nobody sets, layer with one caller.
- `shrink:` same logic in fewer lines (show it).

End with `net: -<N> lines possible`, or "Lean already". A single focused test is never bloat. This lens finds complexity only; correctness and security belong to the other passes.

### Large or cross-cutting PRs
For a PR over about 15 files, review in passes by area (core type, call sites, tests, resources). Say which pass found what.

## Output format
The first line of the report is `VERDICT: PASS` or `VERDICT: CHANGES` (CHANGES when any blocking issue is open). Then:

# PR Review Summary
- Platform: iOS / Android
- Spec: the ticket ID and title, or "No spec available"
- Overall risk: Low / Medium / High / Critical
- Main risk areas:
- Per pass: `[Spec]` N findings, worst: ... / `[Rules]` N findings, worst: ... (name the worst of each pass on its own; do not pick one winner across passes)

# Detailed Findings
One line each, most serious first:
- **[Critical | High | Medium | Low | Potential issue]** `[Spec]` or `[Rules]` `file:line` - what is wrong, what could happen to the user, and a polite suggestion. Add "(breaks MVVM/MVI)", "(breaks Rule N)" or the rules file name when it applies. A `[Spec]` finding quotes the ticket line.
  Evidence: short quoted line, call path, or test result.

## Code Smells
- `file:line` - smell name, suggestion, evidence. (Not blocking unless it hides a bug.)

## Copies Found
- The knowledge, and each copy as `file:line`. Write "none found" only after you ran the search.

## Missing Test Coverage
- The production line and the test that should fail without it.

## Deferred / Not Verified
- Each deferred item with its reason, and each thing you did not verify.

## Final Verdict
- Approve: Yes / No (No if any blocking issue is open, a copy is left in place, or a fix has no test that would fail without it).
- Blocking issues:
- Non-blocking improvements:
