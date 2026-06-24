# AI-DLC Skills Onboarding Prompt

This agent installs individual AI-DLC skills into a project that uses its own delivery process. No process change is required — you select the skills that fit your existing ceremonies and the agent copies them to a location of your choosing.

---

## Prerequisites — copy these folders from the AI-DLC base repo before starting

Copy both of the following from `repository-agents/` in the base repo into your **project root**:

```
process-skills-agent/       ← this agent (the trigger and guide)
process-onboarding-agent/   ← the source of the skill files
```

---

## Trigger

Once both folders are in your project root, say the following to your AI assistant:

> **"Read `process-skills-agent/onboard.md` and follow the instructions inside it."**

---

## Instructions for the AI Agent

You are about to install individual AI-DLC skills into a project that uses its own bespoke delivery process. Read and follow `process-skills-agent/skills-guide.md` in full. The steps below tell you how to execute it.

---

### Session Brief — Present this to the engineer before asking anything

Before asking any questions or taking any action, present the following overview:

> **AI-DLC Skills Onboarding — Session Overview**
>
> This session installs individual AI-DLC skills into your project. It does **not** install the full AI-DLC framework — no master rule file, no governance structure, no process change. You keep your existing delivery process exactly as it is.
>
> Each skill is a self-contained prompt protocol. You invoke it by reading the skill file to your AI assistant at the moment you need it. The skills are standalone capabilities that enhance specific moments in any delivery process — planning, risk assessment, incident analysis, stakeholder communication, and more.
>
> **What will happen in this session:**
> 1. I'll ask five questions about your existing process to understand what you already do and where AI assistance would add the most value.
> 2. I'll present a curated catalogue of available skills, grouped by delivery moment, with a dependency note for each — some skills work standalone, others need minor configuration, and a few require the full AI-DLC framework to be in place.
> 3. You'll select the skills you want to install.
> 4. I'll ask where in your project you want the skill files to live.
> 5. I'll copy the selected skill files to that location and write an Adoption Card — a single reference document that lists each installed skill, how to invoke it, and what it produces.
>
> **Expected output from this session:**
> - The selected skill `.md` files at your chosen location
> - An `ai-dlc-skills-adoption-card.md` file at that location — your team's quick reference for invoking each skill
>
> Ready to start? I have five questions before we choose skills.

---

### Step 0 — Pre-flight check

Before asking any questions, verify:

1. `process-onboarding-agent/skills/` exists and contains the skill files. If it is absent, stop and tell the engineer: *"Please copy `process-onboarding-agent/` from the AI-DLC base repo into this project root before continuing."*
2. Read `process-skills-agent/skills-guide.md` from top to bottom.
3. Do not copy any files or create any documents until Step 5 of the guide.
