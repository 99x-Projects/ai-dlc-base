# Contributing Improvements Back to the Framework

← [Back to README](../README.md)

---

## Overview

Every project running the 99x Intent Delivery Framework produces improvements through its retros. Most of those improvements are specific to that project's stack or domain and stay local. Some, however, would make the framework better for **every** project — a sharper quality gate, a clearer skill, a better template. Those are the ones that get promoted back here, to `ai-dlc-base`.

This is how the framework compounds: one team's hard-won fix becomes every team's default.

The promotion path has two halves:

1. **Classification and drafting** — happens automatically inside the project repo, via the `knowledge-promotion.md` skill.
2. **Raising the PR** — a manual step the engineer does against this repo, `ai-dlc-base`.

---

## Step 1 — Classification Runs Automatically After Every Retro

The `process-onboarding-agent/skills/knowledge-promotion.md` skill runs as the final step of the Post-Retro Improvement Workflow, immediately after a retro's approved improvements are applied. It is mandatory — no improvement is closed without a classification. It can also be invoked directly against a single improvement: *"run knowledge promotion for [improvement]."*

For each improvement, the skill classifies it as:

| Classification | Meaning | Outcome |
|---|---|---|
| **Generic** | Would still be an improvement on a completely different project — different stack, domain, team size | Drafted for promotion to `ai-dlc-base` |
| **Project-specific** | Relies on this project's domain, stack, or conventions | Stays local; no base repo change |

**The changed file is the primary signal:**

| Target file type | Default classification |
|---|---|
| `process-onboarding-agent/skills/*.md` | Generic — skills are copied verbatim into every project |
| `process-onboarding-agent/ops/**/_template.md` | Generic — templates are shared across all projects |
| `process-onboarding-agent/rules/engagement.md` | Generic — copied verbatim into every project |
| `process-onboarding-agent/setup-guide.md` | Generic — the shared framework specification |
| `process-onboarding-agent/onboard.md` | Generic — the shared onboarding protocol |
| `process-onboarding-agent/rules/prompt-quality-gate.md` | Likely generic — evaluated by content |
| `process-onboarding-agent/guidelines/acceptance-patterns.md` | Likely generic — evaluated by content |
| `process-onboarding-agent/rules/code-standards.md` | Project-specific — encodes the project's tech stack |
| `process-onboarding-agent/rules/security.md` | Project-specific — unless it addresses a universal pattern |
| `process-onboarding-agent/rules/architecture.md` | Project-specific — encodes project ADRs |
| `process-onboarding-agent/guidelines/domain-glossary.md` | Project-specific |
| `process-onboarding-agent/guidelines/edge-cases.md` | Project-specific |
| `process-onboarding-agent/guidelines/forbidden-zones.md` | Project-specific |
| `process-onboarding-agent/guidelines/entry-points.md` | Project-specific |
| `CLAUDE.md` / `.cursor/rules/project-rules.mdc` (or any `.mdc` under `.cursor/rules/`) / `.cursorrules` (legacy) / `copilot-instructions.md` | Project-specific |

For anything ambiguous, the skill applies one test: *if this improvement were applied to a totally different project, would it still be an improvement?* Yes → generic.

---

## Step 2 — Generic Improvements Get a Drafted Change

If an improvement is generic, the skill maps the project file to its counterpart in `ai-dlc-base` and drafts the exact change:

```
Base repo change draft
──────────────────────────────────────────────
File:    [relative path in ai-dlc-base]

Current text (or "New addition"):
[exact text that exists in the base repo file, or "N/A — new addition"]

Proposed replacement:
[exact text to replace it with]

Why this improves all AI-DLC projects:
[one or two sentences]
──────────────────────────────────────────────
```

You're asked to approve it — **Yes** (record and promote), **Revise** (adjust the draft), or **No** (decline with a reason). Once you confirm, the decision and full draft are written into the improvement file's `Knowledge Promotion` field, so the change is never just in someone's memory — it's traceable back to the project and retro that produced it.

---

## Step 3 — Raise the PR Against `ai-dlc-base`

The skill drafts the change; it never touches this repository directly. You carry the draft into a PR here:

1. Open a PR against `ai-dlc-base`.
2. Apply the change to the file named in the draft, under `process-onboarding-agent/` — copy the **Proposed replacement** text exactly, don't paraphrase it.
3. In the PR description, include:
   - The **source project and retro** (link to the improvement file if the project repo is accessible).
   - The **"why this generalises"** reasoning from the draft.
   - Before/after text if the diff isn't self-explanatory (e.g. a template or rule change).
4. Request review as normal — promoted changes go through the same review bar as any other PR to this repo.

### Example PR description

```
## Summary
Promote a knowledge-promotion improvement from [Project X], retro 2026-06-30.

Tightens the wording in `prompt-quality-gate.md` so vague acceptance criteria
are rejected before elaboration starts, not caught later in review.

## Source
[Project X] — ops/operate/retros/2026-06-30-retro.md, improvement #3

## Why this generalises
The gap (ACs passing quality gate without a measurable outcome) isn't specific
to Project X's domain — any project's elaboration step benefits from catching
this earlier.
```

---

## What Doesn't Get Promoted

Declined improvements are recorded too — the `Knowledge Promotion` field is set to `Declined — [reason]` rather than silently dropped. This keeps the project's improvement history complete even when nothing comes back here. Common decline reasons:

- The fix only makes sense given this project's stack or domain.
- The change is a workaround for a project-specific constraint, not a gap in the process itself.
- A similar generic improvement was already promoted from another project.

---

## Summary

```
Retro → Improvements applied → Knowledge Promotion skill runs automatically
      → Generic?  → draft change → engineer approves → recorded in improvement file
                                                        → engineer raises PR against ai-dlc-base
      → Project-specific? → recorded, stays local
```

The mandatory step is the classification after every retro — that's what prevents "we should really upstream this" from being forgotten. The manual step is only the PR itself.
