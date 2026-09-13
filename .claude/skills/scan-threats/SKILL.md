---
name: scan-threats
description: Poll real CVE/security-news sources, filter against AEGIS's actual stack, synthesize a plain-language finding, and escalate only if it's real
allowed-tools: Read, Write, Bash, WebFetch, WebSearch, mcp__trinity__report, mcp__trinity__list_reports, mcp__trinity__list_channel_groups, mcp__trinity__send_group_message, mcp__trinity__chat_with_agent
user-invocable: true
metadata:
  version: "1.0"
  created: 2026-09-13
  author: aegis-threat-intel
---

# Scan Threats

## Purpose

One real polling cycle: check real sources, filter hard against what actually matters to AEGIS, and say either "nothing relevant this cycle" or a short, honest, plain-language finding — then escalate only the latter. Never fabricate a scan result.

## Process

### Step 0: Confirm the feed is real

If this is the first run, or it's been a while, run `/audit-feed-source` first (or at minimum repeat its Step 2 NVD reachability probe inline). If nothing is reachable, stop and say so — do not report "nothing relevant" when the truth is "couldn't check." A failed check and a clean check are different facts.

### Step 1: Load the stack profile

Read `memory/stack-profile.md` (written by `/update-stack-profile`). If it doesn't exist, use the CLAUDE.md defaults as a one-time fallback (Postgres, Docker/Docker Compose, Oracle Cloud, nginx, GitHub Actions, typical SMB infra: SSO providers, common CMS/ERP, AWS/GCP/Azure, and the LLM/agent-security space itself) and tell Hamid you're running `/update-stack-profile` is recommended so this isn't re-derived every cycle.

### Step 2: Query real sources

NVD, per watch-list keyword, respecting the rate limit (space calls if `NVD_API_KEY` isn't set):

```bash
curl -sS -m 15 -H "apiKey: ${NVD_API_KEY:-}" \
  "https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=<keyword>&pubStartDate=$(date -u -v-1d +%Y-%m-%dT00:00:00.000)&pubEndDate=$(date -u +%Y-%m-%dT00:00:00.000)"
```

Batch related keywords into as few consolidated passes as the API allows rather than one call per component (token-saving habit from `aegis-infra`). Also run one `WebSearch` pass for security-news disclosures that wouldn't show up as a formal CVE yet (e.g. new prompt-injection/agent-security research relevant to AEGIS's own product category).

If a source that Step 0 confirmed reachable fails mid-run, say so for that source specifically rather than silently dropping it from the results.

### Step 3: Filter hard

Discard anything CVSS < 7.0 unless it directly affects a watch-list component with a known exploit in the wild. For each finding that clears the bar, note: id (CVE or news item), affected component + version range, severity, whether it plausibly touches AEGIS's own stack vs. a customer's typical stack (or both), and whether a patch/mitigation exists.

Most cycles will filter down to zero. That is the expected, correct outcome most of the time — say "nothing relevant this cycle" plainly rather than padding the report to look busy.

### Step 4: Cross-check against prior findings

Read `memory/finding-history.json` (create on first run). Only surface a genuinely new finding, or a material status change on a prior one (patch released, exploit now public) — don't re-flag something already reported and unchanged.

If a finding looks like it plausibly overlaps with something `aegis-ceo`'s own `/cve-watch` may already have reported (same CVE, similar timeframe), note that ambiguity explicitly in the escalation rather than presenting it as a clean new catch — see CLAUDE.md's "Known overlap, not yet resolved."

### Step 5: Synthesize

```
## Threat Scan — [date/time]

[N] new relevant finding(s):

- **[CVE id or news item]** ([severity]) — [component] — [one-line impact, plain language] — [patch/mitigation status] — [AEGIS-own-stack / customer-typical / both]

[If nothing new: "No new relevant findings since the last check. [N] sources queried, [M] items reviewed."]
```

### Step 6: Persist

Append the cycle's result (even a clean one, briefly) to `memory/finding-history.json` with timestamp and status, so the next run's Step 4 has something to compare against.

### Step 7: Escalate real findings only

If Step 5 produced at least one real finding, follow CLAUDE.md's **Escalation channels** in order (Trinity report → operator-queue alert if urgent → `aegis-ceo` chat if permitted → Slack `#aegis-threat-intel` if bound), and report honestly which of those actually succeeded. A "nothing relevant" cycle does not need channels 2–4 — at most publish it via the Trinity report so the append-only history shows the cycle ran, or skip publishing entirely and rely on `memory/finding-history.json`.

## Outputs

- A direct threat-scan summary in chat
- Updated `memory/finding-history.json`
- A Trinity report (when deployed and there's something worth recording)
- Escalation via the channels in CLAUDE.md, only for real findings — never for a clean cycle
