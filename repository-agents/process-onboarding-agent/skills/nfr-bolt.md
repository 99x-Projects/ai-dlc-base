# Skill: NFR Bolt

**Purpose:** A bolt workflow for non-functional quality attribute improvements — performance, security hardening, accessibility, reliability, observability, or scalability. Operates on existing intents rather than creating a new one. Every AC must carry a measurable threshold; vague targets ("faster", "more secure") are rejected.

**Trigger:** Engineer says "improve performance of X", "harden security for X", "accessibility improvements", "NFR bolt for X", "non-functional work on X", or "reliability improvements to X". Routed from Section 6 of the master rule file. Do not create a new intent file — load this skill and reference the existing affected intents instead.

**Requires:** A fully onboarded AI-DLC project with at least one delivered intent. The master rule file, quality gate (`rules/prompt-quality-gate.md`), review checklist (`skills/review-checklist.md`), dependency map (`ops/inception/dependency-map.md`), unit and bolt templates, and the `ops/` folder structure must all be in place. This skill is a bolt-type variant that operates within the framework — it is not a standalone performance or security checklist. If AI-DLC is not yet installed, start with `process-onboarding-agent/onboard.md` first.

**What this skill does not replace:** intent files for existing features (it references them), mob elaboration (for new features), UAT, or the standard retro. An NFR bolt improves how existing features behave — it does not add new behavior. Any NFR change that requires new behavior visible to users must be captured as a new intent using the standard feature flow.

NFR categories: Performance · Security Hardening · Accessibility · Reliability / Resilience · Observability · Scalability

---

## Step 1 — NFR Intake

Ask the engineer four questions:

> "To scope this NFR bolt accurately:
> 1. **Which quality attribute?** Performance / Security Hardening / Accessibility / Reliability / Observability / Scalability / Other — specify.
> 2. **Which component or feature?** Name the service, module, or user flow being improved.
> 3. **What is the current state?** A measurement or description of today's behavior — e.g. p95 latency is 1.2 s; screen reader navigation is broken on checkout; no retry logic on payment calls.
> 4. **What is the target threshold?** A specific, measurable outcome — e.g. p95 latency < 300 ms; WCAG 2.1 AA compliant; three retries with exponential backoff on payment calls."

Do not proceed without a measurable target threshold. "Better", "faster", and "more secure" are not valid thresholds — ask again until a number or verifiable criterion is provided.

---

## Step 2 — Identify Affected Intents

Read `process-onboarding-agent/ops/inception/dependency-map.md` and the backlog. Identify which existing intents or units touch the component being improved.

State the affected intents to the engineer:

> "This NFR bolt affects the following existing intents: [list]. I will cross-reference this bolt in each intent's Implementation Summary after completion."

If the NFR improvement changes behavior visible to users (e.g. a new error retry message, a changed loading state): flag those intents' ACs for review after the bolt is done.

---

## Step 3 — Establish the Baseline

Before generating any unit, confirm the current measured value:

> "What is the current measured value for [the threshold from Step 1]? This becomes the before-state for verification."

If no measurement exists: the first unit in this bolt must establish the measurement tooling or instrumentation before any improvement work begins. Record the baseline once measured.

---

## Step 4 — Create the Units

NFR bolts may contain multiple units if the improvement spans more than one component. Apply the standard unit template with these constraints:

**ACs must use measurable thresholds:**
```
Given [the component under load / in its operational context]
When [the NFR scenario — e.g. 100 concurrent users, a screen reader navigating checkout]
Then [the measurable threshold is met — e.g. p95 response time < 300 ms with no errors]
```

Vague ACs such as "the page loads faster" are not accepted.

**Observability is mandatory on every NFR unit:**
- Metric or signal that confirms the threshold is met in production
- Alert threshold that would indicate regression back below the target

**No feature scope creep:** NFR units must not change business logic. If a performance fix requires a data model change that alters behavior, that is a new feature intent — not part of this NFR bolt. Flag it and defer.

---

## Step 5 — Risk Assessment

Run `process-onboarding-agent/skills/bolt-risk-assessment.md` before any unit executes.

NFR bolts often have wider blast radii than they appear: performance changes can alter caching behavior, security hardening can break existing integrations, accessibility changes can shift layout for sighted users. Do not skip this step.

---

## Step 6 — Execute

Execute units in the planned order. Review each unit's output using `process-onboarding-agent/skills/review-checklist.md` before proceeding to the next.

---

## Step 7 — Verify the Threshold

After all units are Done, verify against the baseline established in Step 3:

> "Please measure [the metric from Step 1] and compare it against the baseline: [value from Step 3]."

The bolt is not Done until the threshold is verified. If the threshold is not met: set bolt status to Blocked, state the gap (measured vs. target), and propose the next step. Do not silently mark it Done.

---

## Step 8 — Update Affected Intent Records

For each intent identified in Step 2:
- Add a note in its Implementation Summary referencing this NFR bolt and the improvement delivered
- If any of the intent's ACs are now outdated: flag them for revision in the next elaboration session for that intent

---

## Step 9 — Close

1. Mark all units Done. Mark bolt Done. Update the backlog.
2. Record the before/after measurement explicitly in the retro file.
3. Create the retro and run the Post-Retro Improvement Workflow.
