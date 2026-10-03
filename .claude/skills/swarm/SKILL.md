---
name: swarm
description: Run a task with several coordinated agents - pick a team, spawn it, track progress and shut it down. Covers Ruflo swarms, hive-mind (queen-led consensus), agent management and autopilot. Use for work touching 3+ files, new features, cross-module refactors, or research that splits into independent parts.
argument-hint: "<task description>"
---

# Swarm orchestration

Claude Code's own `Agent` tool does the actual work. Ruflo adds the
coordination layer: topology, shared memory, task tracking and learned routing.

## When to swarm

- **Yes**: 3+ files, new features, cross-module refactors, API changes,
  security or performance work, research with independent parts.
- **No**: single-file edits, 1-2 line fixes, docs, config tweaks, questions.
  Just do those directly.

## 1. Route

Ask Ruflo which agents fit, based on what worked before:

```
mcp__claude-flow__hooks_route({ task: "$ARGUMENTS" })
```

```bash
npx -y ruflo@3.51.1 hooks route -t "$ARGUMENTS" -K 3
```

## 2. Pick a team

Agents available in this repo (`.claude/agents/`):

| Agent | Role |
|-------|------|
| `researcher` | Read-only investigation, reports findings |
| `planner` | Breaks work into tasks with dependencies |
| `architect` | Specification, pseudocode and design (SPARC S/P/A) |
| `coder` | Implements changes |
| `tester` | Writes and runs tests, TDD |
| `reviewer` | Reviews for bugs, security and quality |
| `swarm-coordinator` | Leads larger swarms and resolves conflicts |

Built-in Claude Code agents also work: `Explore` (fast read-only search),
`Plan`, `general-purpose`.

| Task | Team | Pattern |
|------|------|---------|
| Bug fix | researcher → coder → tester | Pipeline |
| Feature | architect → coder → tester → reviewer | Pipeline |
| Refactor | architect → coder → reviewer | Pipeline |
| Research | 2-4 × researcher in parallel | Fan-out |
| Large change | swarm-coordinator + workers | Supervisor |

## 3. Initialize (optional, for tracking)

```
mcp__claude-flow__swarm_init({ topology: "hierarchical", maxAgents: 8, strategy: "specialized" })
```

Topologies: `hierarchical` (default, one lead, least drift), `mesh` (peers,
for independent research), `hierarchical-mesh`, `ring`, `star`.

## 4. Spawn

Spawn every agent in one message with `run_in_background: true` and tell
each one who to report to:

```
Agent({ name: "researcher", subagent_type: "researcher", run_in_background: true,
        prompt: "Investigate <X>. SendMessage findings to 'architect'." })
Agent({ name: "architect", subagent_type: "architect", run_in_background: true,
        prompt: "Wait for 'researcher'. Design the change. SendMessage the plan to 'coder'." })
Agent({ name: "coder", subagent_type: "coder", run_in_background: true, isolation: "worktree",
        prompt: "Wait for 'architect'. Implement. SendMessage a summary to 'tester'." })
Agent({ name: "tester", subagent_type: "tester", run_in_background: true,
        prompt: "Wait for 'coder'. Test it. SendMessage results to 'reviewer'." })
Agent({ name: "reviewer", subagent_type: "reviewer", run_in_background: true,
        prompt: "Wait for 'tester'. Review and report to the lead." })
```

Rules:

- Name every agent so others can `SendMessage` it.
- Never two writers in one worktree: give each writing agent
  `isolation: "worktree"` and a non-overlapping set of files.
- Read-only agents may run in parallel freely.
- After spawning, keep doing independent work; do not poll. Agents report
  back when done.

## 5. Track and finish

| Goal | MCP | CLI |
|------|-----|-----|
| Swarm status | `swarm_status`, `swarm_health` | `swarm status` |
| Agents | `agent_list`, `agent_status`, `agent_health` | `agent list`, `agent health` |
| Tasks | `task_create`, `task_list`, `task_status`, `task_complete` | `task list`, `task status <id>` |
| Shut down | `swarm_shutdown` | `swarm stop` |

Then record the outcome so routing improves:

```
mcp__claude-flow__hooks_post-task({ taskId: "<id>", success: true, agent: "coder", task: "$ARGUMENTS", storeDecisions: true })
```

## Hive-mind (queen-led consensus)

For decisions several agents must agree on:

```bash
npx -y ruflo@3.51.1 hive-mind init -t hierarchical-mesh
npx -y ruflo@3.51.1 hive-mind spawn -n 4
npx -y ruflo@3.51.1 hive-mind task -d "<task>"
npx -y ruflo@3.51.1 hive-mind status
npx -y ruflo@3.51.1 hive-mind shutdown
```

MCP: `hive-mind_init`, `hive-mind_spawn`, `hive-mind_consensus`,
`hive-mind_broadcast`, `hive-mind_memory`, `hive-mind_status`,
`hive-mind_shutdown`.

## Autopilot (keep going until all tasks are done)

```bash
npx -y ruflo@3.51.1 autopilot enable
npx -y ruflo@3.51.1 autopilot config --max-iterations 20 --timeout 60
npx -y ruflo@3.51.1 autopilot status
npx -y ruflo@3.51.1 autopilot disable
```

Autopilot re-engages Claude at every stop until the task list is empty, so it
spends tokens continuously. Always set a low `--max-iterations`.
