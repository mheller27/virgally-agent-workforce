---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Open Houses Runbook

## Goal

Keep open-house rows complete, current, and correctly linked to listings.

## Cadence

- Follow the canonical cadence in `../../README.md`.
- Domain deep dives run only on `FAIL` or `UNKNOWN` persisting for 2 consecutive daily runs.

## KPI Contract

- Open-house lane error rate: target `< 2%` sustained.
- Incremental cursor freshness (`mls_sync_state`): should update regularly.
- In-window truth alignment: stale-row cleanup and upsert behavior should remain balanced.
- Listing-link resolution quality: low unresolved/skip rate.

## Escalation States

- `PASS`: lane health and mapping quality in range.
- `FAIL`: sustained errors, stale cursor, or mapping failure spike; urgent Slack alert.
- `UNKNOWN`: insufficient evidence; escalate after 2 consecutive daily runs.

## Evidence Sources

- Incremental/reconcile run summaries and error logs.
- `public.mls_sync_state` row for incremental lane.
- `public.mls_open_houses` and listing-link sample checks.

## Suggested Checks (Adaptive)

1. Track incremental and reconcile processed/upsert/deleted trends.
2. Monitor incremental cursor movement and stale `nextLink` patterns.
3. Sample recent open-house rows to verify listing UUID linkage.
4. Watch for drift between expected in-window density and observed table volume.

## Adaptation Rule

The agent should improve query precision over time while preserving KPI meanings and escalation thresholds unless explicitly approved.

## Reporting

- Daily Slack summary with KPI deltas and top risks.
- Urgent Slack alert on sustained mapping failures or pipeline degradation.
