---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Open Houses: How This Domain Works

## Domain Boundary

This domain maintains open-house truth linked to listing UUIDs.

## Primary Functions

- `supabase/functions/mls_open_houses_incremental/index.ts`
- `supabase/functions/mls_open_houses_reconcile/index.ts`

## Data Flow

1. Pull OpenHouse data from MLS Grid.
2. Resolve MLS identifiers (`ListingKey`, `ListingId`) to local listing UUIDs.
3. Upsert in-window open-house rows.
4. Reconcile lane removes stale in-window rows missing from MLS response.

## Core Tables and RPCs

Core tables:

- `public.mls_open_houses`
- `public.mls_listings`
- `public.mls_sync_state` (incremental lane)

Core controls:

- Lease keys for incremental/reconcile open-house lanes.
- Cron auth secret for scheduled invocation.

## Controls and Invariants

- Incremental lane must maintain cursor progression.
- Reconcile lane must preserve in-window truth by upsert + stale-delete behavior.
- Listing identity resolution must remain deterministic.

## Dependencies

- Depends on healthy listing sync for UUID mapping.
- Depends on MLS source availability and rate limits.

## Known Residual Risks

- Exact scheduler cadence and overlap pressure are live-runtime concerns.
- Mapping quality depends on listing identity consistency in `mls_listings`.
