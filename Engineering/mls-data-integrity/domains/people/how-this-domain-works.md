---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# People: How This Domain Works

## Domain Boundary

This domain derives and maintains people-related MLS entities from listing data.

## Primary Functions

- `supabase/functions/mls_people_sync/index.ts` (full crawler)
- `supabase/functions/mls_people_recent/index.ts` (recent incremental)

## Data Flow

1. Read listing-derived source data (`mls_listings` payload fields).
2. Extract/normalize office and agent identities.
3. Upsert offices and agents.
4. Maintain listing-agent links.

## Core Tables

- `public.mls_listings` (source dependency)
- `public.mls_offices`
- `public.mls_agents`
- `public.mls_listing_agents`
- `public.mls_sync_state` (cursor state)

## Controls and Invariants

- Cron auth secret required for scheduled invocation.
- Cursor progression in `mls_sync_state` is required for deterministic coverage.
- Upsert/link refresh model should remain idempotent.

## Dependencies

- Depends on property-sync quality and listing identity consistency.
- If listing ingestion stalls or listing identity fields degrade, people derivation quality degrades.

## Known Residual Risks

- Lease-locking posture for people lanes is weaker than some other domains (requires explicit runtime policy decision).
- Exact production cadence must be discovered from live scheduler state.
