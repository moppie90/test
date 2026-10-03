---
name: sparc
description: SPARC development method (Specification, Pseudocode, Architecture, Refinement with TDD, Completion) and its working modes such as code, tdd, debug, security-review, docs, devops, integration, optimize and ask. Use for building a feature end to end in disciplined phases, or to work in one specific mode.
argument-hint: "[mode] <task>"
---

# SPARC

`/sparc <task>` runs all phases. `/sparc <mode> <task>` works in one mode.

## Phases

| # | Phase | Output | Agent |
|---|-------|--------|-------|
| 1 | **Specification** | Requirements, acceptance criteria, edge cases, constraints | `architect` |
| 2 | **Pseudocode** | Algorithm and data flow in plain steps, complexity notes | `architect` |
| 3 | **Architecture** | Components, interfaces, files to touch, risks | `architect` |
| 4 | **Refinement** | Implementation via TDD: failing test → code → refactor | `tester` + `coder` |
| 5 | **Completion** | Full test run, review, docs where asked, integration | `reviewer` |

Rules:

- Do not start a phase before the previous one's output exists.
- Write the phase 1-3 output to the conversation (or `docs/` only if the user
  asked for documentation), and recall earlier decisions first:
  `mcp__claude-flow__memory_search({ query: "<feature>", namespace: "decisions" })`.
- Refinement is test-first. Each change starts with a failing test.
- Store the key design decisions afterwards in the `decisions` namespace
  (see the `memory` skill).

To run the phases as a tracked Ruflo workflow instead:

```bash
npx -y ruflo@3.51.1 workflow run -t sparc --task "<task>" --dry-run
```

## Modes

Pick a mode for focused work. Each is a stance, not a separate tool.

| Mode | Focus | Agent |
|------|-------|-------|
| `spec` | Phases 1-2 only: requirements and pseudocode | `architect` |
| `architect` | Phase 3 only: design and interfaces | `architect` |
| `code` | Clean, modular implementation from an existing design | `coder` |
| `tdd` | Red-green-refactor loop, London-school mocks where useful | `tester` + `coder` |
| `debug` | Reproduce, trace, isolate root cause, fix with a regression test | `coder` |
| `review` | Bugs, security, readability of a change | `reviewer` |
| `security-review` | Secrets, injection, authz, unsafe dependencies | `reviewer` |
| `optimize` | Measure, find the bottleneck, improve, measure again | `coder` |
| `research` | Investigate code, docs or the web and report | `researcher` |
| `docs` | Concise usage and integration docs (only when asked) | `coder` |
| `devops` | CI/CD, deployment, infrastructure | `coder` |
| `integration` | Merge parallel work into one tested whole | `swarm-coordinator` |
| `orchestrate` | Split a large objective over several agents (see `swarm` skill) | `swarm-coordinator` |
| `ask` | Help formulate the task and pick the right mode | — |

For a mode that needs several agents, follow the spawning rules in the
`swarm` skill (named agents, one writer per worktree).
