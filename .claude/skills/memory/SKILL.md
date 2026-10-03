---
name: memory
description: Store, search and learn from Ruflo's persistent vector memory (.swarm/memory.db). Use before a task to recall earlier patterns and decisions, after a task to save what worked, and for neural pattern training. Also the entry point for the AgentDB / ReasoningBank library references.
---

# Ruflo memory

Ruflo keeps a SQLite + HNSW vector memory in `.swarm/memory.db` (plus
`.swarm/agentdb-memory.db`). It is committed to git, so it survives between
cloud sessions. Commit it after storing anything worth keeping.

## Before a task: recall

MCP (preferred):

```
mcp__claude-flow__memory_search({ query: "<task keywords>", namespace: "patterns", limit: 5 })
mcp__claude-flow__memory_search_unified({ query: "<task keywords>" })   // all stores at once
```

CLI:

```bash
npx -y ruflo@3.51.1 memory search -q "<task keywords>" -n patterns
npx -y ruflo@3.51.1 memory search -q "<question>" --smart          # query expansion + reranking
```

## After a task: store

```
mcp__claude-flow__memory_store({ key: "<short-name>", value: "<what worked and why>", namespace: "patterns" })
```

```bash
npx -y ruflo@3.51.1 memory store -k "<short-name>" --value "<what worked>" -n patterns --provenance agent_output
```

Use namespaces consistently:

| Namespace | Contents |
|-----------|----------|
| `patterns` | Approaches that worked (and ones that failed) |
| `decisions` | Architecture and product decisions with the reason |
| `project` | Facts about this repo: commands, conventions, gotchas |

## Inspect and maintain

| Goal | MCP | CLI |
|------|-----|-----|
| List entries | `memory_list` | `memory list -n <ns>` |
| Get one entry | `memory_retrieve` | `memory retrieve -k <key> -n <ns>` |
| Delete | `memory_delete` | `memory delete -k <key> -n <ns>` |
| Statistics | `memory_stats` | `memory stats` |
| Clean up stale entries | `memory_cleanup` | `memory cleanup` |
| Export / import | `memory_export` / `memory_import` | `memory export` / `memory import` |
| Snapshot DB | — | `memory backup` |
| Import Claude Code auto-memory | `memory_import_claude` | — |

## Neural pattern learning

```bash
npx -y ruflo@3.51.1 neural status
npx -y ruflo@3.51.1 neural train -p coordination
npx -y ruflo@3.51.1 neural patterns --action list
```

MCP: `neural_status`, `neural_train`, `neural_patterns`, `neural_predict`.

## AgentDB / ReasoningBank library references

The files in `references/` document the standalone AgentDB library
(`agentdb@3.0.0-alpha.20`, imported via `agentic-flow/reasoningbank`) for
building memory into your own application code. They are not needed to use
Ruflo's memory above and may lag behind the library; check the library's own
`--help` before relying on a CLI flag from them.

| Reference | Topic |
|-----------|-------|
| `references/agentdb-vector-search.md` | Semantic search, RAG |
| `references/agentdb-memory-patterns.md` | Session and long-term memory patterns |
| `references/agentdb-optimization.md` | Quantization, HNSW tuning, caching |
| `references/agentdb-advanced.md` | QUIC sync, multi-DB, hybrid search |
| `references/agentdb-learning.md` | Reinforcement-learning plugins |
| `references/reasoningbank-agentdb.md` | ReasoningBank on AgentDB |
| `references/reasoningbank-intelligence.md` | Pattern recognition and strategy optimization |
