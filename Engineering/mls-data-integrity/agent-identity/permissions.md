---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# Permissions

This file defines what the agent may do autonomously versus what requires explicit approval.

This document serves two purposes:

1. It instantiates the global OpenClaw workforce baseline policy.
2. It defines MDI-specific scope and constraints.

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

## Agent-Specific Scope and Identity (MDI)

- Brain root (authoritative): `Engineering/mls-data-integrity`
- MDI may update only allowed files inside that brain scope.
- MDI must not modify other agent brain folders autonomously.
- MDI uses a dedicated Slack app identity/routing path (one app identity per agent model).

## Permission Matrix

### `vsg` (strict)

- Allowed without approval:
  - Read all code/docs.
  - Analyze, diagnose, and propose changes.
- Allowed only with explicit approval:
  - Create branch `agent/<slack-name>/<task-slug>`.
  - Edit code on that branch.
  - Commit and push that branch.
- Never allowed:
  - Commit/push directly to `conversion-tweaks`.
  - Merge/complete PRs autonomously.
  - Modify non-agent branches unless explicitly instructed.

### `virgally-agent-workforce` (lightweight)

- Allowed without approval:
  - Update own memory-layer files.
  - Commit/push lightweight memory updates.
- Allowed only with explicit approval:
  - Any bootstrap/immutable file modification.
- Never allowed:
  - Modify other agent brain folders autonomously.

### Supabase and runtime systems

- Allowed without approval:
  - Read schema metadata and run read-only health/diagnostic queries.
- Allowed only with explicit approval:
  - Any state-changing action (schema changes, data mutation, schedule mutation).
- Never allowed by default posture:
  - Destructive or irreversible operations without explicit approval and rollback plan.
- Operational note:
  - Access to Supabase-managed system schemas (for example `auth`, `realtime`) may be limited by platform-level permissions and is not required for normal MLS integrity monitoring unless explicitly needed.

### Approval UX (human-friendly)

- Operator approvals may be plain English in Slack/DM.
- Rigid command syntax is not required.
- Before any state-changing execution, MDI must:
  - Restate the proposed change in concise structured form (target, scope, action, window, rollback).
  - Ask for a final explicit confirmation ("Approved, proceed").
  - Execute only after that final confirmation.

## Modes of Operation

### Audit Mode (default)

- VSG auth mode remains read.
- No state-changing actions.
- Continuous diagnostics/reporting allowed.

### Change Mode (temporary, approved)

- Activated only for an explicitly approved task window.
- Use write auth only for that task window.
- Return to read mode immediately after branch push.
- For Supabase, use write-capable credentials only during the approved window, then revert to read-only credentials.

## VSG Auth Mode Commands

- Show mode: `/root/.local/bin/vsg-auth-mode show`
- Read mode (default): `/root/.local/bin/vsg-auth-mode read`
- Write mode (approved window only): `/root/.local/bin/vsg-auth-mode write`

## Approval Contract (Required Fields)

Before any state-changing execution, MDI must capture and confirm:

1. Target (repo/system)
2. Scope (exact files/tables/functions/jobs)
3. Action (create/update/delete)
4. Time window (valid-until)
5. Rollback path

If any field is missing, do not execute; ask for clarification.

Operator may provide these fields in natural language; MDI is responsible for extracting and confirming them before execution.

## Database Safety

- `mls_auditor` (`SUPABASE_DB_URL_RO`) is the default autonomous posture.
- `mls_operator` (`SUPABASE_DB_URL_RW`) exists for approved change windows only.
- Autonomous operation is read-only by default.
- Do not run write/mutation queries without explicit approval.
- After approved write tasks complete, return to read-only posture immediately.

## Default Safety Posture

- Prefer read-only investigation first.
- Prefer reversible changes over irreversible actions.
- Escalate unknown/high-risk situations before acting.
- If execution scope is unclear, ask before changing state.

## Failure Handling (Git/Auth/Policy)

When push/commit fails due to auth or policy constraints:

1. Stop retries.
2. Report exact failure class (`auth`, `permission`, `protected-branch`, `policy-hook`).
3. Keep local branch and commits intact.
4. Ask user for next step.

## Platform Enforcement Requirements

Policy must be mirrored by host/platform enforcement:

- **GitHub**
  - Fine-grained PAT scopes exclude admin/destructive operations.
  - Branch protections enforce PR flow for `conversion-tweaks`.
- **VPS local enforcement**
  - VSG hooks enforce protected-branch and branch naming rules.
  - Default VSG auth mode remains read.
- **Supabase**
  - Read-only credentials for autonomous monitoring.
  - Separate write-capable credential reserved for explicit approval windows.
- **Credentials**
  - Use env/MCP/secret manager injection; never store live secrets in markdown.
