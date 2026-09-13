---
name: update-dashboard
description: Refresh dashboard.yaml with current metrics from memory files
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
user-invocable: true
metadata:
  version: "1.0"
  created: 2026-09-13
  author: aegis-threat-intel
---

# Update Dashboard

Refresh `dashboard.yaml` with current metrics gathered from this agent's memory files.

## Process

### Step 1: Gather Metrics

Read the agent's data sources to collect current values:
- `memory/finding-history.json` — last scan timestamp, count of open (unresolved) findings
- `memory/stack-profile.md` — count of watch-list components
- The result of the last `/audit-feed-source` run, if recorded, for the "Feed Source" status
- Recent git activity: `git log --oneline -10`

### Step 2: Update Dashboard

Read `dashboard.yaml`, update widget values with fresh data:
- Update the `updated` timestamp to now
- "Feed Source" status widget: green if the last audit found the source reachable, yellow if partially, red if unreachable, gray if never audited
- "Last Scan" / "Open Findings" from `memory/finding-history.json`
- "Stack Profile Components" from `memory/stack-profile.md`
- "Recent Scan Results" list from the last few entries in `memory/finding-history.json`

Write the updated `dashboard.yaml`.

### Step 3: Publish a KPI snapshot report (Trinity)

If the `mcp__trinity__report` tool is available (i.e. running on Trinity), also publish the same headline numbers as a report so they accumulate as history alongside the live snapshot:

- `report_type`: `aegis_threat_intel.kpi_snapshot`
- `display_hint`: `kpi`
- `payload`: `{ "tiles": [ {"label": "Open Findings", "value": "..."}, {"label": "Feed Source", "value": "..."}, {"label": "Last Scan", "value": "..."} ] }`

Skip this step silently if the tool isn't available — the dashboard refresh above still succeeds.

### Step 4: Confirm

Report what was updated:
```
Dashboard refreshed:
- Feed source: [old] → [new]
- Open findings: [old] → [new]
- Last updated: [timestamp]
```

Note: On Trinity remote, the dashboard path is `/home/developer/dashboard.yaml`.

## Outputs

- Updated `dashboard.yaml` with current metrics
