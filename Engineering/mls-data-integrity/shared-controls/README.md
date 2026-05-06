---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# Shared Controls

This file is the global operating policy contract that applies across all domains.

## Cross-Domain Invariants

### 1) Auth and invocation safety

- Scheduled lanes must enforce cron-secret validation (`MLS_SYNC_CRON_SECRET` + `x-cron-secret`).
- If auth posture is uncertain in any lane, classify as risk and escalate before recommending rollout changes.

### 2) Lease safety

- Lease-gated lanes must use `public.try_acquire_job_lease` and `public.release_job_lease`.
- A locked response is a valid control outcome, not an automatic failure.
- Rapidly increasing lock contention is an operational risk signal.

### 3) Cursor/state safety

- Cursor-driven lanes must show forward movement over expected windows.
- Persistent stale cursor state is incident-class unless explicitly explained by planned downtime.

### 4) Idempotency and retry posture

- Writes should remain replay-safe (upsert or equivalent conflict-safe behavior).
- Queue/worker flows must tolerate at-least-once delivery.
- Retry/backoff should favor containment over hot-loop amplification.

### 5) Risk classification language

Every analysis/report should separate:

- Confirmed (direct evidence)
- Inferred (reasoned but partial evidence)
- Unknown (requires human/runtime confirmation)

## PASS / FAIL / UNKNOWN Framework

- `PASS`: gate criteria met with sufficient current evidence.
- `FAIL`: hard-gate breach, integrity-impacting threshold breach, or emergency signal.
- `UNKNOWN`: evidence is incomplete or contradictory.

Escalation rules:

- Any `PASS -> FAIL` transition triggers incident mode.
- Any `UNKNOWN` persisting for 2 consecutive daily runs triggers escalation.

## Incident Mode Contract

When triggered:

1. Stop proposing additional rollout/optimization actions.
2. Capture evidence snapshot (logs, query outputs, impacted lanes).
3. Propose smallest containment or rollback path.
4. Re-run the relevant domain runbook after mitigation.

## Global Investigation Pattern (FAIL / Persistent UNKNOWN)

1. Identify the affected lane, function, worker, and dependency boundary.
2. Check:
   - logs and error signatures,
   - queue/backlog conditions,
   - cursor/state progression,
   - recent deploy/config changes,
   - upstream MLS/API health signals.
3. Classify likely source:
   - upstream,
   - ingestion,
   - processing,
   - storage,
   - config.
4. Propose the smallest safe containment action.
5. Do not execute state-changing remediation without explicit approval.
6. Keep deep investigation scoped to the affected domain unless cross-domain impact is evidenced.

### Cloud Run failure heuristics

- Repeated job failures (3 consecutive) can be classified as a `Hard-Gate Failure`.
- `429 Too Many Requests` usually indicates a rate-limit pressure issue.
- `504 Gateway Timeout` usually indicates an upstream/source timeout issue.

## Slack Reporting Contract

### Daily report (scheduled)

- Channel: Slack (operator-defined destination).
- Window: rolling (`last report timestamp -> now`).
- Required sections:
  - domain statuses (`PASS` / `FAIL` / `UNKNOWN`)
  - KPI values + threshold + delta
  - top risks
  - recommended next action
  - evidence pointers

### Urgent incident alert (immediate)

Trigger on:

- hard-gate failure,
- severe freshness/cursor failure,
- sustained queue/worker critical failure,
- repeated job failures reaching hard-gate threshold,
- user-visible MLS integrity degradation risk.

Urgent message must include:

- impact summary,
- affected domain/lane,
- confidence level,
- evidence source,
- immediate containment recommendation.

## Source-Of-Truth Rule For Volatile Data

Do not hardcode volatile runtime details in core memory contract files.

Always discover live values from systems of record:

- Supabase cron UI/state for exact schedules
- Cloud Run/PubSub metrics for runtime health
- repository code paths for current implementation contract

## Policy vs Enforcement Map

Define policy in Layer 2 and enforce it in platform configuration:

- **File access**
  - Policy location: `../memory-governance.md`
  - Enforcement location: OS/container/repo controls (read-only mounts/ACL/protection). `chmod 444` can be a local fallback, not the only control.
- **GitHub**
  - Policy location: `../agent-identity/permissions.md`
  - Enforcement location: PAT scopes + branch protections.
- **Supabase**
  - Policy location: `../agent-identity/permissions.md`
  - Enforcement location: read-only DB role credentials (`mls_auditor`).
- **Google Cloud**
  - Policy location: `../agent-identity/permissions.md`
  - Enforcement location: least-privilege IAM service account roles.
- **Credentials**
  - Policy location: `../agent-identity/vault/`
  - Enforcement location: env/MCP/secret manager injection.

## Provenance

Legacy detail references:

- `../agent-identity/how-the-system-works.md`
- `../domains/`
- `../memory-governance.md`

See also:

- `../memory-governance.md` for immutable-vs-mutable write policy.
