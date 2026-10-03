---
name: github
description: GitHub work with agent help - multi-agent PR review, issue triage, releases, GitHub Actions and multi-repo changes. Use for reviewing or opening PRs, managing issues, preparing a release or fixing CI.
argument-hint: "<review|issue|release|ci|repo> [details]"
---

# GitHub with Ruflo

Do the GitHub actions themselves with the GitHub MCP tools (`mcp__github__*`)
or the `gh` CLI, whichever this environment has. Use Ruflo for the parts it
adds: risk analysis, reviewer suggestions, and agent coordination.

Never push, merge, close, release or comment on GitHub without the user
asking for it.

## PR review (multi-agent)

1. Assess the change:
   ```bash
   npx -y ruflo@3.51.1 analyze diff --risk --classify --reviewers
   ```
   MCP: `analyze_diff`, `analyze_diff-risk`, `analyze_diff-classify`,
   `analyze_diff-reviewers`, `analyze_file-risk`.
2. Spawn reviewers in parallel, each with one focus (see the `swarm` skill):
   correctness, security, tests, performance. Only add the ones the risk
   analysis calls for.
3. Merge their findings, drop duplicates and anything unverified, and rank
   by severity.
4. Post the review only when the user asks, as one review with inline
   comments.

## Issues

- **Triage**: read open issues, label by type and priority, flag duplicates,
  propose (not apply) labels unless asked.
- **Issue → work**: turn an issue into a task list, then run it as a swarm or
  SPARC flow. Track it with `mcp__claude-flow__task_create` if it spans
  sessions.
- Ruflo MCP `github_issue_track` keeps Ruflo-side tracking of issues.

## Releases

1. Collect merged PRs since the last tag; group as features, fixes, breaking
   changes.
2. Draft the changelog and version bump (semver) for the user to approve.
3. Run the full test suite and `npx -y ruflo@3.51.1 security scan --type deps`
   before tagging.
4. Create the tag or release only after explicit approval.

## CI / GitHub Actions

- Read the failing job's logs, reproduce locally, fix the root cause.
- Never disable or skip a test to get green.
- Ruflo MCP `github_workflow` can list and trigger workflows.

## Multi-repo changes

Plan per repository, give each repository its own agent and branch, keep a
single checklist of the cross-repo dependencies, and open PRs in dependency
order.

## Repository insight

Ruflo MCP: `github_repo_analyze` (structure and health), `github_metrics`
(activity), `github_pr_manage` (PR tracking).
