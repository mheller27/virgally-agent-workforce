---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Media + Thumbnails Runbook

## Goal

Detect and escalate integrity regressions in media ingestion and thumbnail generation quickly, with adaptive checks.

## Cadence

- Follow the canonical cadence in `../../README.md`.
- Domain deep dives run only on `FAIL` or `UNKNOWN` persisting for 2 consecutive daily runs.

## KPI Contract

- Media lane error rate (`seed`, `change`, `reconcile`): target `< 2%` sustained.
- Thumbnail enqueue failure rate: target `< 0.5%`.
- Thumbnail availability SLA for new source media: initial target `p95 <= 15 minutes`.
- Shared media lease lock pressure (`mode: "locked"`): should be low and non-growing.

## Escalation States

- `PASS`: metrics in range, no urgent integrity risk.
- `FAIL`: threshold breach or hard-gate failure; immediate Slack incident alert.
- `UNKNOWN`: insufficient evidence; if persistent across 2 consecutive daily runs, escalate.

## Evidence Sources

- Edge function run summaries/log payloads.
- Cloud Run logs/metrics for thumbnail worker.
- Pub/Sub subscription backlog metrics.
- Supabase table samples (`mls_listings`, `mls_media`, `storage.objects`).

## Suggested Checks (Adaptive)

1. Compare per-lane `errors_count` vs `processed_listings` over rolling window.
2. Track `thumbnail_enqueue_failures`, `thumbnail_tasks_enqueued`, and `thumbnail_missing_backfill_tasks_enqueued`.
3. Sample recent `mls_media.media_url` values, derive `.../thumbnails/...`, and verify object existence.
4. Monitor shared `mls_media_pipeline` lock contention trend for schedule pressure.
5. Correlate queue backlog age with worker 5xx/retry patterns.

## Adaptation Rule

The agent should refine query templates and sampling methods over time while preserving KPI definitions, thresholds, and escalation semantics.

## Reporting

- Daily Slack summary: status by gate/KPI, trend deltas, top risks, next action.
- Urgent Slack alert: immediate when `FAIL` condition is triggered.
