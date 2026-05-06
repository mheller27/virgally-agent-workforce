---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Property Sync (`mls_listings`): How This Domain Works

## Domain Boundary

This domain maintains listing truth in `public.mls_listings` and dependent event/index outputs.

## Primary Lanes

Reconcile:

- `supabase/functions/mls-sync-property-reconcile/index.ts`
- Broad alignment pass with cursor resume and lease protection.

Incremental:

- `supabase/functions/mls-sync-property-incremental/index.ts`
- Recent-change ingestion with cursor continuation and stale `nextLink` recovery behavior.

Enrich:

- `supabase/functions/mls-sync-property-enrich/index.ts`
- Candidate selection via `public.mls_listings_needing_enrichment`.
- Cooldown behavior aligned through `enrich_failed_at`.

Support/optional:

- `supabase/functions/non_active_seeding_functions/mls-sync-property-seed/index.ts`
- `supabase/functions/mls-sync-property-status-repair/index.ts`

## Core Tables and RPCs

Core tables:

- `public.mls_listings`
- `public.mls_sync_state`
- `public.mls_status_events`
- `public.mls_price_events`
- `public.mls_listing_tile_index` (trigger-driven downstream index)

Core RPCs:

- `public.try_acquire_job_lease`
- `public.release_job_lease`
- `public.mls_listings_needing_enrichment`
- `public.refresh_mls_listing_tile_index_v1`

## Controls and Invariants

- Reconcile and incremental share lease `property_pipeline`.
- Cursor continuity uses `mls_sync_state` fields (`last_mod_ts`, `last_next_link`, `last_listing_key`).
- Cron auth must be enforced for scheduled lanes.
- Upsert semantics are required for restart safety.

## Dependencies

- Media lanes depend on listing status/update/media fields in `mls_listings`.
- People/open-house lanes depend on reliable listing identity mapping.

## Known Residual Risks

- Exact production cron cadence and offset map are not encoded here.
- Some support lanes may be unscheduled/legacy and need live confirmation.
- Runtime assurance still depends on periodic evidence collection (not static code inspection only).
