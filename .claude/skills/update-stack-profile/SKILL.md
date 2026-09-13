---
name: update-stack-profile
description: Update the watch-list definition (what counts as AEGIS's stack and its typical customer stack) as it evolves
allowed-tools: Read, Write, Edit, AskUserQuestion
user-invocable: true
metadata:
  version: "1.0"
  created: 2026-09-13
  author: aegis-threat-intel
---

# Update Stack Profile

## Purpose

Keep the filter definition `/scan-threats` runs against current. The watch list is what makes this agent useful instead of noisy — it belongs in a maintained memory file, not only baked into CLAUDE.md defaults that go stale silently.

## Process

### Step 1: Load or seed the profile

Read `memory/stack-profile.md`. If it doesn't exist, seed it from the CLAUDE.md defaults (Postgres, Docker/Docker Compose, Oracle Cloud, nginx, GitHub Actions, typical SMB infra: SSO providers like Okta/Azure AD, common CMS/ERP, AWS/GCP/Azure, and the LLM/agent-security space itself).

### Step 2: Ask what changed

Use `AskUserQuestion` (interactively) or ask plainly in the reply (headless): what component was added, retired, or reclassified (e.g. moved from "AEGIS's own stack" to "customer-typical only," or vice versa)? Don't guess at a change that wasn't stated.

### Step 3: Update the profile

Write `memory/stack-profile.md` as a simple, structured list `/scan-threats` can parse at a glance:

```markdown
# AEGIS Stack Profile

Last updated: [date]

## AEGIS's own stack (highest priority)
- Docker / Docker Compose (Docker Engine, containerd)
- PostgreSQL
- nginx
- Oracle Cloud Infrastructure (the VM AEGIS runs on)
- GitHub Actions (CI/CD)

## Customer-typical SMB infrastructure (secondary priority)
- SSO providers: Okta, Azure AD / Entra
- Common CMS/ERP software
- Popular cloud providers: AWS, GCP, Azure

## Adjacent watch (context, not primary)
- The LLM/agent-security space itself — new prompt-injection or agent-security disclosures relevant to AEGIS's own product category

## Change log
- [date]: [what changed and why]
```

Keep the change log — it's what lets a later `/reconcile-docs` or a human review see why the profile looks the way it does, not just what it currently says.

### Step 4: Confirm

Report the diff plainly: what was added, removed, or reclassified, and where in the priority tiers it now sits.

## Outputs

- Updated `memory/stack-profile.md`, with a change log entry
