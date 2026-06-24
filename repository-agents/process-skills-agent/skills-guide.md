# AI-DLC Skills Guide

This document is the protocol for the AI-DLC Skills Onboarding Agent. It defines the discovery interview, the skills catalogue with dependency classifications, the selection and installation process, and the Adoption Card format.

---

## Purpose

This agent serves teams that already have a working AI-assisted delivery process and want to adopt individual AI-DLC capabilities without installing the full framework. The output is a small set of skill files at a chosen location, plus an Adoption Card the team can refer to whenever they want to invoke a skill.

---

## Step 1 — Process Discovery Interview

Ask the following five questions one at a time. Wait for each answer before continuing. Record all answers — they determine which skills to recommend and flag as most relevant in Step 2.

1. > "Walk me through how a typical feature gets built on your team — from the moment someone decides to build it to the moment it is shipped. What are the main steps and ceremonies?"

   *Recording for:* where planning, elaboration, review, and delivery moments exist — which skills map to which steps.

2. > "When your team uses AI assistance today, what does that look like? What do you ask it to do, and where in the process does it help the most?"

   *Recording for:* current AI touchpoints and gaps — which skills would extend rather than duplicate existing practice.

3. > "What are the most common failure modes on your team — the things that go wrong repeatedly? Think about bugs, missed requirements, production incidents, or process slippage."

   *Recording for:* root-cause-analysis, hotfix, dependency-audit relevance.

4. > "How do you communicate progress and decisions to stakeholders — product owners, clients, or leadership?"

   *Recording for:* progress-digest, knowledge-promotion relevance.

5. > "Is there a specific capability gap you were hoping AI-DLC skills might fill — something your current process does not do well?"

   *Recording for:* direct signal on where to focus the recommendation.

After all five questions are answered, summarise your understanding back to the engineer in two or three sentences before moving to Step 2. Give them a chance to correct anything before you present the catalogue.

---

## Step 2 — Present the Skills Catalogue

Present the full catalogue below. Group it by delivery moment. For each skill:
- Lead with what it does and when to use it
- State the dependency classification clearly
- Flag with a `★ Recommended` marker any skill that directly addresses something the engineer mentioned in the discovery interview

Use this dependency classification key at the top of the catalogue:

```
◆ Standalone      — works without any other AI-DLC files
◈ Needs config    — works after one small piece of information is provided (explained below)
◇ Framework-only  — requires the full AI-DLC framework to be installed first
```

---

### PLANNING & DESIGN

**Design Session** (`design-session.md`) `◆ Standalone`

Runs a structured Phase 0 design session before any building begins. Locks down API contracts, data model decisions, and architectural patterns through conversation, then produces a design artifact the team can reference during build.

- **When to invoke:** at the start of planning a new feature, before breaking it into tasks
- **How to invoke:** `"Read [skill-path]/design-session.md and run a design session for [feature name]."`
- **What you get:** a design document with API surface, data model sketch, architectural decisions, and binding constraints for the build team
- **Adapts to your process:** the output path is configurable — the engineer states where to write the design artifact

---

**Risk Assessment** (`bolt-risk-assessment.md`) `◆ Standalone`

Runs a structured risk assessment before a batch of work executes. Interrogates blast radius, cross-component risks, rollback options, and whether a feature flag is warranted.

- **When to invoke:** before starting a sizeable feature, refactor, or infrastructure change
- **How to invoke:** `"Read [skill-path]/bolt-risk-assessment.md and run a risk assessment for [work description]."`
- **What you get:** a risk summary covering blast radius, sequencing risks, rollback feasibility, and feature flag recommendation

---

### QUALITY & ACCEPTANCE

**UAT Sign-off** (`uat.md`) `◆ Standalone`

Generates a plain-language UAT demo script from a set of acceptance criteria, walks the engineer through each step, and records pass/fail/deferred outcomes. Blocks closure of a feature until a sign-off status is recorded.

- **When to invoke:** when a feature is complete and ready for acceptance testing with the stakeholder or product owner
- **How to invoke:** `"Read [skill-path]/uat.md and run a UAT session for [feature name] using these acceptance criteria: [paste ACs]."`
- **What you get:** a structured walkthrough of each AC, a recorded outcome per step, and a sign-off status

---

### INCIDENT & ROOT CAUSE

**Root Cause Analysis** (`root-cause-analysis.md`) `◆ Standalone`

Applies structured 5-Whys analysis to a resolved incident or recurring problem. Classifies the root cause as Solution Design, Technology Selection, or Process, and proposes concrete remediation actions.

- **When to invoke:** after a production incident is resolved, or when a pattern of recurring bugs is noticed
- **How to invoke:** `"Read [skill-path]/root-cause-analysis.md and run a root cause analysis on [incident description]."`
- **What you get:** a 5-Whys chain, root cause classification, and recommended remediation steps

---

**Hotfix Workflow** (`hotfix-bolt.md`) `◈ Needs config`

Runs an emergency intake for a production incident: three questions (symptom, severity, rollback availability), a minimal unit of work, blast radius check, and a mandatory retro within 24 hours.

- **When to invoke:** when a production incident needs an immediate response
- **How to invoke:** `"Read [skill-path]/hotfix-bolt.md and start a hotfix for [symptom]."`
- **What you get:** a structured intake, a focused fix scope, and a retro prompt
- **Configuration needed:** the skill references a bolt file and retro template from the AI-DLC framework. For bespoke process use, tell the AI where to write the incident record when it asks. The core intake and blast radius check work standalone; the retro link is advisory.

---

### COMMUNICATION & REPORTING

**Progress Digest** (`progress-digest.md`) `◆ Standalone`

Generates a plain-language, one-page progress summary for a feature — written for non-technical stakeholders. Translates the current state of a feature into what is being built, what is done, and what comes next.

- **When to invoke:** when a stakeholder update is needed, or before a sprint review
- **How to invoke:** `"Read [skill-path]/progress-digest.md and generate a progress digest for [feature name]. Here is the current state: [describe or paste context]."`
- **What you get:** a stakeholder-ready summary in plain language, saved to a file

---

**Knowledge Promotion** (`knowledge-promotion.md`) `◆ Standalone`

Evaluates improvements or learnings from a retro or incident and classifies each as generic (shareable across teams) or project-specific (keep local). For generic improvements, drafts the exact change needed in shared resources so the engineer can raise it with the wider team or guild.

- **When to invoke:** at the end of a retrospective, after applying improvements
- **How to invoke:** `"Read [skill-path]/knowledge-promotion.md and run knowledge promotion on these retro improvements: [paste improvements]."`
- **What you get:** a classification for each improvement (generic / project-specific) and a drafted contribution for any that are worth sharing

---

### MAINTENANCE & HEALTH

**Dependency Audit** (`dependency-audit.md`) `◈ Needs config`

Reads package manifests, classifies findings by severity (critical / high / medium / low), and produces a remediation list with suggested actions for high and critical issues.

- **When to invoke:** monthly, or before a release, or whenever a security advisory is published for a dependency you use
- **How to invoke:** `"Read [skill-path]/dependency-audit.md and run a dependency audit on this project."`
- **What you get:** a classified findings list and remediation suggestions
- **Configuration needed:** the skill normally schedules the next audit in a master rule file. For bespoke process use, track the next audit date in your own process document and invoke manually. The audit itself works standalone.

---

**Compact Docs** (`compact-docs.md`) `◈ Needs config`

Archives operational documents older than a configured threshold — keeping the active workspace clean without losing institutional memory.

- **When to invoke:** when your process documents folder has grown large and is hard to navigate, or at a cadence your team sets (e.g., end of each sprint)
- **How to invoke:** `"Read [skill-path]/compact-docs.md and archive documents older than [N] days in [folder path]."`
- **What you get:** older documents moved to an archive subfolder; a summary of what was archived
- **Configuration needed:** state the archive threshold (number of days) and the folder to act on when you invoke the skill. The skill normally reads this from a master rule file; without one, you provide it inline.

---

**Process Health** (`process-health.md`) `◈ Needs config`

Computes four process health metrics — improvement adoption rate, quality gate failure rate, AC revision rate, and bolt velocity — and surfaces decay signals. Saves a dated health report automatically.

- **When to invoke:** periodically (e.g., every few sprints) to check whether process discipline is holding or drifting
- **How to invoke:** `"Read [skill-path]/process-health.md and run a process health check. The project is [project name] and our process artifacts are at [folder path]."`
- **What you get:** a scored health report with decay signals and suggested actions
- **Configuration needed:** the skill reads metrics from AI-DLC ops artifacts (retro files, improvement files). For bespoke process use, point it at your equivalent artifacts (retro notes, issue tracker exports, etc.) or use it as a conversation-based check — the AI asks you the questions and computes scores from your answers.

---

### NEW TEAM MEMBERS

**New Engineer Induction** (`new-engineer-induction.md`) `◇ Framework-only`

Walks a new team member through the project's governance framework, explains each section in plain language, demonstrates the quality gate with a project-specific example, and produces a personalized quick-reference card.

- **Framework dependency:** this skill is built around the AI-DLC master rule file and framework files. It reads those files to personalize the session. Without them, it cannot run as designed.
- **If you still want it:** a team without AI-DLC can use it as a checklist prompt — read the skill file and substitute your own process documentation. The AI will ask you where your equivalent files are.

---

### WORKFLOW SHORTCUTS

The following three skills are bolt-type workflow shortcuts designed to operate within a fully installed AI-DLC framework. They reference the quality gate, review checklist, unit/bolt templates, and ops folder structure.

**Bug Bolt** (`bug-bolt.md`) `◇ Framework-only`
Lightweight workflow for fixing a specific, reproducible bug — four-question intake, recurrence check, single focused unit, mandatory RCA if recurring.

**NFR Bolt** (`nfr-bolt.md`) `◇ Framework-only`
Non-functional improvement workflow — measurable threshold AC, before/after baseline, cross-reference to affected features.

> For the above two skills: if you install the full AI-DLC framework later, these slots in seamlessly. You can install them now as reference material if you wish, but they will not run correctly without the framework artifacts in place.

---

## Step 3 — Selection

After presenting the catalogue, ask:

> "Which skills would you like to install? You can name them by number, name, or category — or say 'all standalone' to install everything marked ◆. I'll flag any `◇ Framework-only` skills if you select them and confirm you want to proceed."

Record the selections. If the engineer selects any `◇ Framework-only` skills, confirm:

> "[Skill name] is designed for a fully installed AI-DLC project. I can install it now as reference material, but it won't run correctly without the full framework in place. Do you still want to include it?"

---

## Step 4 — Choose the Installation Location

Ask:

> "Where in your project would you like the skill files to live? I'll copy the selected skills there and write the Adoption Card in the same location.
>
> Common choices:
> - `docs/ai-skills/` — inside your existing docs folder
> - `process/skills/` — next to your existing process documents
> - `ai-dlc-skills/` — a new top-level folder dedicated to AI skills
>
> Or name any path that fits your existing structure."

Record the answer. Set **`SKILLS_ROOT`** to the path the engineer provides.

---

## Step 5 — Install Selected Skills

Copy each selected skill file from `process-onboarding-agent/skills/[filename].md` to `{SKILLS_ROOT}/[filename].md`. Do not modify the skill file content.

After copying, list every installed file and confirm the file count with the engineer before proceeding to Step 6.

---

## Step 6 — Deliver the Adoption Card

Write the following file to `{SKILLS_ROOT}/ai-dlc-skills-adoption-card.md`:

```markdown
# AI-DLC Skills — Adoption Card

Generated: YYYY-MM-DD
Project: [project name from discovery interview]
Skills location: {SKILLS_ROOT}

---

## How to use a skill

Read the skill file to your AI assistant at the moment you need it:

> "Read `{SKILLS_ROOT}/[skill-filename].md` and [invocation phrase]."

Each skill file contains its full protocol. The AI follows it for that session only — there is no persistent configuration.

---

## Installed Skills

| Skill | File | Invoke when | How to invoke |
|---|---|---|---|
[one row per installed skill, populated from the catalogue entries above]

---

## Skills Not Installed

| Skill | Reason | Revisit when |
|---|---|---|
[one row per skill not selected — brief reason (e.g. "not relevant to current process", "framework-only — revisit if adopting full AI-DLC") and a condition for revisiting]

---

## Dependency Notes

[List any ◈ Needs config skills that were installed, with the specific configuration step the engineer needs to provide at invocation time]

---

## Considering full AI-DLC adoption?

If your team reaches a point where you want structured governance across the full delivery cycle — not just individual skills — the onboarding agent installs the complete framework in one session. Copy `process-onboarding-agent/` from the AI-DLC base repo into your project root and say:

> "Read `process-onboarding-agent/onboard.md` and follow the instructions inside it."

The archaeology path will read your existing project and generate rules and guidelines that match your actual conventions, not a generic template.
```

After writing the Adoption Card, present a summary to the engineer:

> "The following skills have been installed at `{SKILLS_ROOT}/`:
>
> [list installed skill filenames]
>
> The Adoption Card at `{SKILLS_ROOT}/ai-dlc-skills-adoption-card.md` is your team's reference for invoking each skill.
>
> You can delete `process-skills-agent/` and `process-onboarding-agent/` from your project root — they are only needed for this session."
