---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Domain Memory

Each subfolder represents one operational domain with exactly two primary documents:

- `how-this-domain-works.md`
- `runbook.md`

Domain folders:

- `media-thumbnails/`
- `property-sync-mls-listings/`
- `people/`
- `open-houses/`

Design rule:

- Keep domain contracts durable and minimal.
- Keep volatile runtime snapshots out of these files.
- Use suggested checks (adaptive) rather than rigid one-off command sequences.
