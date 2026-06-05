# Skill: Progress Digest

**Purpose:** Generate a plain-language, one-page progress summary for a feature intent — written for non-technical stakeholders (product owners, clients, leadership) who cannot read engineering artifacts. Translates the intent file, bolt status, and unit statuses into a clear picture of what is being built, what is done, and what comes next.

**Trigger:** Engineer-initiated at any point during or after delivery of an intent. Invoked by saying something like "generate a progress digest for [intent]" or "create a stakeholder update for [intent]."

**Never run automatically.** The engineer decides when to share a digest and with whom.

**Output:** A single markdown file at `ai-dlc/ops/inception/intents/[intent-slug]-digest.md`. If a digest already exists for this intent, it is overwritten — there is always exactly one current digest per intent.

---

## Step 1 — Identify the Intent

Ask the engineer:

> "Which intent should I generate a progress digest for?"

Read the following in full before proceeding:
- The intent file
- The bolt file(s) linked from the intent's "Bolts That Delivered This Intent" table or from the backlog
- The `ai-dlc/ops/build/backlog.md` — to get current unit statuses for units under this intent

Do not ask follow-up questions. All content needed is in these artifacts. If a unit status in the backlog contradicts the intent file, use the backlog as the source of truth for status.

---

## Step 2 — Translate Status to Plain Language

Before writing the digest, map all technical statuses and terms to plain language. Apply these translations throughout the digest — never use the technical terms in the output:

**Status translations:**

| Technical status | Plain language |
|---|---|
| Open | Not yet started |
| Planned | Scheduled |
| In Progress | Currently in progress |
| Done | Complete |
| Blocked | On hold |
| Deferred | Postponed |
| Cancelled | Removed from scope |

**Term translations:**

| Technical term | Plain language equivalent |
|---|---|
| Intent | Feature / initiative |
| Unit | Work item / task (or describe the functionality directly) |
| Bolt | Phase of work |
| Elaboration | Planning session |
| AC / Acceptance Criteria | Success condition |
| Backlog | Work queue |
| Refactor | Internal improvement |
| API | System interface (or omit entirely if not relevant to the stakeholder) |
| Entity / Data model | [describe what it represents, not what it is] |

**Rule:** If a piece of information has no plain-language equivalent that a non-technical stakeholder would understand, omit it from the digest entirely. The digest is not a complete technical record — it is a curated communication.

---

## Step 3 — Calculate Progress

From the backlog and bolt files, count units under this intent by status:

```
Total work items:     [N]
  Complete:           [N]
  Currently in progress: [N]
  Scheduled:          [N]
  On hold:            [N]
  Not yet started:    [N]
```

Derive an overall completion percentage: (Done units / total units) × 100, rounded to the nearest 10%.

If all units are Done and the intent status is Implemented, the overall status is **Delivered**. Otherwise:

| Condition | Overall status |
|---|---|
| No units started | Not yet started |
| At least one unit Done or In Progress | In progress |
| All units Done, intent not yet Implemented | Wrapping up |
| Intent status = Implemented | Delivered |
| Any unit Blocked | In progress — some items on hold |

---

## Step 4 — Write the Digest

Write the digest using only the sections below. Do not add sections, headers, or commentary beyond what is defined here. Do not include links to internal files — stakeholders cannot access them.

```markdown
# Progress Update: [Intent name in plain language]

**As of:** YYYY-MM-DD  
**Overall status:** [status from Step 3]  
**Progress:** [X]% complete

---

## What We Are Building

[2–3 sentences drawn from the intent's "What" and "Why" sections. Written from the user or business perspective — describe what the user will be able to do, not how the system works. Do not mention technology, architecture, or implementation details.]

## Why This Matters

[1–2 sentences from the intent's "Why" section. Focus on the business or user outcome. Omit if the answer would duplicate "What We Are Building".]

## Progress Summary

**Complete:** [N of N work items]  
**In progress:** [N items]  
**Remaining:** [N items]  
**On hold:** [N items — include brief plain-language reason for each if known]

## What Has Been Done

[Bullet list. Each bullet describes a completed work item in plain language — what capability or outcome was achieved, not what code was written. Use the unit name and AC descriptions as source material, translated into plain language. Omit this section if nothing is complete yet.]

- [Plain-language description of completed work item]
- [Plain-language description of completed work item]

## What Is Happening Now

[Bullet list. Each bullet describes an in-progress work item and what it will deliver when done. Omit this section if nothing is in progress.]

- [Plain-language description of in-progress work item and its outcome]

## What Comes Next

[Bullet list. Each bullet describes a scheduled or not-yet-started work item. Omit this section if there is nothing remaining.]

- [Plain-language description of upcoming work item]

## On Hold

[Bullet list with one line per blocked item: what is paused and why, in plain language. Omit this section entirely if nothing is blocked.]

- [Plain-language description of what is paused and why]

## What Success Looks Like

[1–2 sentences drawn directly from the intent's "Success Looks Like" section. This tells the stakeholder how they will know the work is done. Do not paraphrase into technical language — keep it in the user-outcome framing of the original.]

## Timeline

[List target dates from the bolt file(s). If no target dates are set, write "Timeline not yet confirmed." Do not estimate — only state what is recorded in the bolt files.]

- [Phase or milestone name]: target [date]
- Delivery: [date, or "to be confirmed"]
```

---

## Step 5 — Review Before Writing

Before creating the file, present the digest to the engineer:

> "Here is the draft progress digest for [intent name]. Review it for accuracy and tone before I save it — particularly: does it describe the work in terms your stakeholders will understand? Is anything sensitive omitted?"

Wait for the engineer to confirm, revise, or reject. If the engineer requests changes, apply them and present the revised draft. Do not write the file until confirmed.

---

## Step 6 — Write the File and Confirm

Write the confirmed digest to `ai-dlc/ops/inception/intents/[intent-slug]-digest.md`.

If a digest file already exists for this intent, overwrite it without warning — the previous version is superseded by the current state.

Confirm:

```
Progress digest written.

File: ai-dlc/ops/inception/intents/[intent-slug]-digest.md
Intent: [intent name]
Status: [overall status]
Progress: [X]% complete ([N] of [N] work items done)

The digest can be regenerated at any time to reflect the current state.
To share it, copy the file contents or export it directly.
```
