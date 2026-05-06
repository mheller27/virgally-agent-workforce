---
layer: memory
status: adaptive
write-policy: agent-writeable
---

# Usage Logs

This file stores MLS Grid usage snapshots from daily Browser Tool scrapes.

## Log Entries

Use the format:

- Timestamp (UTC)
- Request Count
- Rate Limit
- Usage Percent
- Notes

Example:

- `2026-05-05T20:00:00Z | request_count=12345 | rate_limit=50000 | usage_percent=24.69 | note=normal`
