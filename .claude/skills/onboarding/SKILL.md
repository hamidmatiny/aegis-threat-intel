---
name: onboarding
description: Track your setup progress — shows what's done, what's next, and walks you through each step
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
user-invocable: true
metadata:
  version: "1.0"
  created: 2026-09-13
  author: aegis-threat-intel
---

# Onboarding

Track and continue your setup progress. This skill reads `onboarding.json`, shows your current status, and walks you through the next incomplete step.

## Process

### Step 1: Load State

Read `onboarding.json` from the agent root directory. If it doesn't exist, inform the user that onboarding is complete or the file was removed.

### Step 2: Show Progress

Display a checklist grouped by phase. Mark the current phase with an arrow. Use checkboxes:

```
## AEGIS Threat Intel — Setup Progress

### Phase 1: Local Setup  ← current
- [ ] Configure environment variables (.env)
- [ ] Run /audit-feed-source to confirm a real source is reachable
- [ ] Run /update-stack-profile at least once
- [ ] Run /scan-threats once and have Hamid review the real trial output
- [ ] Install recommended plugins

### Phase 2: Trinity Deployment
- [ ] Deploy to Trinity (as a free-pool agent)
- [ ] Confirm OmniRoute auth mode is actually live (with aegis-infra)
- [ ] Run a skill remotely

### Phase 3: Schedules
- [ ] Enable /scan-threats cadence (only after the trial run is approved)
- [ ] Verify first scheduled execution

**Progress: 0/10 complete**
```

### Step 3: Guide Next Step

Identify the first incomplete step in the current phase. Based on which step it is, provide specific guidance:

**For `env_configured`:**
- Check if `.env` exists. If not, guide: `cp .env.example .env` then fill in values.
- `NVD_API_KEY` is optional — the agent works without it, just more rate-limited.
- After user confirms, mark done.

**For `first_skill_run`:**
- Tell the user to run `/audit-feed-source` — this establishes whether a real feed is actually reachable before anything else proceeds.
- After it runs (even if it reports gaps — that's a valid, useful result), mark done.

**For `stack_profile_seeded`:**
- Tell the user to run `/update-stack-profile` at least once, confirming or adjusting the CLAUDE.md defaults for what counts as AEGIS's stack.
- After it runs, mark done.

**For `first_trial_run`:**
- Tell the user to run `/scan-threats` once. This step is done only once Hamid has actually seen and reviewed the real output — not merely once the skill ran.

**For `plugins_installed`:**
- Run the install commands for each plugin selected during setup:
  ```
  /plugin install agent-dev@abilityai
  /plugin install trinity@abilityai
  /plugin install utilities@abilityai
  ```
- Run each install command via Bash. Note successes and failures.
- After all attempted, mark done.

**For `onboarded` (Trinity phase):**
- Guide the user to run `/trinity:onboard`.
- After completion, mark done and advance phase.

**For `auth_mode_confirmed`:**
- This is NOT the same as deploying. Ask `aegis-infra` (or Hamid) to confirm the agent's Trinity auth mode is actually OmniRoute-routed (`gemini/gemini-3.7-flash`), not defaulting to Claude subscription auth. If it's still on subscription auth, flag that clearly rather than assuming the tier assignment took effect automatically.
- After confirmed, mark done.

**For `first_remote_run`:**
- Tell user to run `mcp__trinity__chat_with_agent` with the agent name and a skill, e.g. `/audit-feed-source`.
- After completion, mark done and advance phase.

**For `schedules_configured`:**
- Only proceed if `first_trial_run` was genuinely reviewed and approved. Tell user the recommended schedule lives in `template.yaml` (`schedules:`); deploying with `/trinity:onboard` reconciles it onto the instance. It ships `enabled: false` by design.
- To turn it on/off on the live agent, use `mcp__trinity__toggle_agent_schedule`.
- After completion, mark done.

**For `first_scheduled_run`:**
- Tell user to check `mcp__trinity__get_schedule_executions` for execution confirmation.
- After verified, mark done.

### Step 4: Update State

After each step is completed, update `onboarding.json`:
- Set the step's `done` to `true`
- If all steps in current phase are done, advance `phase` to the next phase
- If all phases complete, congratulate the user

### Step 5: Phase Transitions

When all steps in a phase are complete:

**Local → Trinity:**
```
## Local Setup Complete!

AEGIS Threat Intel is fully configured and working locally — a real trial scan has
been run and reviewed.

Run /onboarding again when you're ready to deploy to Trinity.
```

**Trinity → Schedules:**
```
## Trinity Deployment Complete!

AEGIS Threat Intel is live on Trinity, and its OmniRoute auth mode is confirmed.
Now let's decide whether to enable the /scan-threats cadence.

Run /onboarding to configure it.
```

**All Complete:**
```
## Onboarding Complete!

AEGIS Threat Intel is fully set up:
- ✓ Local environment configured, real trial scan reviewed
- ✓ Deployed to Trinity (free-pool, OmniRoute-routed)
- ✓ /scan-threats cadence running

You're all set. The onboarding.json file can be kept as a record or deleted.
```

## Outputs

- Updated `onboarding.json` with progress
- Step-by-step guidance for the current task
- Phase transition messages at milestones
