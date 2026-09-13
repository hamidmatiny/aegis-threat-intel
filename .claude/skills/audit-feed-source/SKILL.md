---
name: audit-feed-source
description: Confirm what CVE/security-news source is actually wired and reachable right now — never claim a scan happened if nothing real is connected
allowed-tools: Read, Write, Bash, WebFetch, WebSearch, AskUserQuestion
user-invocable: true
metadata:
  version: "1.0"
  created: 2026-09-13
  author: aegis-threat-intel
---

# Audit Feed Source

## Purpose

Establish, honestly, whether this agent can actually reach a real CVE/security-news source right now. "NVD is a well-known public API" is a fact about the world, not a fact about whether this session can reach it — confirm before `/scan-threats` ever claims to have checked anything.

## Process

### Step 1: Check declared credentials

Read `.env` (if present) for `NVD_API_KEY`. Its absence is not a blocker — NVD's public tier works unauthenticated, just more heavily rate-limited — but note it either way, since it affects how many watch-list keywords `/scan-threats` can query per run before hitting a 429.

### Step 2: Probe the NVD API directly

```bash
curl -sS -m 10 -o /dev/null -w "%{http_code}\n" \
  -H "apiKey: ${NVD_API_KEY:-}" \
  "https://services.nvd.nist.gov/rest/json/cves/2.0?resultsPerPage=1"
```

A `200` confirms the feed is genuinely reachable from this environment right now. Anything else (network error, `403`, `429`) means it is not — report the actual status, don't assume success.

### Step 3: Probe WebSearch/WebFetch for security-news coverage

Run one real `WebSearch` (e.g. "recent CVE disclosure docker postgres") and confirm results actually come back — this is the secondary source `/scan-threats` uses for disclosures that wouldn't show up as a formal CVE yet (prompt-injection research, agent-security news). If `WebSearch`/`WebFetch` aren't available in this session, say so.

### Step 4: Check for a stack profile

Read `memory/stack-profile.md` (created by `/update-stack-profile`). If it doesn't exist yet, this audit still passes on the feed-reachability question, but `/scan-threats` has nothing to filter against yet — flag that as a real gap, not something to paper over with the CLAUDE.md defaults from memory.

### Step 5: Report ground truth plainly

```
## Feed Source Audit — [date]

**NVD API reachable:** yes / no (HTTP [code])
**NVD_API_KEY configured:** yes / no (affects rate limit only)
**WebSearch/WebFetch available:** yes / no
**Stack profile present:** yes ([N] components) / no — run /update-stack-profile first

**Can /scan-threats do a real run right now?** yes / no — [why]
```

If anything is missing, ask Hamid directly (`AskUserQuestion` interactively, or a plain question in the reply headless) rather than proceeding with a partial or fabricated check — this is exactly the "ask if none is obviously wired" case from the initial scope.

### Step 6: Escalate, don't fix silently

This skill is read-only. If a feed genuinely isn't wired, that's a setup gap for Hamid or `aegis-infra` to close (e.g. an `NVD_API_KEY` to add), not something this skill works around.

## Outputs

- A ground-truth reachability report (chat and, on Trinity, nothing published — this is a setup check, not a finding)
- No configuration changes
