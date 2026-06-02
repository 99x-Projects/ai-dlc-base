# AI-DLC Agent Suite for IDEs

The canonical source of the AI-DLC framework at 99x. This repository contains everything needed to onboard, operate, and audit any software project using AI assistance — with Claude Code, Cursor, or GitHub Copilot.

---

## What Is AI-DLC?

AI-DLC is a structured operating system for building software with AI assistance. It is not a tool — it is a set of files, conventions, and ceremonies that govern how a team and its AI work together. The output is a repository where:

- Every feature starts with a written intent and testable acceptance criteria
- Every AI interaction is gated by a quality check and logged for audit
- Every failure feeds back into rules that prevent recurrence
- Any engineer (or AI tool) opening the repo knows exactly how to work in it

The process runs in three phases that form a continuous loop:

```
Inception → Build → Operate → Improvements → (back to Inception)
```

---

## Supported AI Tools

AI-DLC works with any of the three AI coding assistants below. The framework content is identical across all three — only the master rule file name and location differ.

| AI Tool | Master rule file | Location in project repo |
|---|---|---|
| **Claude Code** | `CLAUDE.md` | Repo root |
| **Cursor** | `.cursorrules` | Repo root |
| **GitHub Copilot** | `copilot-instructions.md` | `.github/` folder |

Each tool loads its master rule file automatically at the start of every session. The onboarding agent asks which tool the team uses and creates the correct file. Mirror files for additional tools can be created at any time — see the setup guide for instructions.

---

## Three Agentic Capabilities

This repository delivers three distinct AI agent behaviours. Each serves a different stage of a project's relationship with AI-DLC.

---

### 1. Onboarding Agent

**What it does:** Installs the AI-DLC framework into a project that does not yet have it.

**Mode:** Push — the agent drives the entire setup process and delivers a ready-to-use governance layer.

**When to use:** When a project is starting fresh or when an existing project wants to adopt AI-DLC for the first time.

**How it works:**
- For **fresh projects**: runs a nine-question structured interview to extract the project's identity, stack, domain language, and constraints, then generates all framework files from those answers.
- For **mature projects**: performs a phased codebase archaeology (architecture mapping, pattern extraction, due diligence audit, debt classification) before generating the framework, ensuring the AI inherits the existing project's conventions rather than overwriting them.

The output is a fully configured `ai-dlc/` folder with a master rule file, rules, skills, guidelines, ops templates, and a completion report flagging anything that still needs engineer input.

**Entry point:** `ai-dlc/onboard.md`

---

### 2. Experience Agent

**What it does:** Governs how the AI behaves inside a project on a day-to-day basis once AI-DLC is installed.

**Mode:** Ambient — the agent is always active. It is not invoked separately; it is what the AI *becomes* once the master rule file and framework files are loaded at the start of every session.

**When to use:** Every session in an onboarded project. The experience agent is the normal working mode.

**How it works:** The master rule file (generated during onboarding) is a routing table that loads the agent's behaviour from the framework files:
- Enforces the **Prompt Quality Gate** before any code is generated
- Runs **mob elaboration** turn by turn — one unit at a time, ACs confirmed before edge cases, sign-off before file creation
- Monitors **engineer engagement** and intervenes when disengagement signals are detected
- Executes the **Post-Retro Improvement Workflow** automatically after every retro — synthesising findings into improvement proposals, applying approved changes to rules and skills files
- Writes an **Implementation Summary** when all units under an intent are delivered
- Suggests running **Root Cause Analysis** when an incident is resolved
- Responds to engineer-triggered skills: `compact-docs` to archive old operational documents, `root-cause-analysis` to analyse patterns across incidents and improvements

The experience agent compounds in quality over time — every retro tightens the rules, every RCA surfaces deeper gaps, and every improvement makes the next bolt safer.

**Entry point:** The master rule file at the repo root (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`)

---

### 3. Diagnostics Agent

**What it does:** Audits a project that is already running AI-DLC and identifies gaps between its current practice and AI-DLC principles.

**Mode:** Pull — the agent requests artifacts from the engineer, scores them against structured rubrics, and delivers a prioritised review report.

**When to use:** Periodically on any active AI-DLC project — after the first few bolts to validate early practice, or whenever the team suspects the process has drifted.

**How it works:** The agent adopts the role of an AI-DLC Process Reviewer and runs through six review domains:
1. **Foundation** — is the master rule file a routing table or a wall of text? Are sections complete and accurate?
2. **Inception** — are intent files written from the user's perspective? Do elaboration sessions follow the turn structure?
3. **Build** — are ACs testable? Is scope bounded? Are pre-generation checks being run?
4. **Operate** — are retros closing the feedback loop? Are improvements actually applied to the target files?
5. **Process Adherence** — assessed through conversation, not files: is the quality gate enforced? Are ACs challenged or passively accepted?
6. **Organization & Structure** — based on where the engineer retrieves artifacts from, are like artifacts colocated, consistently named, cross-linked, and discoverable?

The agent never asks for files by specific path — the engineer shares whatever they have from wherever they keep it. The agent evaluates content quality and organizational structure independently.

The output is a comprehensive Review Report containing: a domain-by-domain scorecard, findings by severity, an Organization Assessment, a Gap Analysis mapping each finding to the specific AI-DLC principle violated, a Remediation Plan (Immediate / Short-term / Long-term actions, produced only when Critical or Important findings exist), a single "First Action" recommendation, and a Patterns section flagging process drift signals.

**Entry point:** `ai-dlc-reviewer/role-play.md`

---

## What Is This Repository?

This repo is the **base template** — the source of truth that gets copied into each project repo. It contains:

| File / Folder | Purpose |
|---|---|
| `ai-dlc/setup-guide.md` | The complete framework specification. Describes every file, rule, and ceremony. |
| `ai-dlc/onboard.md` | The bootstrap trigger. Drop this into a target repo and follow the instructions to start onboarding. |
| `ai-dlc/skills/compact-docs.md` | Engineer-triggered skill to archive operational documents older than the project's configured threshold. |
| `ai-dlc/skills/root-cause-analysis.md` | Skill to run 5-Whys analysis on incidents and improvements, classify root causes (Solution Design / Technology / Process), surface cross-cutting patterns, and produce linked intent or improvement artifacts. |
| `ai-dlc/rules/engagement.md` | Engineer engagement monitoring — signals of disengagement, intervention script, and escalation protocol. Copied verbatim into every project. |
| `ai-dlc/ops/inception/intents/_template.md` | Template for writing a feature intent (includes Implementation Summary section, written when all units under the intent are delivered) |
| `ai-dlc/ops/inception/elaborations/_template.md` | Template for logging a mob elaboration session |
| `ai-dlc/ops/build/units/_template.md` | Template for an atomic unit of work |
| `ai-dlc/ops/build/bolts/_template.md` | Template for a planned batch of units |
| `ai-dlc/ops/build/backlog.md` | Starter backlog file with Reference Link Registry — all unit/bolt links use reference-style Markdown so compact-docs only updates the registry, never the table rows |
| `ai-dlc/ops/operate/retros/_template.md` | Template for a bolt retrospective (includes Post-Retro Improvement Workflow — AI-driven, runs immediately after every retro) |
| `ai-dlc/ops/operate/incidents/_template.md` | Template for a production incident |
| `ai-dlc/ops/operate/improvements/_template.md` | Template for a process improvement triggered by a retro or incident |

The remaining files (most of `rules/`, all of `guidelines/`, and the master rule file) are **generated per project** by the AI agent during onboarding — they cannot be shared across projects because they encode each project's specific stack, domain, and conventions.

---

## How to Use This Repository

### Onboarding a new project

1. **Copy the `ai-dlc/` folder** from this repo into the root of your target repository.

2. **Open your AI assistant** (Claude Code, Cursor, or GitHub Copilot) inside the target repo.

3. **Say:**
   > "Read `ai-dlc/onboard.md` and follow the instructions inside it."

4. The agent will ask you which AI tool you're using and whether the project is fresh or mature, then run the onboarding process end to end — including a nine-question interview (fresh projects) or a phased codebase archaeology (mature projects) — creating all framework files, populating the master rule file, and delivering a completion report.

5. **Review the completion report.** Fill in any sections the agent flagged as requiring engineer input before the first Bolt runs.

### What the agent creates in your project repo

After onboarding, your target repo will contain:

```
<your-project>/
  CLAUDE.md (or .cursorrules / .github/copilot-instructions.md)
  ai-dlc/
    setup-guide.md
    onboard.md
    rules/
      prompt-quality-gate.md
      code-standards.md
      security.md
      architecture.md
      engagement.md
    skills/
      mob-elab-prompts.md
      review-checklist.md
      compact-docs.md
      root-cause-analysis.md
    guidelines/
      domain-glossary.md
      edge-cases.md
      acceptance-patterns.md
      forbidden-zones.md      ← mature projects only
      entry-points.md         ← mature projects only
      dev-setup.md
      team-rollout.md
    prompts/
    ops/
      inception/intents/
      inception/elaborations/
      build/backlog.md
      build/units/
      build/bolts/
      operate/retros/
      operate/incidents/
      operate/improvements/
    Instructions2FDE.md
    README.md
```

---

## AI-DLC Reviewer (Diagnostics)

For teams already running AI-DLC, the `ai-dlc-reviewer/` folder provides a complementary **pull** mode — a structured role-playing audit session that validates an existing project's artifacts and practice against AI-DLC principles.

| File | Purpose |
|---|---|
| `ai-dlc-reviewer/role-play.md` | Bootstrap trigger. Copy into the target repo and read it to the AI to start a review session. |
| `ai-dlc-reviewer/review-guide.md` | Full review protocol — five domains, rubrics for every artifact type, scoring system, and report format. |

**How to use:**

1. Copy the `ai-dlc-reviewer/` folder into the root of the project repo being audited.
2. Open your AI assistant inside that repo.
3. Say: `"Read ai-dlc-reviewer/role-play.md and follow the instructions inside it."`
4. The AI adopts the reviewer persona and runs through six review domains — Foundation, Inception, Build, Operate, Process Adherence, and Organization & Structure — requesting artifacts, scoring them against rubrics, and delivering a comprehensive report with gap analysis and a prioritised remediation plan.

The reviewer never generates code or creates files unprompted. It is a diagnostic tool, not an onboarding tool.

---

## How to Update This Repository

When any 99x project completes a retro and produces an improvement that changes a **generic rule** (one that should apply to all projects, not just that project's specific stack), bring it back here:

1. Open a PR against this repo
2. Apply the change to the relevant file in `ai-dlc/`
3. Note the source project and retro in the PR description

This is how the framework compounds across teams over time.

---

## Reference

- Full framework specification: [ai-dlc/setup-guide.md](ai-dlc/setup-guide.md)
- Agent bootstrap instructions: [ai-dlc/onboard.md](ai-dlc/onboard.md)
