# AI-DLC Onboarding Agent

The canonical source of the AI-DLC framework at 99x. This repository contains everything needed to onboard any project — new or existing — to the AI-DLC process using Claude Code, Cursor, or GitHub Copilot.

---

## What Is AI-DLC?

AI-DLC is a structured operating system for building software with AI assistance. It is not a tool — it is a set of files, conventions, and ceremonies that govern how a team and its AI work together. The output of onboarding is a repository where:

- Every feature starts with a written intent and testable acceptance criteria
- Every AI interaction is gated by a quality check and logged for audit
- Every failure feeds back into rules that prevent recurrence
- Any engineer (or AI tool) opening the repo knows exactly how to work in it

The process runs in three phases that form a continuous loop:

```
Inception → Build → Operate → Improvements → (back to Inception)
```

---

## What Is This Repository?

This repo is the **base template** — the source of truth that gets copied into each project repo. It contains:

| File / Folder | Purpose |
|---|---|
| `ai-dlc/setup-guide.md` | The complete framework specification. Describes every file, rule, and ceremony. |
| `ai-dlc/onboard.md` | The bootstrap trigger. Drop this into a target repo and follow the instructions to start onboarding. |
| `ai-dlc/skills/compact-docs.md` | Engineer-triggered skill to archive operational documents older than the project's configured threshold. |
| `ai-dlc/ops/inception/intents/_template.md` | Template for writing a feature intent (includes Implementation Summary section, written when all units under the intent are delivered) |
| `ai-dlc/ops/inception/elaborations/_template.md` | Template for logging a mob elaboration session |
| `ai-dlc/ops/build/units/_template.md` | Template for an atomic unit of work |
| `ai-dlc/ops/build/bolts/_template.md` | Template for a planned batch of units |
| `ai-dlc/ops/build/backlog.md` | Starter backlog file |
| `ai-dlc/ops/operate/retros/_template.md` | Template for a bolt retrospective (includes Post-Retro Improvement Workflow — AI-driven, runs immediately after every retro) |
| `ai-dlc/ops/operate/incidents/_template.md` | Template for a production incident |
| `ai-dlc/ops/operate/improvements/_template.md` | Template for a process improvement triggered by a retro or incident |

The remaining files (`rules/`, `guidelines/`, and the master rule file) are **generated per project** by the AI agent during onboarding — they cannot be shared across projects because they encode each project's specific stack, domain, and conventions.

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
    skills/
      mob-elab-prompts.md
      review-checklist.md
      compact-docs.md
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
