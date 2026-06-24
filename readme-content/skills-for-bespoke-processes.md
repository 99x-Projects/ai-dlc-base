# Skills for Bespoke AI-Driven Delivery Processes

← [Back to README](../README.md)

---

## Overview

Teams that already have a working AI-assisted delivery process — and want to continue using it — can adopt individual AI-DLC skills without installing the full framework. No master rule file is generated, no governance structure is imposed, and no process change is required.

Each skill is a self-contained prompt protocol: a markdown file the engineer reads to their AI assistant at the moment they need it. The agent interviews the team about their existing process, presents a curated catalogue of available skills grouped by delivery moment, and copies the selected skills to a location of the team's choosing. The output is the selected skill files plus an Adoption Card — a single reference document listing each installed skill, how to invoke it, and what it produces.

---

## Skill Dependency Classifications

Skills are classified by dependency so teams can make informed choices:

| Classification | Meaning |
|---|---|
| **◆ Standalone** | Works without any other AI-DLC files — invoke directly |
| **◈ Needs config** | Works after one small piece of information is provided at invocation (e.g. archive threshold, output path) |
| **◇ Framework-only** | Requires the full AI-DLC framework to be installed first |

**Standalone and needs-config skills** (usable immediately): Design Session, Risk Assessment, UAT Sign-off, Root Cause Analysis, Progress Digest, Knowledge Promotion, Dependency Audit, Compact Docs, and Process Health.

**Framework-only skills** (Bug Bolt, NFR Bolt, New Engineer Induction) can be installed as reference material for future use if your team plans to adopt AI-DLC fully later.

---

## How to Use

1. Copy both folders from `repository-agents/` in this repo into your project root:
   - `process-skills-agent/`
   - `process-onboarding-agent/` (source of the skill files)

2. Open your AI assistant inside the project repo.

3. Say:
   > "Read `process-skills-agent/onboard.md` and follow the instructions inside it."

4. The agent interviews you about your existing process (five questions), presents the skills catalogue with recommendations tailored to your answers, installs your selected skills at a location you choose, and writes the Adoption Card.

---

## What Happens in the Session

**Step 1 — Process discovery interview.** Five questions about how your team currently works, where AI assistance already helps, common failure modes, stakeholder communication, and what gap you want to fill. The agent uses your answers to flag relevant skills with a ★ Recommended marker.

**Step 2 — Skills catalogue.** The full catalogue of 13 skills, grouped by delivery moment (Planning & Design, Quality & Acceptance, Incident & Root Cause, Communication & Reporting, Maintenance & Health, New Team Members, Workflow Shortcuts), with dependency classification for each.

**Step 3 — Selection.** You choose which skills to install. The agent warns before installing any ◇ Framework-only skills.

**Step 4 — Installation location.** You choose where the skill files live in your project (e.g. `docs/ai-skills/`, `process/skills/`, or any path that fits your structure).

**Step 5 — Installation.** Selected skill files are copied from `process-onboarding-agent/skills/` to your chosen location.

**Step 6 — Adoption Card.** The agent writes `ai-dlc-skills-adoption-card.md` at your chosen location — a single reference listing every installed skill, its invocation phrase, what it produces, and any configuration needed.

---

## Upgrading to Full AI-DLC

If your team later decides to adopt the full AI-DLC framework, the onboarding agent's archaeology path will read your existing project and generate governance files that match your actual conventions — the skills you already installed carry over without change.

Copy `repository-agents/process-onboarding-agent/` into your repo and say:
> "Read `process-onboarding-agent/onboard.md` and follow the instructions inside it."

The archaeology path reads what you have already built and extends it rather than starting from scratch.
