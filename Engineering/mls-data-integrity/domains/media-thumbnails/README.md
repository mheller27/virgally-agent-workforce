---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Domain: Media + Thumbnails

Primary contract files:

- `how-this-domain-works.md`
- `runbook.md`

Direct sources of truth:

- `../../supabase/functions/mls_media_seed/index.ts`
- `../../supabase/functions/mls_media_change/index.ts`
- `../../supabase/functions/mls_media_reconcile/index.ts`
- `../../supabase/functions/property_media_thumbnail_enqueue/index.ts`
- `../../supabase/functions/mls_media_prune_lifecycle/index.ts`
- `../../scripts/thumbnail-pubsub-worker.ts`
- `../../scripts/reseed-pruned-listings-media.ts`
