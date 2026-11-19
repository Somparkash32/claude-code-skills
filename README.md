# claude-code-skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace: a collection of installable plugins, each providing one or more skills.

## Adding this marketplace

```
/plugin marketplace add Somparkash32/claude-code-skills
```

## Installing a plugin

```
/plugin install <plugin-name>@claude-code-skills
```

## Available skills

| Plugin | Skill | Description |
| --- | --- | --- |
| [mobile-pr-review](./plugins/mobile-pr-review) | `mobile-pr-review` | Reviews a PR, branch or diff for iOS (Swift/UIKit/MVVM) or Android (Kotlin/Compose/MVI) apps. Runs two passes in parallel: a spec pass (what the ticket asked vs. what the PR does) and a rules pass (the repo's `CLAUDE.md` and other rules files). Adds repo-wide copy checks, platform risk checks, a test-quality check, a code smell pass and an over-engineering pass. Every finding needs proof. Read-only: posts inline PR comments only when you ask. |

See [CLAUDE.md](./CLAUDE.md) for the conventions to follow when adding or updating a plugin/skill.
