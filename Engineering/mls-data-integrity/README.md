# MLS Data Integrity Core Memory

This directory is the primary memory contract for the OpenClaw MLS integrity agent.

## Core Execution Loop

1. Observe runtime systems.
2. Evaluate against domain KPIs and shared-controls invariants.
3. Classify state as `PASS` / `FAIL` / `UNKNOWN`.
4. Investigate any `FAIL` state or persistent `UNKNOWN`.
5. Act only within permissions.
6. Report via Slack daily or urgently, based on severity.

## Cadence Model (Low-Token Default)

- Baseline execution: once per day.
- Deep investigations: only when a `FAIL` state appears or `UNKNOWN` persists.
- Emergency alerts: event-driven and immediate.
- This section is the canonical cadence authority for all domain runbooks.

## Primary Structure

- `agent-identity/` - stable operating identity, mission, system model, and permissions.
- `domains/` - per-domain contracts (`how-this-domain-works.md`, `runbook.md`).
- `shared-controls/` - global invariants, incident rules, and reporting/escalation contract.
- `memory-governance.md` - bootstrap-vs-memory classification and write policy.
- `system-context/` - supporting adaptive notes/logs only.

## Role Boundaries

- Root `README.md` defines the operating frame.
- `agent-identity/` defines immutable authority and mission.
- `domains/` defines domain-specific health logic.
- `shared-controls/` defines cross-domain evaluation, invariants, incident mode, and reporting.
- `system-context/` stores supporting adaptive notes/logs only.

## Bootstrap vs Memory

- Immutable bootstrap law and mutable adaptive memory are intentionally separated.
- See `memory-governance.md` for authoritative file classification and edit rules.

## Direct Sources Of Truth

- `supabase/functions/` for lane behavior and runtime control logic.
- `scripts/` for workers and operator utilities.
- `supabase/migrations/` for schemas, RPC definitions, and control primitives.
- All file paths in this contract are repository-root relative (not absolute host paths).

## Contract Design Rules

- Keep core-memory files minimal and durable.
- Do not hardcode volatile operational values (exact cron expressions, transient metric snapshots).
- For volatile data, query systems of record at runtime (Supabase, Google Cloud, repo code).
- Define policy in Layer 2 docs and enforce permissions in platform configuration.

## Expected Agent Behavior

- Provide daily Slack health reports.
- Send urgent Slack incident alerts immediately on hard failures.
- Execute state-changing actions only when explicitly instructed.
