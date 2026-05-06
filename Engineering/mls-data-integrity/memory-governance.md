# Memory Governance

This file defines which core-memory files are immutable bootstrap law versus mutable learning memory.

## Two-Layer Model

## Bootstrap (Immutable Law)

Purpose:

- Define non-negotiable operating constraints.
- Prevent agent self-modification from weakening safety/authority boundaries.

Write policy:

- Treat as read-only during autonomous operation.
- Any change requires explicit human approval.

## Memory (Adaptive Knowledge)

Purpose:

- Capture learned operational knowledge.
- Improve checks, diagnostics, and optimization strategy over time.

Write policy:

- Agent may update with clear rationale and evidence.
- Updates must preserve bootstrap constraints.

## File Classification

## Bootstrap files (immutable unless human-approved)

- `agent-identity/README.md`
- `agent-identity/how-the-system-works.md`
- `agent-identity/your-primary-job.md`
- `agent-identity/vault/*`
- `agent-identity/permissions.md`
- `shared-controls/README.md`
- `memory-governance.md`

## Memory files (agent can evolve)

- `system-context/usage-logs.md`
- `domains/**`

## Provenance-only references (not primary contract)

- direct repository sources (`supabase/functions/`, `scripts/`, `supabase/migrations/`)

## Hard Safety Rules

1. Never auto-edit Bootstrap files.
2. Never relax permission boundaries without explicit human instruction.
3. Never change incident/escalation semantics without explicit human instruction.
4. Memory updates must not conflict with Bootstrap files.

## Memory Update Protocol (for adaptive files)

When updating Memory files, include:

- what changed,
- why it changed,
- evidence source (query/log/code path),
- expected impact on health monitoring or reliability.

## Memory Update Triggers

Update adaptive memory when:

- a recurring incident pattern is confirmed,
- KPI thresholds/tuning guidance changes with evidence,
- investigation patterns materially improve signal quality,
- a domain runbook check consistently produces false positives/negatives.

## Do Not Update Memory For

Do not update memory for:

- one-off noisy anomalies without corroboration,
- undocumented assumptions with no evidence,
- temporary outages where root cause remains unknown,
- changes that would conflict with Bootstrap policy.

## Conflict Handling (Memory vs Live Behavior)

If adaptive memory conflicts with live system behavior:

1. Trust live evidence over stale memory text.
2. Mark the memory statement as stale/suspect.
3. Capture evidence (logs, queries, code path).
4. Propose the smallest memory correction.
5. Do not alter Bootstrap constraints while resolving the conflict.
