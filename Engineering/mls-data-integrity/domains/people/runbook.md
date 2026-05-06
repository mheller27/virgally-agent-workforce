---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# People Runbook

## Goal

Maintain correct and timely derivation of offices/agents/listing-agent links from listings.

## Cadence

- Follow the canonical cadence in `../../README.md`.
- Domain deep dives run only on `FAIL` or `UNKNOWN` persisting for 2 consecutive daily runs.

## KPI Contract

- People lane error rate: target `< 2%` sustained.
- Cursor freshness in `mls_sync_state` for people jobs: should update regularly.
- Link integrity: no growing gap between recently updated listings and corresponding listing-agent linkage.

## Escalation States

- `PASS`: cursors moving and derivation outputs healthy.
- `FAIL`: sustained errors or linkage/cursor degradation; urgent Slack alert.
- `UNKNOWN`: insufficient evidence; escalate after 2 consecutive daily runs.

## Evidence Sources

- People lane run summaries/logs.
- `public.mls_sync_state` rows for people jobs.
- Table-level trend checks for `mls_offices`, `mls_agents`, `mls_listing_agents`.

## Suggested Checks (Adaptive)

1. Compare full-crawler and recent-lane processed volumes for consistency.
2. Track people cursor movement and detect prolonged staleness.
3. Sample recently updated listings and verify linked people entities exist.
4. Monitor repeated write conflicts or duplicate-link churn signals.

## Adaptation Rule

The agent should iterate on query design and sampling strategy as evidence improves, without changing KPI thresholds/escalation semantics unless explicitly approved.

## Reporting

- Daily Slack summary with health state, trend changes, and risk notes.
- Urgent Slack alert on sustained derivation failure or stuck cursor.
