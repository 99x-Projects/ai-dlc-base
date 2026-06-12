# Skill: Hotfix Bolt

**Purpose:** An emergency bolt workflow for production issues that cannot wait for a full planning cycle. Prioritises speed over ceremony — elaboration is skipped entirely. A mandatory retro and RCA follow within 24 hours. Speed does not suspend quality gates: the review checklist and pre-generation checks still apply.

**Trigger:** Engineer says "hotfix", "production issue", "prod is down", "emergency fix for X", or "P1 in production". Routed from Section 6 of the master rule file. Load this skill immediately — do not begin elaboration or ask for intent files.

**Requires:** A fully onboarded AI-DLC project. The master rule file, quality gate (`rules/prompt-quality-gate.md`), review checklist (`skills/review-checklist.md`), incident template (`ops/operate/incidents/_template.md`), and retro template must all be in place. This skill is a bolt-type variant that operates within the framework — it is not a standalone incident-response runbook. If AI-DLC is not yet installed, start with `ai-dlc/onboard.md` first.

**What this skill does not replace:** the retro (mandatory within 24 hours), RCA (`skills/root-cause-analysis.md`), incident file, or any quality gate step. A hotfix bolt compresses ceremony — it does not eliminate accountability.

---

## Step 1 — Hotfix Intake

Ask the engineer three questions immediately:

> "To move fast safely, I need three things:
> 1. **What is broken in production?** One sentence — symptom and user impact.
> 2. **How severe is it?** P1 (full outage or data loss risk) / P2 (degraded service, workaround exists) / P3 (minor defect, low user impact).
> 3. **Is a rollback available right now?** Yes / No / Partial."

Do not proceed until all three are answered.

If no rollback is available, state this explicitly before continuing:

> "No rollback is available. This fix is the only recovery path — proceed with extra care."

---

## Step 2 — Confirm the Blast Radius

Before writing any code, ask:

> "What is the blast radius of the proposed fix? Which components, services, or users are affected if the fix has an unintended side effect?"

Record the answer. If the blast radius is wide (multiple services, database schema change, shared API contract): slow down and consider whether a partial mitigation (feature flag, rate limit, traffic shift) is safer than a direct fix. State the trade-off to the engineer and wait for their decision.

---

## Step 3 — Create the Minimal Unit

Create a single unit with the minimum viable AC set:

**Context:** The production symptom, the root hypothesis, and the proposed change — three sentences maximum.

**Acceptance Criteria:**
```
Given [the production state that caused the issue]
When [the fix is applied]
Then [the system returns to the expected behavior]

Given [normal production traffic after fix]
When [the fix is deployed]
Then [no new errors or regressions appear in monitoring]
```

**Scope:**
- In scope: the specific change needed to restore service
- Out of scope: any improvement, refactoring, or "while we're here" addition — log these for the next bolt, do not act on them now

**Pre-generation checks:** Grep for the affected component. Confirm the fix does not touch unrelated code paths.

---

## Step 4 — Execute

Execute the unit. Review output using `ai-dlc/skills/review-checklist.md`.

Speed does not suspend quality gates. If output fails review: do not retry blindly — state the failure and ask the engineer how to proceed.

---

## Step 5 — Verify Before Production

If a staging environment is available:

> "Before deploying to production, please verify the fix in staging against the reproduction steps."

If no staging environment exists: note this explicitly and ask the engineer to confirm acceptance of the risk before deploying.

---

## Step 6 — Create the Incident File

If an incident file does not already exist for this event, create one now at `ops/operate/incidents/YYYY-MM-DD-<unix_timestamp>-[slug].md`. Populate at minimum:

- Detection time and how it was detected
- Production impact: who was affected and for how long
- Root hypothesis: what caused it (even if unconfirmed)
- Rollback availability (from Step 1)
- Fix deployed timestamp

---

## Step 7 — Close and Schedule the Retro

1. Mark unit Done. Mark bolt Done. Update the backlog.
2. State clearly:

> "The hotfix is complete. A retro is mandatory within 24 hours — please schedule it now. The incident file is at [path]. Run `ai-dlc/skills/root-cause-analysis.md` during or immediately after the retro."

3. Do not run the retro now if the production event is still being monitored. Close this session and return when the system is stable.
