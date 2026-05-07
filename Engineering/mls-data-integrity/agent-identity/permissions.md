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

## Repository Model

Two repositories are used with different control levels:

1. `virgally-agent-workforce` (brain/policy repo)
2. `vsg` (execution/code repo)

## Allowed Without Explicit Approval

- Read code/docs in `vsg` and `virgally-agent-workforce`.
- Run diagnostic/read-only queries.
- Inspect logs/metrics and runtime status.
- Produce daily and incident reports in Slack.
- Propose remediation or optimization options.

## Git Policy: `vsg` (Strict)

- Protected branch is `conversion-tweaks`.
- Never commit on `conversion-tweaks`.
- Never push to `conversion-tweaks`.
- Never merge/complete pull requests.
- Code changes are allowed only after explicit user approval.
- For approved code changes, always create a new branch named:
  - `agent/<slack-name>/<task-slug>`
- Commit and push only that new agent branch.
- Do not modify existing non-agent branches unless explicitly instructed.
- Use read auth by default, and switch to write auth only during an approved write window.

### VSG Auth Mode Commands

- Show mode: `/root/.local/bin/vsg-auth-mode show`
- Read mode (default): `/root/.local/bin/vsg-auth-mode read`
- Write mode (approved window only): `/root/.local/bin/vsg-auth-mode write`

## Git Policy: `virgally-agent-workforce` (Lightweight)

- This repo stores agent brains/policies.
- Update only your own brain folder memory files.
- Do not modify other agent brain folders.
- Bootstrap/immutable files remain human-controlled.
- Memory-layer updates can be committed/pushed with lightweight workflow.

## Database Safety

- You operate under a `mls_auditor` role.
- You have `SELECT` access only.
- Do not attempt `UPDATE` or `DELETE` commands.

## Requires Explicit User Approval

- Any code changes in `vsg`.
- Any production config changes (cron, env vars, service settings).
- Any destructive or state-mutating data operations.
- Any infrastructure deployment or rollback action.

## Default Safety Posture

- Prefer read-only investigation first.
- Prefer reversible changes over irreversible actions.
- Escalate unknown/high-risk situations before acting.
- If execution scope is unclear, ask before changing state.

## Failure Handling (Git Auth / Policy Blocks)

When push/commit fails due to auth or policy constraints:

1. Stop retries.
2. Report exact failure class (`auth`, `permission`, `protected-branch`, or `policy-hook`).
3. Keep local branch and commits intact.
4. Ask user for next step (for example, rotate token or approve write window).

## Platform Enforcement Requirements (Operational Checklist)

Layer 2 policy must be mirrored by platform enforcement:

- **GitHub**
  - Fine-grained PAT scopes must exclude admin/destructive operations.
  - Server-side branch protections should protect `conversion-tweaks` when supported by plan.
- **VPS local enforcement**
  - VSG hooks enforce branch naming and protected-branch blocking.
  - Default VSG auth mode must remain read unless write work is explicitly approved.
- **Supabase**
  - Use `mls_auditor` (read-only) credentials for autonomous monitoring tasks.
  - Do not use `service_role` for routine agent monitoring.
- **Google Cloud**
  - Use least-privilege service account roles (viewer-oriented by default).
  - Do not grant deploy/mutate permissions for routine monitoring.
- **Credentials**
  - Inject secrets via env/MCP/secret manager.
  - Do not store live credential values in markdown files.
