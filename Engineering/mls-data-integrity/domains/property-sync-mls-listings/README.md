---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Domain: Property Sync (`mls_listings`)

Primary contract files:

- `how-this-domain-works.md`
- `runbook.md`

Direct sources of truth:

- `../../supabase/functions/mls-sync-property-reconcile/index.ts`
- `../../supabase/functions/mls-sync-property-incremental/index.ts`
- `../../supabase/functions/mls-sync-property-enrich/index.ts`
- `../../supabase/functions/mls-sync-property-status-repair/index.ts`
- `../../supabase/functions/non_active_seeding_functions/mls-sync-property-seed/index.ts`
- `../../supabase/migrations/0000_2026_04_30_production_baseline.sql`
