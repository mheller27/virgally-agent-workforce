---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Property Sync (`mls_listings`) Runbook

## Goal

Ensure listing truth, cursor movement, and dependent event/index outputs remain healthy.

## Cadence

- Follow the canonical cadence in `../../README.md`.
- Domain deep dives run only on `FAIL` or `UNKNOWN` persisting for 2 consecutive daily runs.

## KPI Contract

- Property lane error rate (reconcile/incremental/enrich): target `< 2%` sustained.
- Cursor freshness (`mls_sync_state.updated_at` by lane): should update regularly.
- Listing freshness lag (`last_update` ingestion lag): should not grow across windows.
- Event integrity (`mls_status_events`, `mls_price_events`): no sustained write drought during listing churn.

## Escalation States

- `PASS`: stable progression and no threshold breach.
- `FAIL`: threshold breach or stuck-cursor condition; immediate Slack incident alert.
- `UNKNOWN`: missing evidence; escalate if persistent across 2 consecutive daily runs.

## Evidence Sources

- Reconcile/incremental/enrich run summaries and errors.
- `public.mls_sync_state` progression checks.
- Listing/event table samples and trend queries.
- Tile-index backfill utility readiness (`tiles:backfill:missing`).

## Suggested Checks (Adaptive)

1. Verify reconcile/incremental lease behavior and locked-return frequency.
2. Track `last_next_link` persistence and stale-link recovery behavior in incremental runs.
3. Compare listing update volume against status/price event write volume.
4. Sample recently changed listings for enrichment eligibility vs processed counts.
5. Watch lag trend in listing freshness and tile index availability.

## Adaptation Rule

The agent should evolve query details and diagnostic depth over time while preserving KPI targets and escalation policy.

## Reporting

- Daily Slack summary with KPI deltas and top risks.
- Urgent Slack alert on hard failures (stuck cursor, sustained error spike, severe freshness lag).
