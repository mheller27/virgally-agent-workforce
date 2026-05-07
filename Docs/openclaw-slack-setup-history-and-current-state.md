# OpenClaw VPS Master Configuration

Last verified: 2026-05-07  
System: OpenClaw on VPS (`root@134.209.67.45`)  
Purpose: single source of truth for runtime config, permissions, platform integrations, and agent setup.

## 1) System Architecture (No Guesswork)

Three layers are intentionally separated:

1. OpenClaw platform code:
   - `/root/openclaw`
2. OpenClaw runtime/config/state:
   - `/root/.openclaw`
3. Agent brain/policy content:
   - `/root/virgally-agent-workforce`

Key principle:

- Host/runtime enforcement and agent policy docs are complementary:
  - host controls enforce,
  - brain docs govern behavior.

## 2) Canonical Sources of Truth

Use these files first when checking behavior:

- Runtime master config: `/root/.openclaw/openclaw.json`
- Runtime env secrets: `/root/.openclaw/.env`
- Agent runtime state: `/root/.openclaw/agents/<agent-id>/...`
- Workforce brain docs: `/root/virgally-agent-workforce/...`
- VSG local enforcement hooks: `/root/vsg/.git/hooks/pre-commit`, `/root/vsg/.git/hooks/pre-push`
- VSG auth mode helper: `/root/.local/bin/vsg-auth-mode`
- VSG token files:
  - `/root/.secrets/vsg_read.token`
  - `/root/.secrets/vsg_write.token`

## 3) OpenClaw Runtime Configuration (Live)

Source: `/root/.openclaw/openclaw.json`

### Global defaults (`agents.defaults`)

- Primary model: `openai/gpt-5.5`
- Registered models:
  - `openai/gpt-5.5`
  - `codex-cli/gpt-5.5`
- Default workspace: `/root/virgally-agent-workforce/admin`
- Agent runtime id: `pi`

### Agent list (`agents.list`)

- `main`
  - heartbeat:
    - `every: 0m`
    - `lightContext: true`
    - `isolatedSession: true`
- `mls-data-integrity`
  - workspace: `/root/virgally-agent-workforce/Engineering/mls-data-integrity`
  - agentDir: `/root/.openclaw/agents/mls-data-integrity/agent`
  - heartbeat:
    - `every: 24h`
    - `lightContext: true`
    - `isolatedSession: true`
    - `session: main`
    - prompt set to low-token daily PASS/FAIL/UNKNOWN reporting behavior

### Slack routing (`bindings`)

Two active routes to `mls-data-integrity`:

1. Channel-specific:
   - `channel: slack`
   - `peer.kind: channel`
   - `peer.id: C0B2W6WATKJ`
2. Slack fallback:
   - `channel: slack`
   - `accountId: "*"`

Effect:

- channel and DM traffic route to `mls-data-integrity` (no fallback to `main`).

### Slack channel config (`channels.slack`)

- `enabled: true`
- `mode: socket`
- `appToken` source: env key `SLACK_APP_TOKEN`
- `botToken` source: env key `SLACK_BOT_TOKEN`
- `groupPolicy: allowlist`
- `dmPolicy: pairing`
- allowlisted channel id: `C0B2W6WATKJ` (`requireMention: true`)

### Command ownership

- `commands.ownerAllowFrom` includes:
  - `slack:U0B1VFHBZ7V`

## 4) Platform Integrations and Status

### Slack

- Status: connected and active
- Mode: Socket Mode
- Routing: to `mls-data-integrity`
- DM behavior: pairing enabled and owner approved

### GitHub (`vsg`)

- Repo owner: `Virgally` org
- Repo URL: `https://github.com/Virgally/vsg.git`
- Default/protected branch: `conversion-tweaks`
- Token model:
  - read token (`Contents: Read-only`)
  - write token (`Contents: Read and write`)
- Auth mode:
  - default read
  - write only in explicit approved write windows

### Supabase

- Status: policy intent documented, credential wiring deferred
- Target posture: read-only (`mls_auditor`) for autonomous monitoring

### MLS Grid

- Status: key names/policy documented, credential wiring deferred
- Required secret keys:
  - `MLS_GRID_USERNAME`
  - `MLS_GRID_PASSWORD`

## 5) Permissions Model (Global vs Agent vs Host)

### Global baseline policy (applies to all agents)

Unless explicitly overridden, every OpenClaw agent in this workforce should follow the same baseline:

- `vsg` is strict:
  - read anytime,
  - write only with explicit approval,
  - branch format `agent/<slack-name>/<task-slug>`,
  - never direct-push protected branch,
  - never self-merge PR.
- Supabase/runtime is read-first by default:
  - schema/read diagnostics allowed,
  - mutation only with explicit approval + rollback path.
- `virgally-agent-workforce` is lightweight:
  - agents can update only their own brain memory-layer files,
  - immutable/bootstrap files remain human-controlled.
- Approval contract is required for any state-changing action:
  - target, scope, action, time window, rollback path.

Agent-specific differences are intentionally limited to:

- brain folder/path scope (where the agent reads and writes),
- file edit boundaries inside that brain,
- dedicated Slack app identity/routing.

### Host/VPS enforcement (hard)

- Immutable bootstrap protections via `chattr +i`
- Git hooks in `/root/vsg/.git/hooks` enforce branch policy
- Token files and auth mode control read/write capability

### OpenClaw runtime enforcement

- Agent workspace mapping in `openclaw.json`
- Slack routing bindings and channel policy
- Heartbeat cadence by agent

### Agent policy enforcement (prompt-injected docs)

- `mls-data-integrity` bootstrap docs define behavior/approval boundaries
- Most important file:
  - `/root/virgally-agent-workforce/Engineering/mls-data-integrity/agent-identity/permissions.md`
- Current implementation note:
  - `mls-data-integrity` `permissions.md` is used as the first concrete implementation of the global baseline policy and includes MDI-specific scope details.

## 6) Agent Cards (Current)

### Agent: `mls-data-integrity`

- Workspace:
  - `/root/virgally-agent-workforce/Engineering/mls-data-integrity`
- Slack:
  - receives channel + DM via bindings
  - dedicated Slack app identity model is in use (one app identity per agent)
- Heartbeat:
  - enabled every 24h
- Core mission:
  - MLS data-integrity operator with PASS/FAIL/UNKNOWN reporting model
- Immutable bootstrap files:
  - tracked in `/root/.openclaw/agents/mls-data-integrity/bootstrap-locked-files.json`
  - includes `agent-identity/*` core files and `shared-controls/README.md`
- Writable memory layer:
  - `domains/**`
  - `system-context/usage-logs.md`

### Agent: `main`

- Workspace:
  - `/root/virgally-agent-workforce/admin`
- Heartbeat:
  - `0m` (effectively disabled cadence)
- Current role:
  - fallback/default agent; retained, not primary operational agent

## 6.1) Agent-Specific Isolation Model

Each agent is isolated by identity and brain scope:

- Brain scope:
  - each agent is bound to a specific subfolder under `virgally-agent-workforce`.
- Edit scope:
  - each agent can update only allowed files in its own brain scope.
- Slack identity scope:
  - each agent uses a dedicated Slack app identity for clean routing and clear operator context.

Current examples:

- `mls-data-integrity` brain: `/root/virgally-agent-workforce/Engineering/mls-data-integrity`
- `main` brain: `/root/virgally-agent-workforce/admin`

## 7) VSG Safety Controls (Authoritative)

### Local VPS controls

In `/root/vsg`:

- `pre-commit` blocks commit on `conversion-tweaks`
- `pre-push` blocks push to `conversion-tweaks`
- branch name required:
  - `agent/<slack-name>/<task-slug>`

### GitHub server-side controls

Classic branch protection is configured for `conversion-tweaks` in org repo settings.

Live validation from VPS:

- direct push to `conversion-tweaks` was rejected with:
  - `GH006: Protected branch update failed`
  - `Changes must be made through a pull request`

Result:

- both local and server-side protections are active.

## 8) Operational Runbooks

### VSG read/write window

Default:

- `/root/.local/bin/vsg-auth-mode read`

Before explicitly approved code-write task:

- `/root/.local/bin/vsg-auth-mode write`

After branch push completes:

- `/root/.local/bin/vsg-auth-mode read`

### Branch workflow for VSG

For approved write tasks:

1. Create branch:
   - `agent/<slack-name>/<task-slug>`
2. Make edits only on that branch.
3. Commit and push that branch.
4. Never commit/push to `conversion-tweaks` directly.
5. Never self-merge PR.

### Git failure handling (policy)

On push/commit failure:

1. Stop retries.
2. Report exact failure class (`auth`, `permission`, `protected-branch`, `policy-hook`).
3. Keep local branch/commits intact.
4. Ask operator for next action.

## 9) Verification Commands

```bash
# OpenClaw status snapshot
cd ~/openclaw && pnpm -s openclaw status --json

# Show OpenClaw config
python3 -m json.tool /root/.openclaw/openclaw.json

# Show current VSG auth mode
/root/.local/bin/vsg-auth-mode show

# Validate GitHub read token reachability
GIT_TERMINAL_PROMPT=0 GIT_ASKPASS=/root/.local/bin/git-askpass-vsg-read \
git ls-remote https://github.com/Virgally/vsg.git HEAD

# Validate GitHub write token reachability
GIT_TERMINAL_PROMPT=0 GIT_ASKPASS=/root/.local/bin/git-askpass-vsg-write \
git ls-remote https://github.com/Virgally/vsg.git HEAD
```

## 10) Outstanding Work

1. Supabase credential wiring (`mls_auditor` read-only) and verification
2. MLS Grid credential injection (`MLS_GRID_USERNAME`, `MLS_GRID_PASSWORD`)
3. Optional: scaffold curated bootstrap docs for `main` admin workspace
4. Optional: standardize a reusable onboarding template for future agents

## 11) Scope and Non-Propagation Clarification

Local `.git` internals are clone-specific:

- `/root/vsg/.git/hooks/*` affects VPS clone only.
- They do not auto-propagate to GitHub contents.
- They do not auto-propagate to other machine clones (for example local MacBook clone).

