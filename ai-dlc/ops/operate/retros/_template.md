# Retrospective: [Bolt name]

**Bolt:** [ops/build/bolts/YYYY-MM-DD-[bolt-slug].md](../../build/bolts/YYYY-MM-DD-[bolt-slug].md)
**Date:** YYYY-MM-DD
**Participants:** [Engineer names]

---

## What Went Well

- [Observation]
- [Observation]

---

## What Didn't Go Well

- [Observation]
- [Observation]

---

## AI-Specific Observations

**Prompts that worked well:**
- [Description — what made them effective]

**Prompts that needed revision:**
- [Description — what was wrong and how it was fixed]

**Quality gate failures:**
- [Description — which component was missing and what was asked]

**Output accepted without sufficient review:**
- [Description — what slipped through and what the consequence was]

---

## Actions

| # | Action | Owner | Target date | Status |
|---|---|---|---|---|
| 1 | [description] | [name] | YYYY-MM-DD | Open |

---

## Improvements Triggered

**This section cannot be left blank.** If no improvements were triggered, state the reason explicitly.

| # | Improvement file | Target rule/guideline | Status |
|---|---|---|---|
| 1 | [ops/operate/improvements/YYYY-MM-DD-[slug].md](../improvements/YYYY-MM-DD-[slug].md) | [ai-dlc/rules/...] | Open |

---

## New Intents Triggered

| Intent | Reason |
|---|---|
| [name] | [what this retro revealed that requires a new capability] |

---

## Post-Retro Improvement Workflow

> **The AI must execute this workflow immediately after the retro document is complete. Do not wait for the engineer to initiate it.**

### Step 1 — Synthesise findings into improvement proposals

Read every entry in "What Didn't Go Well", "AI-Specific Observations", and "Actions". For each finding that points to a fixable rule, skill, or guideline, draft one improvement proposal covering:
- Which file needs to change (`ai-dlc/rules/`, `ai-dlc/skills/`, `ai-dlc/guidelines/`, or the master rule file)
- The exact current text to replace (or "new addition" if nothing exists yet)
- The exact proposed replacement text
- Why the change prevents the finding from recurring

Present all proposals to the engineer as a numbered list — one proposal per finding — before creating any files.

### Step 2 — Engineer review

For each proposal, the engineer approves, rejects, or asks for revision. Record the decision next to each item. Do not proceed to Step 3 until all proposals have a decision.

### Step 3 — Create improvement files and apply changes

For each approved proposal:
1. Create an improvement file at `ops/operate/improvements/YYYY-MM-DD-[slug].md` using the template
2. Apply the change directly to the target file
3. If the master rule file was changed, update all mirror files in the same pass (see Step 8 of the setup guide)
4. Set the improvement status to **Applied** and record the date

### Step 4 — Update this retro

Update the "Improvements Triggered" table above with the improvement file links and mark each as Applied or Rejected. Close the retro only after all approved improvements are applied.

### Step 5 — Knowledge Promotion Evaluation

After all improvements are applied, read `ai-dlc/skills/knowledge-promotion.md` and run it against every improvement marked Applied in this retro. This step evaluates whether each improvement is generic enough to benefit all AI-DLC projects and should be contributed back to the base repository.

**This step is mandatory and must not be skipped.** A retro is not fully closed until every Applied improvement has a `Knowledge Promotion` status recorded in its improvement file — even if that status is "Project-specific."
