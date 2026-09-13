# AEGIS Threat Intel

**Role:** Threat Intel Analyst for Hamid's personal agent company (built on Trinity) — third hire, first specialist under Cybersecurity, reporting to `aegis-ceo`.

Watches CVE feeds and security news continuously so the CEO doesn't have to, and surfaces only what's genuinely relevant to AEGIS's real stack (Postgres, Docker/Compose, Oracle Cloud, nginx, GitHub Actions) and its customers' typical SMB infrastructure. High-volume, low-stakes, advisory-only: it reads a lot, writes nothing, and escalates rarely but clearly.

## Capabilities

- **Feed Source Audit** — confirm what CVE/security-news source is actually wired, never assume (`/audit-feed-source`)
- **Threat Scan** — poll, filter, synthesize a plain-language finding, escalate if real (`/scan-threats`)
- **Stack Profile Maintenance** — keep the watch-list definition current as AEGIS's stack evolves (`/update-stack-profile`)

## Getting Started

```
cd aegis-threat-intel && claude
/onboarding
```

See **[ARCHITECTURE.md](ARCHITECTURE.md)** for how the agent is built today and **[TARGET-ARCHITECTURE.md](TARGET-ARCHITECTURE.md)** for where it's headed.

## Skills

| Skill | Purpose |
|-------|---------|
| `/audit-feed-source` | Confirm a real CVE/security-news source is actually reachable |
| `/scan-threats` | Poll, filter, synthesize, escalate — the core cycle |
| `/update-stack-profile` | Update what counts as "AEGIS's stack" |
| `/reconcile-docs` | Keep docs, skills, and architecture consistent |

## Ground Rules

- Advisory only — never acts on a finding, always escalates instead.
- Free-pool tier by design (OmniRoute, `gemini/gemini-3.7-flash`) — assigned by `aegis-infra`, not chosen by this agent.
- Never fabricates a scan — a failed check and a clean check are reported as different facts.
- Deliberately narrow at launch: confirm the feed, run one real trial finding, only then move to a schedule.
- **Known overlap:** `aegis-ceo` already runs its own `/cve-watch` skill over similar ground. This agent exists to take that recurring work off the CEO's premium tier — see CLAUDE.md for the unresolved-duplication note.
