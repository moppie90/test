---
name: ruflo
description: Ruflo itself - status, health check (doctor), sessions, background daemon, upgrading, and an overview of which skill covers what. Use when Ruflo or its MCP server misbehaves, to check system state, or to find the right Ruflo skill.
---

# Ruflo

Ruflo 3.51.1 is pinned in `.mcp.json` and in every command in this repo. Use
`npx -y ruflo@3.51.1 ...`, never `@latest` or the old `claude-flow` /
`@claude-flow/cli` names.

## Which skill for what

| Need | Skill |
|------|-------|
| Several agents on one task, hive-mind, autopilot | `swarm` |
| Recall or store patterns and decisions | `memory` |
| Phased feature work, TDD, debug/review modes | `sparc` |
| Repeatable multi-step pipelines | `workflow` |
| PRs, issues, releases, CI | `github` |
| Code quality, security, performance, verification | `analyze` |
| Hook behaviour, routing training, background workers | `hooks` |

## Status and health

```bash
npx -y ruflo@3.51.1 status
npx -y ruflo@3.51.1 doctor            # full check
npx -y ruflo@3.51.1 doctor --fix      # prints fix commands, applies nothing except the memory sidecar
npx -y ruflo@3.51.1 doctor -c mcp     # one component (mcp, memory, learning, mods, ...)
```

MCP: `system_status`, `system_health`, `mcp_status`.

If the auto-memory hook says it cannot load `@claude-flow/memory`, run
`npx -y ruflo@3.51.1 doctor -c learning`. That rewrites
`.claude-flow/memory-package.json`, which is machine-specific and therefore
not committed.

## Sessions

```bash
npx -y ruflo@3.51.1 session save -n "<name>"
npx -y ruflo@3.51.1 session list
npx -y ruflo@3.51.1 session restore <id>
```

## Background daemon (off by default)

Each daemon worker run starts a headless Claude session and spends tokens.

```bash
npx -y ruflo@3.51.1 daemon start
npx -y ruflo@3.51.1 daemon status
npx -y ruflo@3.51.1 daemon stop
```

## Upgrading Ruflo

1. Change the version in `.mcp.json` and replace `ruflo@3.51.1` everywhere in
   `.claude/` and `CLAUDE.md`.
2. Refresh the signed helpers: `npx -y ruflo@<new> init upgrade`.
3. Do **not** use `init upgrade --add-missing` or `init --force`: they bring
   back the ~250 generated commands, skills and agents that were merged into
   the skills and agents in this repo, many of which call commands that no
   longer exist.
4. Run `doctor` and commit.
