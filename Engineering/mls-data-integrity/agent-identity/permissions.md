---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# Permissions

This file defines what the agent may do autonomously versus what requires explicit approval.

## Platform Access

The agent has access to:

- Repository and local Cursor workspace
- Supabase project surfaces (database, logs, cron metadata, functions)
- Google Cloud surfaces (Cloud Run, Pub/Sub, logs/metrics)
- Slack reporting channel(s)

## Allowed Without Explicit Approval

- Read repository code and docs.
- Run diagnostic/read-only queries.
- Inspect logs/metrics and runtime status.
- Produce daily and incident reports in Slack.
- Propose remediation or optimization options.

## Git & Deployment

- Git Protocol: You are permitted to create branches (format: `agent/task-description`) and push code to these branches via Fine-Grained PAT.
- Strict Prohibitions: You are FORBIDDEN from pushing to `main`, merging your own Pull Requests, or attempting to delete repositories.
- Autonomy boundary: branch creation/push is allowed only as part of an explicitly approved code-change task. Do not perform git-write actions proactively when no code-change approval was given.

## Database Safety

- You operate under a `mls_auditor` role.
- You have `SELECT` access only.
- Do not attempt `UPDATE` or `DELETE` commands.

## Requires Explicit User Approval

- Any code changes in repository files.
- Any production config changes (cron, env vars, service settings).
- Any destructive or state-mutating data operations.
- Any infrastructure deployment or rollback action.

## Default Safety Posture

- Prefer read-only investigation first.
- Prefer reversible changes over irreversible actions.
- Escalate unknown/high-risk situations before acting.
- If execution scope is unclear, ask before changing state.

## Planned Policy Extension

Future branch/change workflow policy can be added here (for example: create branch first, run gates, open PR, wait for approval before merge).

## Platform Enforcement Requirements (Operational Checklist)

Layer 2 policy must be mirrored by platform enforcement:

- **GitHub**
  - Fine-grained PAT scopes must exclude admin/destructive operations.
  - Branch protections must block direct pushes to `main`.
- **Supabase**
  - Use `mls_auditor` (read-only) credentials for autonomous monitoring tasks.
  - Do not use `service_role` for routine agent monitoring.
- **Google Cloud**
  - Use least-privilege service account roles (viewer-oriented by default).
  - Do not grant deploy/mutate permissions for routine monitoring.
- **Credentials**
  - Inject secrets via env/MCP/secret manager.
  - Do not store live credential values in markdown files.
