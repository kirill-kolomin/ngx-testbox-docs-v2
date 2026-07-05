---
sidebar_position: 3
---

# AI agent skill

`ngx-testbox` has a dedicated AI agent skill package for coding agents that work with Angular integration tests.

The package is published separately as [`ngx-testbox-agent-skill`](https://www.npmjs.com/package/ngx-testbox-agent-skill).

## When to use it

Use the skill when an agent should:

- create tests with `ngx-testbox`
- review existing integration tests
- debug timing or stabilization issues
- refactor tests toward black-box patterns

The skill is focused on the `ngx-testbox` testing APIs, including:

- `runTasksUntilStableAsync`
- `runTasksUntilStable`
- `DebugElementHarness`
- `testboxTestId`
- predefined HTTP call instructions

## Install

Install the skill with `skillpm`:

```bash
npx skillpm install ngx-testbox-agent-skill
```

After installation, you can verify that it is available with:

```bash
skillpm list
```

## Package links

- [npm package](https://www.npmjs.com/package/ngx-testbox-agent-skill)
- [repository README](https://github.com/kirill-kolomin/ngx-testbox-agent-skill#readme)

## Compatibility

This skill is intended for `ngx-testbox` version 2 and later.

If you are still using the older `ngx-testbox` documentation set, migrate to the [v2 docs](https://kirill-kolomin.github.io/ngx-testbox-docs-v2/) and library version before relying on the skill guidance.
