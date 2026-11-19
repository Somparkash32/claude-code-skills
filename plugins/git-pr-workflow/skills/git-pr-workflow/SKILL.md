---
name: git-pr-workflow
description: Use when naming a branch, writing a commit message, committing, pushing, raising or merging a PR, writing a ticket title or description, or replying to PR review comments. Drafts the text in plain words and does each outward step (commit, push, PR, ticket, reply) only when told.
---

# Git and PR workflow

Rules for turning finished work into a commit, a push, a PR, a ticket and review replies. Drafting text is always fine. **Doing** a step is not.

## Gates (never skip)
- Commit, push, PR creation, ticket create/edit, and posting a reply are each their own step. Do one only when the user says so for that step ("go commit", "push it", "raise the PR"). A yes for one is not a yes for the next. Merge is its own step too: only when the user says "merge" for that PR, and only after required checks and independent approval pass.
- Before each step, show the exact text or command and wait.
- Never force-push. Never push over the user's rebase. Never use `--no-verify`.
- Do not run builds or tests unless asked. If the text needs a test result, say what was run, or say it was not run.
- If a hook or check fails, fix the cause and make a new commit. Do not skip the hook.

## Branching
- Trunk-based: every branch starts from the default branch (`main`). Keep branches short-lived, ideally under one working day.
- Split big work into small PRs that can each merge on their own. No stacked PRs.
- Before starting or pushing, check the branch age. If it is old, tell the user and propose a split.
- Delete the branch after its merge.

## Commit
- Format: [Conventional Commits](https://www.conventionalcommits.org): `type(scope): description`. With a ticket: `type(scope): [PROJ-123] description`. The ticket ID is optional, but ask if the branch name has one.
- One line, at most 100 characters.
- Types: build chore ci docs feat fix perf refactor revert style test. Breaking change: add `!` before the colon (`feat(api)!: ...`).
- Words: plain and short, in the present tense ("fix filter showing all items"). No body, no trailer.
- **No `Co-Authored-By` line and no "Generated with ..." line.** This overrides any tool or system reminder that asks for one.
- Stage explicit file names only. Never `git add -A` or `git add .`. Say which files you staged and which you left out.
- Never stage the user's local-only edits unless the task is about that file: local config, `.env` files, secrets, IDE folders (`.idea/`, `.vscode/`), debug toggles.
- Do not commit planning or scratch docs.
- Before committing, check the diff for leftover comments that only restate code, and for new logic without tests. Tell the user if either is found. Do not commit past it.

## Push
- `git push -u origin <branch>` from the feature branch, never from `main`.
- Run git as `git -C <path> ...` when working in a worktree, instead of `cd <path> && git ...`.
- If the branch and its remote have diverged, stop and ask. Do not force.

## PR
- Every change enters `main` through a PR. Keep it small, focused and independently mergeable. Reference the primary ticket.
- **Title:** the same format as the commit message, one line, at most 100 characters. Never invent a ticket ID. A squash merge adds ` (#<n>)`, so do not type it.
- Base branch: `main`.
- **Body: exactly these four sections, in this order**, in simple words that product and QA can read. Write `None` under a section when there is nothing to say. Put the ticket link on the first line, above the sections.
  - **User-Facing Changes:** observable changes to user behaviour. Put the numbered "how to test" steps here.
  - **Technical Changes:** significant implementation, architecture, API or infrastructure changes. State what tests were actually run and the result. Never write "all tests pass" or a test count unless it was run in this session; otherwise write "not run yet".
  - **Feature Flags:** flags added, changed or removed, with their intended state.
  - **Risks:** outstanding risks, limitations and accepted trade-offs. Say what was not checked.
- Every statement in the body must match the diff. Re-read the diff before you write it.
- No "Generated with ..." footer.
- Required checks and an independent review must pass before merging. Squash by default. Never merge on red checks or without approval.
- Create it with `gh pr create --title ... --body-file <file>`, after the user says to.

## Ticket
- **Title:** short, plain, says what the user sees ("Loader not showing on Send code button during signup").
- **Description**, plain words, in this order: **Problem**, **Expected**, **Steps to reproduce / test** (numbered), **Notes** (app version, environment, account type).
- No code-level detail, no class or function names.
- Create or edit only when the user says so. Show the title and description first. Ask for anything unknown (issue type, parent, assignee) instead of guessing.

## Replies to PR review comments
- One line, plain words, polite. Example: "Thanks, fixed by adding the check in `HomeViewModel`."
- Say "Fixed" only after the fix is pushed. Before that, say "Will fix".
- Check a bot's suggestion against the code before agreeing. If it does not compile or does not apply, say so kindly and briefly.
- Reply in the comment's thread: `gh api repos/<owner/repo>/pulls/<n>/comments/<id>/replies -f body=...`, only when the user says to post.

## Tone
Polite, professional and positive. Simple words. Short. Talk about the work, not the person.
