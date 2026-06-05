# AI-DLC Onboarding Prompt

Copy this file into the root of your target repository. Then say the following to your AI assistant:

---

> **Read `ai-dlc/onboard.md` and follow the instructions inside it.**

---

## Instructions for the AI Agent

You are about to onboard this repository to the AI-DLC framework. Read and follow `ai-dlc/setup-guide.md` in full. The steps below tell you exactly how to execute it.

### Step 0 — Orient yourself

1. Read `ai-dlc/setup-guide.md` from top to bottom before taking any action.
2. Check whether AI-DLC has already been set up in this repo by looking for files that are **only created during onboarding** — not files that ship with the base repo. The following are reliable indicators of an existing setup:
   - A master rule file at the repo root: `CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`
   - Any of these generated rules files: `ai-dlc/rules/prompt-quality-gate.md`, `ai-dlc/rules/code-standards.md`, `ai-dlc/rules/security.md`, `ai-dlc/rules/architecture.md`
   - Any of these generated skills files: `ai-dlc/skills/mob-elab-prompts.md`, `ai-dlc/skills/review-checklist.md`, `ai-dlc/skills/unit-template.md`
   - Any file inside `ai-dlc/guidelines/`
   - `ai-dlc/Instructions2FDE.md`

   **Do not treat the following as evidence of existing setup** — they are part of the base repo and are present in every fresh copy:
   - `ai-dlc/ops/` and all files inside it (including `ai-dlc/ops/inception/dependency-map.md`)
   - `ai-dlc/rules/engagement.md`
   - `ai-dlc/skills/compact-docs.md`
   - `ai-dlc/skills/root-cause-analysis.md`
   - `ai-dlc/skills/design-session.md`
   - `ai-dlc/skills/bolt-risk-assessment.md`
   - `ai-dlc/skills/progress-digest.md`
   - `ai-dlc/skills/uat.md`
   - `ai-dlc/skills/process-health.md`
   - `ai-dlc/skills/dependency-audit.md`
   - `ai-dlc/skills/knowledge-promotion.md`
   - `ai-dlc/skills/new-engineer-induction.md`
   - `ai-dlc/setup-guide.md`
   - `ai-dlc/onboard.md`

   If any indicator from the first list exists, report what you found and ask the engineer whether to extend the existing setup or start fresh.
3. Do not create or modify any files until Step 0 is complete.

### Step 1 — Identify the AI tool

Ask the engineer:

> "Which AI tool will be the primary tool for this project — Claude Code, Cursor, or GitHub Copilot? (You can add mirror files for additional tools later.)"

Record the answer. Use it to determine the master rule file name and path (see the table in `ai-dlc/setup-guide.md` — Before You Begin).

### Step 2 — Identify the project type

Ask the engineer:

> "Is this a fresh project with no significant codebase yet, or a mature project with existing code and team conventions?"

- **Fresh** → go to Step 3-F below.
- **Mature** → go to Step 3-M below.

---

## Path F — Fresh Project

### Step 3-F — Run the structured interview

Run the nine-question interview defined in `ai-dlc/setup-guide.md` under **Fresh Project — Structured Interview**. Ask one question at a time. Wait for each answer before continuing. Do not skip questions.

### Step 4-F — Create all files

Once all nine questions are answered, execute Steps 1–9 of the setup guide in order:
1. Create the full `ai-dlc/` folder structure with all template files.
2. Write the master rule file (all 9 sections) using interview answers for Sections 1–5 and Section 9.
3. Write all `rules/` files using interview answers and guide defaults.
4. Write all `skills/` files from the templates in this base repo.
5. Write all `guidelines/` files (domain glossary pre-populated from interview question 6; others as stubs).
6. Write all ops template files.
7. Write `Instructions2FDE.md`.
8. Write `ai-dlc/README.md`.
9. Create the backlog and the first intent from interview question 8.
10. Create mirror files for any additional tools the engineer wants to support.

### Step 5-F — Deliver the completion report

Present the completion report defined in `ai-dlc/setup-guide.md` under **Onboarding Completion**.

---

## Path M — Mature Project

### Step 3-M — Codebase archaeology (Phase M1)

Execute Phase M1 from the setup guide. Start with **M1.0 — Scope Agreement** to define the analysis segments, then offer the engineer the **parallel archaeology option (M1.0-P)**:

> Multiple engineers can each analyze a segment on their own machine simultaneously. Independent findings that match across sessions have higher confidence than single-session findings, and engineers familiar with a module will surface risks a cold analysis misses. Ask the engineer if they want to run segments in parallel — if yes, generate a Segment Assignment Brief for each segment and pause until all reports are returned.

If running in **single session**: proceed through M1.1 → M1.2 → M1.3 → M1.4 one segment at a time.

If running in **parallel mode**: pause after generating briefs. Resume at **M1.5 — Parallel Synthesis** once all Segment Reports are received, then continue to Phase M2.

Phases to execute:
1. M1.0 — Scope Agreement (define segments)
2. M1.0-P — Offer parallel mode (mandatory offer, engineer decides)
3. M1.1 — Architecture Mapping (per segment)
4. M1.2 — Pattern Extraction (per segment)
5. M1.3 — Due Diligence Audit (per segment)
6. M1.4 — Debt and Gap Mapping (per segment)
7. M1.5 — Parallel Synthesis (only if parallel mode was chosen)

### Step 4-M — Repository overlay (Phase M2)

1. Write the master rule file populated from archaeology output (Sections 1–5 from what you found; Sections 6–8 from guide defaults).
2. Create `ai-dlc/guidelines/forbidden-zones.md` — ask the engineer: *"Which files, modules, or patterns must the AI never modify without senior engineer approval?"* Record their answer as the initial forbidden zones list.
3. Create `ai-dlc/guidelines/entry-points.md` — ask the engineer: *"Which modules or features should AI-DLC Bolts start with?"* Record their answer as the initial entry points list.
4. Create `ai-dlc/rules/code-standards.md` from extracted patterns.
5. Create the full `ai-dlc/` folder structure with all remaining files and templates.

### Step 5-M — Blast radius controls (Phase M3)

1. Report current test coverage for the first entry-point module (run tests if possible, or report that coverage cannot be determined and flag it as a required manual check).
2. Ask the engineer: *"What feature flag mechanism does this project use, or should we establish one?"* Record the answer in the master rule file Code Rules section.
3. Confirm the default AC *"All integration tests for [affected module] pass without modification"* has been added to the master rule file Section 6 and to `ai-dlc/rules/code-standards.md`.

### Step 6-M — Deliver the completion report

Present the completion report defined in `ai-dlc/setup-guide.md` under **Onboarding Completion**.
