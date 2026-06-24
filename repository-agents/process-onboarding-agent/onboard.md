# AI-DLC Onboarding Prompt

## Prerequisites — copy these folders from the AI-DLC base repo before starting

Copy both of the following from `repository-agents/` in the base repo into your **project root**:

```
process-onboarding-agent/       ← this agent (the trigger and setup guide)
process-estimation-agent/       ← delivery estimation for release planning (keep after onboarding)
```

## Trigger

Once both folders are in your project root, say the following to your AI assistant:

---

> **"Read `process-onboarding-agent/onboard.md` and follow the instructions inside it."**

---

## Instructions for the AI Agent

You are about to onboard this repository to the AI-DLC framework. Read and follow `process-onboarding-agent/setup-guide.md` in full. The steps below tell you exactly how to execute it.

### Session Brief — Present this to the engineer before asking anything

Before asking any questions or taking any action, present the following overview to the engineer:

> **AI-DLC Process Onboarding — Session Overview**
>
> This session installs the AI-DLC framework into your repository. By the end of it, your project will have a fully configured governance layer that governs how you and your AI work together on every feature from this point forward.
>
> **A note before we begin:** At any point during this session, if you have a question about a step, need further clarification, or don't have the exact answer to a question I'm asking — just say so. I'll help you work through it so we don't get blocked.
>
> **What will happen in this session:**
> 1. I'll ask where your process documentation lives so I know where to install the framework.
> 2. I'll check whether AI-DLC is already set up in this repo.
> 3. I'll ask which AI coding tool you're using (Claude Code, Cursor, or GitHub Copilot).
> 4. I'll ask whether this is a fresh project or a mature one with existing code — then run one of two paths:
>    - **Fresh project:** A structured nine-question interview to capture your product identity, technology stack, domain language, and constraints — then I generate all framework files from your answers.
>    - **Mature project:** A phased codebase archaeology (architecture mapping, pattern extraction, due diligence audit, debt classification) before generating a framework that inherits your existing conventions rather than overwriting them.
>
> **Expected outputs from this session:**
> - A **master rule file** (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`) at your repo root — loaded automatically by your AI tool at the start of every future session
> - A complete **`intent-execution-framework/`** folder installed at your chosen process documentation path, containing:
>   - Rules files (prompt quality gate, code standards, security, architecture, engagement)
>   - Skills files (elaboration prompts, review checklist, UAT, bolt risk assessment, and more)
>   - Guidelines files (domain glossary, edge cases, acceptance patterns, dev setup)
>   - Ops templates (intents, units, bolts, retros, incidents, improvements)
> - A **completion report** listing every file created and flagging anything that needs your review before the first feature bolt runs
> - **`process-estimation-agent/`** already in your project root and ready to use — once mob elaboration produces units, invoke it in Mode 2 to get bolt-level estimates and a release milestone map
>
> **When this session ends:** Close it and open a brand new session. Your AI tool will automatically load the master rule file — that is the experience agent, ready to run your first feature.
>
> Ready to start? I have one question before we begin.

---

### Preliminary Step — Locate the Framework Home

Before doing anything else, ask the engineer:

> "Where do you keep process and workflow documentation in this repository? If you have an existing folder (for example `docs/`, `documentation/`, or `process/`), I'll install the framework inside it. If you don't have one yet, I'll create it at `docs/process/`. Either way, the framework will live in a subfolder called `intent-execution-framework/`."

Record the answer and set **`FRAMEWORK_ROOT`**:
- If the engineer names an existing folder (e.g. `docs/`, `wiki/`, `process/`): `FRAMEWORK_ROOT` = `{named path}/intent-execution-framework`
- If the engineer says there is no such folder: `FRAMEWORK_ROOT` = `docs/process/intent-execution-framework`

Use `FRAMEWORK_ROOT` as the base path for every framework file created during this session. **Do not create framework files under `process-onboarding-agent/`** — that folder is the bootstrap agent, not the installed framework.

> **After the completion report is delivered**, remind the engineer: "You can now delete `process-onboarding-agent/` from your project root — it is only needed for this onboarding session and serves no purpose in the running framework. Keep `process-estimation-agent/` — it is used for delivery estimation once mob elaboration produces units."

---

### Step 0 — Orient yourself

1. Read `process-onboarding-agent/setup-guide.md` from top to bottom before taking any action.
2. Check whether AI-DLC has already been set up in this repo by looking for files that are **only created during onboarding** — not files that ship with the base repo. The following are reliable indicators of an existing setup (check at `FRAMEWORK_ROOT`):
   - A master rule file at the repo root: `CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`
   - Any of these generated rules files: `{FRAMEWORK_ROOT}/rules/prompt-quality-gate.md`, `{FRAMEWORK_ROOT}/rules/code-standards.md`, `{FRAMEWORK_ROOT}/rules/security.md`, `{FRAMEWORK_ROOT}/rules/architecture.md`
   - Any of these generated skills files: `{FRAMEWORK_ROOT}/skills/mob-elab-prompts.md`, `{FRAMEWORK_ROOT}/skills/review-checklist.md`, `{FRAMEWORK_ROOT}/skills/unit-template.md`
   - Any file inside `{FRAMEWORK_ROOT}/guidelines/`
   - `{FRAMEWORK_ROOT}/Instructions2FDE.md`

   **Do not treat the following as evidence of existing setup** — they are part of the base repo and are present in every fresh copy:
   - `process-onboarding-agent/ops/` and all files inside it (including `process-onboarding-agent/ops/inception/dependency-map.md`)
   - `process-onboarding-agent/rules/engagement.md`
   - `process-onboarding-agent/skills/compact-docs.md`
   - `process-onboarding-agent/skills/root-cause-analysis.md`
   - `process-onboarding-agent/skills/design-session.md`
   - `process-onboarding-agent/skills/bolt-risk-assessment.md`
   - `process-onboarding-agent/skills/progress-digest.md`
   - `process-onboarding-agent/skills/uat.md`
   - `process-onboarding-agent/skills/process-health.md`
   - `process-onboarding-agent/skills/dependency-audit.md`
   - `process-onboarding-agent/skills/knowledge-promotion.md`
   - `process-onboarding-agent/skills/new-engineer-induction.md`
   - `process-onboarding-agent/setup-guide.md`
   - `process-onboarding-agent/onboard.md`

   If any indicator from the first list exists, report what you found and ask the engineer whether to extend the existing setup or start fresh.
3. Do not create or modify any files until Step 0 is complete.

### Step 1 — Identify the AI tool

Ask the engineer:

> "Which AI tool will be the primary tool for this project — Claude Code, Cursor, or GitHub Copilot? (You can add mirror files for additional tools later.)"

Record the answer. Use it to determine the master rule file name and path (see the table in `process-onboarding-agent/setup-guide.md` — Before You Begin).

### Step 2 — Identify the project type

Ask the engineer:

> "Is this a fresh project with no significant codebase yet, or a mature project with existing code and team conventions?"

- **Fresh** → go to Step 3-F below.
- **Mature** → go to Step 3-M below.

---

## Path F — Fresh Project

### Step 3-F — Run the structured interview

Run the nine-question interview defined in `process-onboarding-agent/setup-guide.md` under **Fresh Project — Structured Interview**. Ask one question at a time. Wait for each answer before continuing. Do not skip questions.

### Step 4-F — Create all files

Once all nine questions are answered, execute Steps 1–9 of the setup guide in order. Create all files under `FRAMEWORK_ROOT`:
1. Create the full `{FRAMEWORK_ROOT}/` folder structure with all template files.
2. Write the master rule file (all 9 sections) using interview answers for Sections 1–5 and Section 9. **Replace every `{FRAMEWORK_ROOT}` placeholder in the templates with the actual resolved path** (e.g. `docs/process/intent-execution-framework`). The master rule file must contain real paths — never the literal string `{FRAMEWORK_ROOT}`.
3. Write all `rules/` files using interview answers and guide defaults.
4. Copy all pre-built skills files from `process-onboarding-agent/skills/` into `{FRAMEWORK_ROOT}/skills/` verbatim (do not modify their content). Also copy `process-onboarding-agent/rules/engagement.md` to `{FRAMEWORK_ROOT}/rules/engagement.md` and all `process-onboarding-agent/ops/` template files into `{FRAMEWORK_ROOT}/ops/`.
5. Write all `guidelines/` files (domain glossary pre-populated from interview question 6; others as stubs).
6. Write `Instructions2FDE.md` at `{FRAMEWORK_ROOT}/Instructions2FDE.md`.
7. Write `{FRAMEWORK_ROOT}/README.md`.
8. Create the backlog and the first intent from interview question 8.
9. Create mirror files for any additional tools the engineer wants to support.

### Step 5-F — Deliver the completion report and hand off

Present the completion report defined in `process-onboarding-agent/setup-guide.md` under **Onboarding Completion**.

The completion report includes a mandatory handoff instruction (Section 5 of the report) telling the engineer to close this session and start a new one. **The onboarding agent's job ends here.** Do not continue beyond the handoff instruction.

After presenting the report, add this note:

> "The framework has been installed at `{FRAMEWORK_ROOT}/`. You can now delete `process-onboarding-agent/` from your project root — it is no longer needed. Keep `process-estimation-agent/` — it is used for delivery estimation once mob elaboration produces units."

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

1. Write the master rule file populated from archaeology output (Sections 1–5 from what you found; Sections 6–8 from guide defaults). **Replace every `{FRAMEWORK_ROOT}` placeholder in the templates with the actual resolved path** (e.g. `docs/process/intent-execution-framework`). The master rule file must contain real paths — never the literal string `{FRAMEWORK_ROOT}`.
2. Create `{FRAMEWORK_ROOT}/guidelines/forbidden-zones.md` — ask the engineer: *"Which files, modules, or patterns must the AI never modify without senior engineer approval?"* Record their answer as the initial forbidden zones list.
3. Create `{FRAMEWORK_ROOT}/guidelines/entry-points.md` — ask the engineer: *"Which modules or features should AI-DLC Bolts start with?"* Record their answer as the initial entry points list.
4. Create `{FRAMEWORK_ROOT}/rules/code-standards.md` from extracted patterns.
5. Create the full `{FRAMEWORK_ROOT}/` folder structure with all remaining files and templates. Copy all pre-built skills files from `process-onboarding-agent/skills/` into `{FRAMEWORK_ROOT}/skills/` verbatim. Also copy `process-onboarding-agent/rules/engagement.md` to `{FRAMEWORK_ROOT}/rules/engagement.md` and all `process-onboarding-agent/ops/` template files into `{FRAMEWORK_ROOT}/ops/`.

### Step 5-M — Blast radius controls (Phase M3)

1. Report current test coverage for the first entry-point module (run tests if possible, or report that coverage cannot be determined and flag it as a required manual check).
2. Ask the engineer: *"What feature flag mechanism does this project use, or should we establish one?"* Record the answer in the master rule file Code Rules section.
3. Confirm the default AC *"All integration tests for [affected module] pass without modification"* has been added to the master rule file Section 6 and to `{FRAMEWORK_ROOT}/rules/code-standards.md`.

### Step 6-M — Deliver the completion report and hand off

Present the completion report defined in `process-onboarding-agent/setup-guide.md` under **Onboarding Completion**.

The completion report includes a mandatory handoff instruction (Section 5 of the report) telling the engineer to close this session and start a new one. **The onboarding agent's job ends here.** Do not continue beyond the handoff instruction.

After presenting the report, add this note:

> "The framework has been installed at `{FRAMEWORK_ROOT}/`. You can now delete `process-onboarding-agent/` from your project root — it is no longer needed. Keep `process-estimation-agent/` — it is used for delivery estimation once mob elaboration produces units."
