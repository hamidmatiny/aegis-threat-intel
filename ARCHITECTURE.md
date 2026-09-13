# AEGIS Threat Intel Architecture (Current State)

**What this is:** the agent as it actually runs today. For where it's deliberately headed, see the companion **`TARGET-ARCHITECTURE.md`**. When a target ships, it moves *out* of that doc and *into* this one.

**Last updated:** 2026-09-13

## Overview

AEGIS Threat Intel is a Trinity-compatible Claude Code agent with three purpose-built skills covering the full core mission (feed audit, threat scan, stack profile maintenance), plus the standard onboarding/dashboard/doc-reconciliation trio every agent in this fleet ships with. It is advisory-only and holds no write access to anything outside its own memory files and reports/messages it sends.

## Components

### Skills

- `/audit-feed-source` — read-only ground-truth check of NVD reachability and WebSearch/WebFetch availability
- `/scan-threats` — the core cycle: query real sources, filter against the stack profile, synthesize a finding, escalate if real
- `/update-stack-profile` — maintains `memory/stack-profile.md`, the filter definition
- `/onboarding` — setup progress tracker
- `/update-dashboard` — refreshes `dashboard.yaml` from memory files
- `/reconcile-docs` — checks CLAUDE.md, README, ARCHITECTURE docs, and skills stay mutually consistent

### Subagents

None yet.

### Data & State

- `onboarding.json` — setup progress
- `dashboard.yaml` — live findings/status snapshot
- `memory/stack-profile.md` — the watch-list definition (created by first `/update-stack-profile` run)
- `memory/finding-history.json` — log of prior scan results, real and clean (created by first `/scan-threats` run)

### Schedules

Declared in `template.yaml`, all shipped `enabled: false` pending the trial-run review described in CLAUDE.md's "Initial scope":

- Threat scan, every 6 hours (`/scan-threats`)
- Dashboard refresh, daily (`/update-dashboard`)
- Doc reconciliation, weekly (`/reconcile-docs`)

## Trinity Integration

Resources: 1 CPU / 1g memory (deliberately minimal — a free-pool, high-volume-polling role should be one of the cheapest agents in the fleet to run). Declared plugins: `agent-dev`, `trinity`, `utilities`. Declared credentials: `NVD_API_KEY` (optional — raises the public NVD rate limit). No MCP servers of its own — `.mcp.json.template` is empty; it uses Trinity's own injected `trinity` MCP tools (`report`, `list_reports`, `list_channel_groups`, `send_group_message`, `chat_with_agent`) once deployed.

This agent's tier, per `aegis-infra`'s assignment, is **free-pool via OmniRoute** (`gemini/gemini-3.7-flash`) — a Trinity-level auth-mode setting owned by `aegis-infra`/the admin, not declared in this repo. See CLAUDE.md's "Tier & model assignment" for the full picture, including the dependency on OmniRoute routing being confirmed live before this assignment actually takes effect.
