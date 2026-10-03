---
name: workflow
description: Run a predefined multi-step Ruflo workflow (development, research, testing, security-audit, code-review, refactoring, sparc) or a custom JSON one, including chaining steps so each step's output feeds the next. Use when a task follows a repeatable sequence of phases.
argument-hint: "<template> <task description>"
---

# Ruflo workflows

## Templates

| Template | Phases |
|----------|--------|
| `development` | Plan → implement → test → review |
| `research` | Gather → analyze → synthesize |
| `testing` | Coverage analysis → write tests → run |
| `security-audit` | Scan → threat model → report |
| `code-review` | Multi-agent review of a change |
| `refactoring` | Analyze → restructure → verify |
| `sparc` | Specification → pseudocode → architecture → refinement → completion |
| `custom` | Your own definition file |

```bash
npx -y ruflo@3.51.1 workflow template list
npx -y ruflo@3.51.1 workflow template show development
```

## Run

Always dry-run first to see the plan:

```bash
npx -y ruflo@3.51.1 workflow run -t development --task "<task>" --dry-run
npx -y ruflo@3.51.1 workflow run -t development --task "<task>" -m 4 --timeout 30
npx -y ruflo@3.51.1 workflow status
npx -y ruflo@3.51.1 workflow stop
```

MCP: `workflow_create`, `workflow_run`, `workflow_execute`,
`workflow_status`, `workflow_pause`, `workflow_resume`, `workflow_cancel`,
`workflow_list`, `workflow_template`, `workflow_validate`.

## Custom workflow file

Use JSON: Ruflo 3.51 only schema-checks JSON workflow files, not YAML.

```json
{
  "name": "feature",
  "steps": [
    { "name": "research",  "agent": "researcher", "task": "Find all code related to the task" },
    { "name": "design",    "agent": "architect",  "task": "Design the change", "dependsOn": ["research"] },
    { "name": "implement", "agent": "coder",      "task": "Implement the design", "dependsOn": ["design"] },
    { "name": "test",      "agent": "tester",     "task": "Test the change", "dependsOn": ["implement"] }
  ]
}
```

```bash
npx -y ruflo@3.51.1 workflow validate -f workflows/feature.json
npx -y ruflo@3.51.1 workflow run -f workflows/feature.json --task "<task>" --dry-run
```

## Chaining steps inside Claude Code

When steps must pass full output to each other and you do not need Ruflo's
tracking, run them as sequential `Agent` calls (or with `SendMessage`
between named agents, see the `swarm` skill) and pass each result into the
next prompt. This replaces the old `stream-chain` command, which Ruflo 3 no
longer ships.
