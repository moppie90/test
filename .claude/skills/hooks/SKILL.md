---
name: hooks
description: Ruflo's self-learning hooks - what runs automatically in this repo, how to pretrain routing on the codebase, view learning metrics, and dispatch background workers (audit, map, testgaps, optimize, document). Use when hooks misbehave or to make routing smarter.
---

# Ruflo hooks

## What runs automatically

`.claude/settings.json` wires every Claude Code hook to
`.claude/helpers/hook-handler.cjs`:

| Event | Action |
|-------|--------|
| `SessionStart` | Restore session state, import auto-memory |
| `UserPromptSubmit` | Route the prompt and suggest an agent |
| `PreToolUse` Bash / Edit | Risk check before commands and edits |
| `PostToolUse` Bash / Edit | Record outcomes for learning |
| `SubagentStart` / `SubagentStop` | Track agents, record task results |
| `PreCompact` | Save state before context compaction |
| `Stop` | Sync auto-memory |
| `SessionEnd` | Persist session state |

The handlers never block a tool call. If one misbehaves, test it directly:

```bash
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | node .claude/helpers/hook-handler.cjs pre-bash
npx -y ruflo@3.51.1 hooks list
npx -y ruflo@3.51.1 doctor
```

## Make routing smarter

Learn from this repository once, then after larger changes:

```bash
npx -y ruflo@3.51.1 hooks pretrain -d medium
npx -y ruflo@3.51.1 hooks build-agents      # optional: derive agent configs from pretrain data
```

Inspect a routing decision:

```bash
npx -y ruflo@3.51.1 hooks route -t "<task>" -K 3
npx -y ruflo@3.51.1 hooks explain -t "<task>"
npx -y ruflo@3.51.1 hooks model-route -t "<task>"     # haiku / sonnet / opus by complexity
```

Close the loop after a task so routing learns:

```
mcp__claude-flow__hooks_post-task({ taskId: "<id>", success: true, agent: "<agent>", task: "<task>", storeDecisions: true })
```

Learning metrics: `npx -y ruflo@3.51.1 hooks metrics -p 7d` or MCP
`hooks_metrics`, `hooks_intelligence_stats`.

## Background workers

| Worker | Use after |
|--------|-----------|
| `audit` | Security-relevant changes |
| `map` | 5+ files changed (refreshes the codebase map) |
| `testgaps` | Adding features |
| `optimize` | Performance work |
| `document` | API changes |
| `refactor`, `deepdive`, `benchmark`, `consolidate`, `ultralearn`, `predict`, `preload` | On demand |

```bash
npx -y ruflo@3.51.1 hooks worker dispatch -t audit -c src/ --sync
npx -y ruflo@3.51.1 hooks worker status
```

MCP: `hooks_worker-dispatch`, `hooks_worker-status`, `hooks_worker-list`.

The `daemon` runs these workers on a schedule, but each run starts a headless
Claude session and spends tokens. It is off by default; start it only on
purpose (`npx -y ruflo@3.51.1 daemon start`, stop with `daemon stop`).

## Optional git hooks

`.claude/helpers/pre-commit` and `post-commit` can be installed as git hooks;
see `.claude/helpers/README.md`.
