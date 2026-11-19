---
name: qa-test-case-creator
description: Use when an iOS ticket or code change needs tests. Reads the ticket and its design frames, turns each acceptance criterion into named cases at the right level, and dispatches the level creator skills in parallel.
---

# QA test case creator

Runs in the main session.

## 1. Read the story
- Ticket: fetch it from your tracker (Jira, Linear, GitHub issue... through its MCP server if one is connected) and its acceptance criteria. If there are none, ask the developer for them; do not invent ACs.
- Design: for each linked frame (for example Figma, through its MCP server), fetch the design context and note the exact copy and the states shown. If no design tool is connected, ask for the copy.

## 2. Build the AC map
One row per case:

| AC | Case (the test method name) | Context (`Given…`) | Level | Production file | Expected copy |
|---|---|---|---|---|---|

Levels:
- logic in a ViewModel, use case, service or mapper → unit
- how a view controller or view renders a state, or what it asks its ViewModel to do → UI
- what the user sees for a given backend response → unit, with the service faked
- a whole journey across screens on the real API → E2E (for example Maestro flows)

## 3. Check the code agrees
Read each production file. If an AC says one thing and the code does another, stop and report: the AC, the code path, what it does. Do not dispatch creators for that AC until the developer decides.

## 4. Dispatch
Dispatch one `qa-test-creator` agent per level, in parallel, in one message. Each request contains: the level skill name, its rows of the AC map, the ticket key, the production files, and the design copy.

## 5. Collect
Merge the returned files, test names, tickets and commands into one list for `qa-test-case-runner`. Every new test file must also be in the test target of the Xcode project; check the creators added it. Show "Proposed memory entries" to the user.
