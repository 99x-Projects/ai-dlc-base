# Skill: Knowledge Promotion

**Purpose:** Evaluate each applied improvement to determine whether it is generic enough to benefit all AI-DLC projects, and if so, draft the exact change needed in the base repository so the engineer can raise a PR. Turns the passive "open a PR if you remember" instruction into a deliberate, structured ceremony that runs after every retro.

**Trigger:** Runs automatically as Step 5 of the Post-Retro Improvement Workflow, immediately after all approved improvements are applied. Can also be invoked directly by the engineer against a specific improvement file: "run knowledge promotion for [improvement]."

**Context:** This skill operates inside a project repository. It cannot modify the base repository directly. Its output is a drafted change — the exact file path and text to add or replace in the base repo — which the engineer carries into a PR against `ai-dlc-base`. The promotion decision and draft are recorded in the improvement file so the context is never lost.

---

## Step 1 — Identify the Improvements to Evaluate

If triggered by the Post-Retro Improvement Workflow, evaluate every improvement that was marked Applied in the current retro session, in order.

If invoked directly, ask the engineer:

> "Which improvement file should I evaluate for knowledge promotion?"

Read the improvement file in full before beginning the evaluation.

---

## Step 2 — Classify: Generic or Project-Specific

For each improvement, determine whether it is **generic** (beneficial to all AI-DLC projects) or **project-specific** (only relevant to this project's stack, domain, or conventions).

**Use the target file as the primary classification signal:**

| Target file type | Classification |
|---|---|
| Any file in `process-onboarding-agent/skills/` | Generic — skills are copied verbatim into every project |
| Any `_template.md` file in `process-onboarding-agent/ops/` | Generic — templates are shared across all projects |
| `process-onboarding-agent/rules/engagement.md` | Generic — copied verbatim into every project |
| `process-onboarding-agent/setup-guide.md` | Generic — the shared framework specification |
| `process-onboarding-agent/onboard.md` | Generic — the shared onboarding protocol |
| `process-onboarding-agent/rules/prompt-quality-gate.md` | Likely generic — evaluate content |
| `process-onboarding-agent/rules/code-standards.md` | Project-specific — encodes the project's tech stack |
| `process-onboarding-agent/rules/security.md` | Project-specific — unless the finding addresses a universal pattern |
| `process-onboarding-agent/rules/architecture.md` | Project-specific — encodes project ADRs |
| `process-onboarding-agent/guidelines/domain-glossary.md` | Project-specific |
| `process-onboarding-agent/guidelines/edge-cases.md` | Project-specific |
| `process-onboarding-agent/guidelines/forbidden-zones.md` | Project-specific |
| `process-onboarding-agent/guidelines/entry-points.md` | Project-specific |
| `process-onboarding-agent/guidelines/acceptance-patterns.md` | Likely generic — evaluate content |
| `CLAUDE.md` / `.cursorrules` / `copilot-instructions.md` | Project-specific |
| New file being created | Evaluate by content |

**For ambiguous cases, apply the content test:**

Ask: *If this improvement were applied to a completely different project — different tech stack, different domain, different team size — would it still be an improvement?*

- Yes, the process or ceremony would be better regardless of context → **Generic**
- No, it relies on knowledge of this project's domain, stack, or specific patterns → **Project-specific**

---

## Step 3 — Handle Project-Specific Improvements

If the improvement is project-specific:

Update the `Knowledge Promotion` field in the improvement file to:

```
**Knowledge promotion:** Project-specific — [one-sentence reason why it does not generalise]
```

State the outcome to the engineer:

> "[Improvement title] is project-specific ([reason]). No base repo change needed."

Proceed to the next improvement.

---

## Step 4 — Handle Generic Improvements

If the improvement is generic, determine the corresponding file in the base repository:

| Project file changed | Base repo file to change |
|---|---|
| `process-onboarding-agent/skills/[skill].md` | `process-onboarding-agent/skills/[skill].md` |
| `process-onboarding-agent/ops/[path]/_template.md` | `process-onboarding-agent/ops/[path]/_template.md` |
| `process-onboarding-agent/rules/engagement.md` | `process-onboarding-agent/rules/engagement.md` |
| `process-onboarding-agent/rules/prompt-quality-gate.md` | `process-onboarding-agent/rules/prompt-quality-gate.md` |
| `process-onboarding-agent/setup-guide.md` | `process-onboarding-agent/setup-guide.md` |
| `process-onboarding-agent/onboard.md` | `process-onboarding-agent/onboard.md` |
| New skill or guideline file | New file at the same relative path |

Draft the exact change needed in the base repo. Be precise — draft the exact text to replace and the exact replacement, the same format used in the improvement file itself:

```
Base repo change draft
──────────────────────────────────────────────
File:    [relative path in base repo]

Current text (or "New addition"):
[exact text that exists in the base repo file, or "N/A — new addition"]

Proposed replacement:
[exact text to replace it with]

Why this improves all AI-DLC projects:
[one or two sentences — the reason this generalises beyond this project]
──────────────────────────────────────────────
```

Present the draft to the engineer and ask:

> "This improvement appears to apply to all AI-DLC projects. I've drafted the change for the base repository above. Would you like to promote it?
> - **Yes** — I'll record the draft in the improvement file. Raise a PR against `ai-dlc-base` with this content, noting this project and retro as the source.
> - **Revise** — Adjust the draft before recording.
> - **No** — Record as declined with a reason."

Wait for the engineer's response.

---

## Step 5 — Record the Decision

**If promoted:**

Update the improvement file's `Knowledge Promotion` field:

```
**Knowledge promotion:** Promoted — PR to be raised against `ai-dlc-base`

### Base Repo Change Draft

**File:** [relative path in base repo]

**Current text:**
[exact current text or "N/A — new addition"]

**Proposed replacement:**
[exact proposed text]

**Why this generalises:**
[reason]

**Source:** [project name], retro [retro file link]
```

Remind the engineer:

> "The draft is recorded in the improvement file. When you raise the PR against `ai-dlc-base`, copy the draft above into the PR body and link back to this improvement file as the source."

**If declined:**

Update the improvement file's `Knowledge Promotion` field:

```
**Knowledge promotion:** Declined — [reason the engineer gave]
```

---

## Step 6 — Summary

After evaluating all improvements in the batch, present a summary:

```
Knowledge Promotion complete.

Improvements evaluated: [N]
  Generic — promoted:        [N]  [list titles]
  Generic — declined:        [N]  [list titles with reason]
  Project-specific:          [N]  [list titles]

[If any promoted:]
Next step: raise a PR against ai-dlc-base for each promoted improvement.
Include the draft text from the improvement file and link back to this project's retro as the source.
```
