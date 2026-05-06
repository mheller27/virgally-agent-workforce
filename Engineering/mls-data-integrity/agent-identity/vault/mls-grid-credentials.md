---
layer: bootstrap
status: immutable
write-policy: human-approval-only
---

# MLS Grid Credentials

## [IMMUTABLE-BOOTSTRAP] - DO NOT SHARE OR REPEAT IN CHAT

Do not store plaintext credentials in this repository.

Store MLS Grid secrets in platform secret storage (env/MCP/secret manager) and reference only the keys here:

- Secret key for username: `MLS_GRID_USERNAME`
- Secret key for password: `MLS_GRID_PASSWORD`
- Injection target: OpenClaw runtime environment / MCP configuration
