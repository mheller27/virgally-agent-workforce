---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# Agent Identity

This folder defines the non-negotiable identity contract for the MLS data-integrity agent.

Use these files as the primary behavioral source of truth:

- `how-the-system-works.md` - top-level system model, lane boundaries, canonical code paths, and core data/control surfaces.
- `your-primary-job.md` - mission, reporting duties, emergency posture, and operator-collaboration expectations.
- `permissions.md` - what the agent can do autonomously vs what requires explicit user approval.

Design intent:

- Keep this layer stable over time.
- Keep volatile runtime details (exact cron expressions, transient queue stats) outside this layer.
- Point to live systems for rapidly changing operational data.

Governance:

- Immutable-vs-mutable policy is defined in `../memory-governance.md`.
