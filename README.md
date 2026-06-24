# 99x Intent Delivery Framework for AI-DLC

The canonical source of the 99x Intent Delivery Framework at 99x. This is the intent-driven delivery subsystem within the broader AI-DLC methodology — containing everything needed to onboard, operate, and audit any software project using AI assistance — with Claude Code, Cursor, or GitHub Copilot.

---

## What Is the 99x Intent Delivery Framework?

The 99x Intent Delivery Framework is the intent-first delivery operating system within 99x's broader AI-DLC methodology. AI-DLC is a vast set of tools, practices, and principles for AI-assisted software delivery — this framework is the subsystem that governs how a team structures, executes, and governs its day-to-day work using those principles. It is not a tool — it is a set of files, conventions, and ceremonies that govern how a team and its AI work together. The output is a repository where:

- Every feature starts with a written intent and testable acceptance criteria
- Every AI interaction is gated by a quality check and logged for audit
- Every failure feeds back into rules that prevent recurrence
- Any engineer (or AI tool) opening the repo knows exactly how to work in it

The process runs in three phases that form a continuous loop:

```
Inception → Build → Operate → Improvements → (back to Inception)
```

Rules, skills, and guidelines are generated through conversation with your team — not stamped from a template. → [What to expect in detail](readme-content/agents.md)

> **Scope note:** This repository covers the intent-driven delivery layer of AI-DLC. It does not represent the full AI-DLC methodology, which spans tooling, governance, team practices, and more.

---

## Supported AI Tools

AI-DLC works with any of the three AI coding assistants below. The framework content is identical across all three — only the master rule file name and location differ.

| AI Tool | Master rule file | Location in project repo |
|---|---|---|
| **Claude Code** | `CLAUDE.md` | Repo root |
| **Cursor** | `.cursorrules` | Repo root |
| **GitHub Copilot** | `copilot-instructions.md` | `.github/` folder |

Each tool loads its master rule file automatically at the start of every session.

---

## Agent Suite

| Agent | What it does | Entry point |
|---|---|---|
| **Onboarding** | Installs the 99x Intent Delivery Framework through a structured interview or codebase archaeology session | `process-onboarding-agent/onboard.md` |
| **Experience** | Governs AI behavior on every working session — quality gates, elaboration, bolts, retros, UAT | Master rule file (auto-loaded at session start) |
| **Diagnostics** | Audits process against AI-DLC principles and delivers a structured gap report — works with or without the framework installed | `process-diagnostic-agent/role-play.md` |
| **Skills** | Installs individual framework skills into a bespoke delivery process without installing the full framework | `process-skills-agent/onboard.md` |
| **Migration** | Migrates a project from the old `ai-dlc/` structure to the new methodology | `process-migration-agent/migrate.md` |
| **Estimation** | Produces structured estimates — ballpark ranges from draft requirements (pre-sales/feasibility) or bolt-level delivery estimates from elaborated units (release planning) | `process-estimation-agent/estimate.md` |

→ [Detailed agent descriptions and diagrams](readme-content/agents.md)

---

## Getting Started

### Onboarding a new project

This is a two-session process: the onboarding agent installs the framework in session 1; the experience agent is live from session 2 onward.

**Session 1:**
1. Copy `process-onboarding-agent/` from `repository-agents/` into your project root.
2. Open your AI assistant inside the project repo.
3. Say: `"Read process-onboarding-agent/onboard.md and follow the instructions inside it."`
4. The agent runs a structured interview (fresh projects) or codebase archaeology (mature projects), creates all framework files, and delivers a completion report.
5. Review the completion report and fill in any flagged items before the first bolt.
6. Close this session — the onboarding conversation is complete.

**Session 2 onwards:**
7. Open a new session. Your master rule file loads automatically — the experience agent is now live.
8. Say: `"Run a mob elaboration for [first intent name]"` to start your first feature.

### Running a diagnostic review

1. Copy `process-diagnostic-agent/` from `repository-agents/` into your project root.
2. Open your AI assistant inside the project repo.
3. Say: `"Read process-diagnostic-agent/role-play.md and follow the instructions inside it."`

Works with any project — no AI-DLC installation required. → [Full diagnostics guide](readme-content/diagnostics-and-review.md)

### Installing skills for a bespoke process

1. Copy both `process-skills-agent/` and `process-onboarding-agent/` from `repository-agents/` into your project root.
2. Open your AI assistant inside the project repo.
3. Say: `"Read process-skills-agent/onboard.md and follow the instructions inside it."`

→ [Skills for bespoke processes](readme-content/skills-for-bespoke-processes.md)

### Migrating from the old `ai-dlc/` structure

1. Copy `process-migration-agent/`, `process-onboarding-agent/`, and `process-diagnostic-agent/` from `repository-agents/` into the project root.
2. Open your AI assistant inside the project repo.
3. Say: `"Read process-migration-agent/migrate.md and follow the instructions inside it."`

→ [Full migration guide](readme-content/migrating-from-old-structure.md)

### Producing an estimate

**Ballpark (pre-sales / feasibility):**
1. Copy `process-estimation-agent/` from `repository-agents/` into your working folder.
2. Open your AI assistant.
3. Say: `"Read process-estimation-agent/estimate.md and follow the instructions inside it."` — select **Mode 1** when prompted.

**Delivery (bolt-level / release planning):**
1. The framework must already be installed and mob elaboration complete.
2. Copy `process-estimation-agent/` from `repository-agents/` into your project root.
3. Say: `"Read process-estimation-agent/estimate.md and follow the instructions inside it."` — select **Mode 2** when prompted.

→ [Full estimation guide](readme-content/estimation.md)

---

## Further Reading

- [Agent descriptions and diagrams](readme-content/agents.md) — detailed behavior lists, how the agents work together, artifact lifecycle
- [Repository contents reference](readme-content/repository-contents.md) — every file and folder in this repo with its purpose
- [Diagnostics and review guide](readme-content/diagnostics-and-review.md) — full diagnostics guide including scope options, People and Tools review
- [Skills for bespoke processes](readme-content/skills-for-bespoke-processes.md) — skills catalogue, dependency classifications, adoption card
- [Incremental adoption activities](readme-content/incremental-adoption.md) — Phase 1–3 activities for teams not ready to fully onboard
- [Migrating from the old structure](readme-content/migrating-from-old-structure.md) — from `ai-dlc/` to the new methodology
- [Contributing improvements back](readme-content/contributing.md) — how the base repo evolves through knowledge promotion
- [Estimation guide](readme-content/estimation.md) — ballpark and delivery estimation modes, tier classifications, calibration from recorded hours

---

## Reference

- Full framework specification: [repository-agents/process-onboarding-agent/setup-guide.md](repository-agents/process-onboarding-agent/setup-guide.md)
- Agent bootstrap instructions: [repository-agents/process-onboarding-agent/onboard.md](repository-agents/process-onboarding-agent/onboard.md)
