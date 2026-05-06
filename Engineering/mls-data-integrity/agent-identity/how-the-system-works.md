---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# How The System Works

This file is the stable top-level model for the MLS listings data ecosystem.
All paths below reference production-branch repository locations.

## Operating Boundary

Primary control plane:

- Supabase cron invokes Supabase Edge Functions.
- Edge Functions read MLS Grid and write Supabase Postgres + Storage.
- Thumbnail tasks are published to Google Pub/Sub and processed by Cloud Run worker.

Primary external systems:

- MLS Grid API (Property, OpenHouse resources)
- Supabase (Postgres, Storage, Edge Functions)
- Google Cloud (Pub/Sub, Cloud Run)

## Canonical Lanes

### 1) Property sync (`mls_listings`)

Primary functions:

- `supabase/functions/mls-sync-property-reconcile/index.ts`
- `supabase/functions/mls-sync-property-incremental/index.ts`
- `supabase/functions/mls-sync-property-enrich/index.ts`

Support/optional functions:

- `supabase/functions/non_active_seeding_functions/mls-sync-property-seed/index.ts`
- `supabase/functions/mls-sync-property-status-repair/index.ts`

Primary outputs:

- `public.mls_listings`
- `public.mls_status_events`
- `public.mls_price_events`

Primary controls:

- Shared lease: `property_pipeline`
- Cursor/state: `public.mls_sync_state`

### 2) MLS media ingestion

Primary functions:

- `supabase/functions/mls_media_seed/index.ts`
- `supabase/functions/mls_media_change/index.ts`
- `supabase/functions/mls_media_reconcile/index.ts`

Storage/data outputs:

- `assets` bucket objects under `mls/<listingId>/<NNN>.<ext>`
- `public.mls_media`
- media metadata fields in `public.mls_listings`

Primary controls:

- Shared lease: `mls_media_pipeline`
- Claim RPCs:
  - `public.mls_claim_listings_for_media_seed`
  - `public.mls_claim_listings_for_media_change`

### 3) Thumbnail generation

Enqueue producers:

- MLS lanes above (`seed/change/reconcile`) for MLS source media paths.
- `supabase/functions/property_media_thumbnail_enqueue/index.ts` for non-MLS media paths.

Worker:

- `scripts/thumbnail-pubsub-worker.ts`

Queue/worker behavior:

- At-least-once queue semantics.
- Worker idempotency via storage existence check + upsert upload.

### 4) Lifecycle prune and remediation

Automated prune lane:

- `supabase/functions/mls_media_prune_lifecycle/index.ts`
- Candidate RPC: `public.mls_claim_lifecycle_prune_candidates`

Manual remediation scripts:

- `scripts/archive/prune-non-active-mls-media.ts`
- `scripts/reseed-pruned-listings-media.ts`

### 5) People lane

Functions:

- `supabase/functions/mls_people_sync/index.ts`
- `supabase/functions/mls_people_recent/index.ts`

Outputs:

- `public.mls_offices`
- `public.mls_agents`
- `public.mls_listing_agents`

### 6) Open houses lane

Functions:

- `supabase/functions/mls_open_houses_incremental/index.ts`
- `supabase/functions/mls_open_houses_reconcile/index.ts`

Outputs:

- `public.mls_open_houses`

## Cross-Lane Data/Control Surfaces

Core tables:

- `public.mls_listings`
- `public.mls_media`
- `public.mls_sync_state`
- `public.job_leases`

Core control RPCs:

- `public.try_acquire_job_lease`
- `public.release_job_lease`
- `public.mls_claim_listings_for_media_seed`
- `public.mls_claim_listings_for_media_change`
- `public.mls_claim_lifecycle_prune_candidates`
- `public.mls_listings_needing_enrichment`

## Stable Contract vs Live Discovery

Stable contract in this repository:

- Lane boundaries and ownership model
- Canonical function paths
- Core tables, leases, cursors, and claim RPCs
- Incident and reporting behavior (see `../shared-controls/README.md`)

Must be discovered live each run window:

- Exact cron expressions and timing offsets
- Current queue depth and worker error rates
- Current Cloud Run runtime posture for reconciliation/backfill jobs
