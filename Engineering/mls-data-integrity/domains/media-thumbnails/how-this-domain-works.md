---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Media + Thumbnails: How This Domain Works

## Domain Boundary

This domain covers:

- MLS media ingestion (`seed`, `change`, `reconcile`)
- Thumbnail enqueue for MLS and non-MLS sources
- Thumbnail worker generation
- Media lifecycle prune/remediation interaction points

## End-to-End Flow

1. Claim/select listings for media processing.
2. Fetch MLS Property with media payload.
3. Upload source media to `assets/mls/...`.
4. Upsert `public.mls_media`, prune stale rows, update `public.mls_listings` media fields.
5. Publish thumbnail tasks for changed media (plus missing-thumbnail backfill checks on unchanged rows).
6. Worker writes derived thumbnail artifacts under `.../thumbnails/...`.
7. Lifecycle prune can remove non-keeper media for eligible listing statuses.

## Function Inventory

Media ingest:

- `supabase/functions/mls_media_seed/index.ts`
- `supabase/functions/mls_media_change/index.ts`
- `supabase/functions/mls_media_reconcile/index.ts`

Thumbnail enqueue:

- `supabase/functions/property_media_thumbnail_enqueue/index.ts` (non-MLS sources; excludes `mls/` by default)

Worker:

- `scripts/thumbnail-pubsub-worker.ts`

Lifecycle/remediation:

- `supabase/functions/mls_media_prune_lifecycle/index.ts`
- `scripts/archive/prune-non-active-mls-media.ts`
- `scripts/reseed-pruned-listings-media.ts`

## Core Tables and RPCs

Core tables:

- `public.mls_listings`
- `public.mls_media`
- `public.mls_sync_state`
- `storage.objects`

Core RPCs:

- `public.try_acquire_job_lease`
- `public.release_job_lease`
- `public.mls_claim_listings_for_media_seed`
- `public.mls_claim_listings_for_media_change`
- `public.mls_claim_lifecycle_prune_candidates`

## Controls and Invariants

- Shared lease for media ingest lanes: `mls_media_pipeline`.
- Cron auth secret required for scheduled Edge lanes (`MLS_SYNC_CRON_SECRET` / `x-cron-secret`).
- Storage writes are idempotent (`upsert: true` patterns).
- Queue is at-least-once; worker idempotency is required.

## Known Residual Risks

- Exact cron expressions and offsets are not encoded in this contract.
- Final steady-state role for `thumbnail-reconcile-worker` requires operator confirmation.
- `mls_media.thumbnail_url` remains compatibility-oriented, not guaranteed canonical thumbnail artifact path.
- Legacy thumbnail claim RPC surfaces may still exist externally and should be treated as unknown until confirmed.
