# AEGIS Threat Intel Target Architecture

**What this is:** where the agent is deliberately headed. The companion to **`ARCHITECTURE.md`** (what runs today). When something here ships, it moves *out* of this doc and *into* `ARCHITECTURE.md`.

**Last updated:** 2026-09-13

## Direction

Stay narrow, cheap, and honest. This agent's entire value is filtering noise down to real signal at near-zero cost — it should never grow write access, never grow customer-facing scope, and never need more than free-pool reasoning to do its job. Anything added here should sharpen the filter or the escalation path, not widen what the agent is allowed to touch.

## Planned Capabilities

- **Resolve the `aegis-ceo` overlap** — once this agent's trial run is reviewed and approved, work with Hamid/`aegis-ceo` to retire or pause `aegis-ceo`'s own `/cve-watch` schedule so there is one clear owner of recurring CVE monitoring instead of two independent watches that could disagree or double-report. Depends on: the trial run actually being approved first (per CLAUDE.md's "Initial scope") — this is explicitly not this agent's call to make unilaterally.
- **Structured escalation acknowledgment** — today escalation is fire-and-report; a future version could confirm `aegis-ceo` actually saw a critical finding (not just that the message sent), closing the loop on genuinely urgent items. Depends on: `aegis-ceo` having a receiving playbook or read-receipt mechanism to check against.
- **Second real source beyond NVD** — today `/scan-threats` leans on NVD plus ad-hoc `WebSearch`; a dedicated second structured feed (e.g. a vendor security-advisory RSS relevant to the actual stack) would reduce reliance on search-result parsing for anything beyond NVD's formal CVE coverage. Depends on: identifying a real, reliably-available source — not adding one speculatively.

Next ideas, unordered: a lightweight severity-trend view (is the fleet's exposure surface getting bigger or smaller over time, from `memory/finding-history.json`), and confirming whether `corp-orchestrator`'s `threat_intel` department agent publishes anything this agent could safely treat as a secondary reference (never a dependency, per the ground-truth separation).
