# Project instructions

This repo uses [Ruflo](https://github.com/ruvnet/ruflo) 3.51.1 as a
coordination layer on top of Claude Code: persistent memory, learned agent
routing, swarms and self-learning hooks. Claude Code does the work; Ruflo
coordinates and remembers.

## Rules

- Do what was asked; nothing more, nothing less.
- Read a file before editing it. Prefer editing existing files over creating
  new ones; create docs only when asked.
- Put new code, tests and scripts in `/src`, `/tests`, `/scripts`, `/config`
  or `/docs`, not in the repo root.
- Keep files under 500 lines.
- Validate input at system boundaries.
- Never commit secrets, credentials or `.env` files.
- Run the project's tests, lint and typecheck after code changes, if they
  exist. There is no `package.json` or test suite yet; check before
  assuming one.

## Approach

For anything beyond a small edit, think it through before acting:

1. **Recall**: search memory for earlier patterns and decisions
   (`mcp__claude-flow__memory_search`, namespaces `patterns`, `decisions`,
   `project`).
2. **Inspect**: read the relevant code and its tests.
3. **Plan**: acceptance criteria, files to touch, risks, and whether this
   needs one agent or a swarm.
4. **Execute** in small steps; tests first where practical.
5. **Verify**: run tests, check the diff for unrequested changes, and report
   honestly what passed, failed or was skipped.
6. **Learn**: store what worked (`mcp__claude-flow__memory_store`, namespace
   `patterns`) and commit `.swarm/` so memory survives the session.

## Agents

| Agent | Use for |
|-------|---------|
| `researcher` | Read-only investigation |
| `planner` | Breaking work into tasks |
| `architect` | Specification, pseudocode, design |
| `coder` | Implementation |
| `tester` | Tests, TDD, production readiness |
| `reviewer` | Read-only review of a change |
| `swarm-coordinator` | Leading a multi-agent swarm |

Built-in `Explore`, `Plan` and `general-purpose` agents are available too.

| Task | Team |
|------|------|
| Bug fix | researcher → coder → tester |
| Feature | architect → coder → tester → reviewer |
| Refactor | architect → coder → reviewer |
| Security | researcher → reviewer (security focus) |
| Research | 2-4 researchers in parallel |

Swarm only for work touching 3+ files, new features, cross-module refactors,
API, security or performance work. Not for single-file edits, small fixes,
docs, config or questions.

Multi-agent rules:

- Name every agent and tell it who to `SendMessage` when done.
- Spawn independent agents in one message with `run_in_background: true`;
  do not poll.
- Never two writers in one worktree: each writing agent gets
  `isolation: "worktree"` and its own files. Only the lead edits shared
  manifests and lockfiles.
- A sub-agent never gets more tools, permissions or spend than its parent.

## Skills

| Skill | Covers |
|-------|--------|
| `/swarm` | Multi-agent teams, hive-mind, autopilot |
| `/memory` | Persistent memory, neural patterns, AgentDB references |
| `/sparc` | Phased development and work modes (tdd, debug, review, ...) |
| `/workflow` | Predefined and custom multi-step workflows |
| `/github` | PR review, issues, releases, CI |
| `/analyze` | Code quality, security, performance, verification |
| `/hooks` | Hook behaviour, routing training, background workers |
| `/ruflo` | Status, doctor, sessions, daemon, upgrading |
| `/browser` | Browser automation via agent-browser |

## Ruflo CLI and MCP

- MCP tools are prefixed `mcp__claude-flow__` (memory, swarm, agents, hooks,
  tasks, analysis, security). Prefer them over the CLI inside Claude Code.
- CLI: always `npx -y ruflo@3.51.1 <command>`. Never `@latest`,
  `claude-flow@alpha` or `@claude-flow/cli`; those resolve to other versions
  or old commands that no longer exist (`sparc`, `hook`, `github`, `truth`,
  `pair`, ...).
- The background daemon is off. It spends tokens on every worker run; start
  it only on purpose.
- When upgrading Ruflo, follow the `ruflo` skill. Do not run
  `init --force` or `init upgrade --add-missing`; they restore the ~250
  generated files that were merged into the skills and agents above.
