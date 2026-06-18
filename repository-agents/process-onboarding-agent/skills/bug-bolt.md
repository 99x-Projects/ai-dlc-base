# Skill: Bug Bolt

**Purpose:** A lightweight bolt workflow for fixing a specific, reproducible bug. Replaces the full mob elaboration ceremony with a focused intake, recurrence check, and a single unit — while preserving all quality gates (prompt gate, pre-generation checks, review checklist) and the post-fix retro.

**Trigger:** Engineer says "fix a bug in X", "there's a bug in X", or "bug: [description]". Routed from Section 6 of the master rule file. Do not run a full mob elaboration when this trigger is detected — load this skill and follow its steps instead.

**Requires:** A fully onboarded AI-DLC project. The master rule file, quality gate (`rules/prompt-quality-gate.md`), review checklist (`skills/review-checklist.md`), unit and bolt templates, and the `ops/` folder structure must all be in place. This skill is a bolt-type variant that operates within the framework — it is not a standalone bug-tracking tool. If AI-DLC is not yet installed, start with `process-onboarding-agent/onboard.md` first.

**What this skill does not replace:** intent files, mob elaboration (for new features), UAT, the retro, or the Post-Retro Improvement Workflow. A bug bolt is strictly scoped to fixing a known defect — use the standard feature flow for any work that adds or changes behavior.

---

## Step 1 — Bug Intake

Ask the engineer four questions in one message:

> "To scope this fix accurately, please tell me:
> 1. **What is the bug?** Describe the symptom — what the user or system experiences.
> 2. **Where does it occur?** Name the component, file, function, or user flow.
> 3. **How is it reproduced?** List the steps or conditions that trigger it.
> 4. **What is the expected behavior?** What should happen instead?"

Record all four answers before proceeding.

---

## Step 2 — Recurrence Check

Search the project's retro files and incident files for similar descriptions.

- If a similar pattern appears in more than one retro or incident: note it as **recurring**. Root Cause Analysis is mandatory after the fix.
- If no similar pattern is found: proceed without the RCA flag.

State the recurrence finding to the engineer before continuing.

---

## Step 3 — Scope the Fix

Identify the minimum set of changes needed to fix the bug without side effects. Ask if unsure:

> "Is this fix isolated to [component], or are there other places where the same logic runs?"

If multiple components are affected, each becomes a separate unit.

---

## Step 4 — Create the Unit

Create a single unit file using the standard unit template with these constraints:

**Context:** What is broken and why — a precise technical statement, not a vague description.

**Acceptance Criteria:**
```
Given [the conditions that trigger the bug]
When [the action that exposes it]
Then [the system behaves correctly — the expected behavior from Step 1]

Given [the same triggering conditions]
When [the fix is NOT present — regression guard]
Then [the broken behavior is detectable — confirm a test would catch a regression]
```

At least one unhappy-path AC must be included.

**Scope:**
- In scope: the specific function, query, component, or flow causing the bug
- Out of scope: refactoring unrelated code, adding features, "while we're here" changes

**Pre-generation checks:** Grep for the affected function or component before generating. Confirm no duplicate fix already exists.

**Observability:** Define a log entry or metric that confirms the bug no longer occurs in production.

---

## Step 5 — Risk Assessment

Before execution, ask one question:

> "Does this fix touch any shared component, database schema, or API contract used by other intents or services?"

If yes: run `process-onboarding-agent/skills/bolt-risk-assessment.md` and record the blast radius in the bolt file.
If no: record "Blast radius: isolated to [component]. No shared interfaces affected."

---

## Step 6 — Execute and Verify

Execute the unit. Review output using `process-onboarding-agent/skills/review-checklist.md`.

After output is accepted, confirm the fix:

> "Can you verify the bug is resolved against the reproduction steps from Step 1?"

Do not close the unit until the engineer confirms the fix is verified.

---

## Step 7 — Close

1. Mark unit Done. Mark bolt Done. Update the backlog.
2. Create a retro file and run the Post-Retro Improvement Workflow.
3. If the bug was flagged as **recurring** in Step 2: read `process-onboarding-agent/skills/root-cause-analysis.md` and run it now. Do not skip.
4. If the bug caused a production impact: create or update an incident file at `ops/operate/incidents/`.
