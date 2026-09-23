# AI-DLC Migration Guide

This document is the protocol for the AI-DLC Migration Agent. It migrates a project from the old AI-DLC structure to the new one across all three agents: Onboarding, Experience, and Diagnostic.

---

## Background — What Changed

### Old structure (per project repo)

In the old methodology, a single `ai-dlc/` folder served as both the bootstrap agent and the installed framework. Everything lived there:

```
project-root/
  CLAUDE.md (or .cursorrules / .github/copilot-instructions.md)
  ai-dlc/
    onboard.md                 ← bootstrap trigger
    setup-guide.md             ← onboarding protocol
    rules/
      engagement.md            ← template (pre-built)
      prompt-quality-gate.md   ← GENERATED — project-specific
      code-standards.md        ← GENERATED — project-specific
      security.md              ← GENERATED — project-specific
      architecture.md          ← GENERATED — project-specific
    skills/
      compact-docs.md          ← template (pre-built)
      root-cause-analysis.md   ← template (pre-built)
      solution-shaping.md      ← template (pre-built)
      design-session.md        ← template (pre-built)
      bolt-risk-assessment.md  ← template (pre-built)
      progress-digest.md       ← template (pre-built)
      uat.md                   ← template (pre-built)
      process-health.md        ← template (pre-built)
      dependency-audit.md      ← template (pre-built)
      knowledge-promotion.md   ← template (pre-built)
      ai-hub-metrics.md        ← template (pre-built)
      process-visualization.md ← template (pre-built)
      notifications.md         ← template (pre-built)
      new-engineer-induction.md ← template (pre-built)
      bug-bolt.md              ← template (pre-built)
      hotfix-bolt.md           ← template (pre-built)
      nfr-bolt.md              ← template (pre-built)
      mob-elab-prompts.md      ← GENERATED — project-specific
      review-checklist.md      ← GENERATED — project-specific
    guidelines/
      domain-glossary.md       ← GENERATED — project-specific
      edge-cases.md            ← GENERATED — project-specific
      acceptance-patterns.md   ← GENERATED — project-specific
      dev-setup.md             ← GENERATED — project-specific
      team-rollout.md          ← GENERATED — project-specific
      forbidden-zones.md       ← GENERATED — mature projects only
      entry-points.md          ← GENERATED — mature projects only
    ops/
      inception/intents/       ← operational data — must be preserved
      inception/elaborations/  ← operational data — must be preserved
      inception/dependency-map.md ← operational data — must be preserved
      build/backlog.md         ← operational data — must be preserved
      build/units/             ← operational data — must be preserved
      build/bolts/             ← operational data — must be preserved
      operate/retros/          ← operational data — must be preserved
      operate/incidents/       ← operational data — must be preserved
      operate/improvements/    ← operational data — must be preserved
    Instructions2FDE.md        ← GENERATED — project-specific
    README.md                  ← GENERATED — project-specific
  ai-dlc-reviewer/
    role-play.md               ← diagnostic bootstrap trigger
    review-guide.md            ← diagnostic protocol
```

The master rule file (e.g. `CLAUDE.md`) referenced paths like:
- `ai-dlc/rules/prompt-quality-gate.md`
- `ai-dlc/skills/mob-elab-prompts.md`
- `ai-dlc/guidelines/domain-glossary.md`
- `ai-dlc/ops/inception/intents/`

### New structure

In the new methodology, the bootstrap agents and the installed framework are separated:

```
project-root/
  CLAUDE.md (or .cursorrules / .github/copilot-instructions.md)
  process-onboarding-agent/    ← bootstrap only; delete after use
  process-diagnostic-agent/   ← periodic audits; keep for duration of project
  {docs-folder}/
    intent-execution-framework/
      rules/                   ← generated + template rules
      skills/                  ← generated + template skills
      guidelines/              ← generated guidelines
      ops/                     ← all operational data
      Instructions2FDE.md
      README.md
```

The master rule file references the new paths:
- `{docs-folder}/intent-execution-framework/rules/prompt-quality-gate.md`
- `{docs-folder}/intent-execution-framework/skills/mob-elab-prompts.md`
- etc.

---

## File Classification

Before migrating, understand which files are which:

**Generated (project-specific) — move, do not overwrite:**
- `ai-dlc/rules/prompt-quality-gate.md`
- `ai-dlc/rules/code-standards.md`
- `ai-dlc/rules/security.md`
- `ai-dlc/rules/architecture.md`
- `ai-dlc/skills/mob-elab-prompts.md`
- `ai-dlc/skills/review-checklist.md`
- `ai-dlc/guidelines/` (all files)
- `ai-dlc/Instructions2FDE.md`
- `ai-dlc/README.md` (the project's own README inside ai-dlc)
- `ai-dlc/ops/` (all operational data — intents, units, bolts, retros, etc.)

**Template (pre-built) — move, then refresh from new `process-onboarding-agent/`:**
- `ai-dlc/rules/engagement.md`
- `ai-dlc/skills/compact-docs.md`
- `ai-dlc/skills/root-cause-analysis.md`
- `ai-dlc/skills/solution-shaping.md`
- `ai-dlc/skills/design-session.md`
- `ai-dlc/skills/bolt-risk-assessment.md`
- `ai-dlc/skills/progress-digest.md`
- `ai-dlc/skills/uat.md`
- `ai-dlc/skills/release-readiness-checklist.md`
- `ai-dlc/skills/process-health.md`
- `ai-dlc/skills/dependency-audit.md`
- `ai-dlc/skills/knowledge-promotion.md`
- `ai-dlc/skills/ai-hub-metrics.md`
- `ai-dlc/skills/process-visualization.md`
- `ai-dlc/skills/notifications.md`
- `ai-dlc/skills/new-engineer-induction.md`
- `ai-dlc/skills/bug-bolt.md`
- `ai-dlc/skills/hotfix-bolt.md`
- `ai-dlc/skills/nfr-bolt.md`

**Bootstrap-only — do not migrate (replaced by new `process-onboarding-agent/`):**
- `ai-dlc/onboard.md`
- `ai-dlc/setup-guide.md`

**Diagnostic bootstrap — do not migrate (replaced by new `process-diagnostic-agent/`):**
- `ai-dlc-reviewer/role-play.md`
- `ai-dlc-reviewer/review-guide.md`

---

## Pre-Flight Scan

Before asking any questions or making any changes, scan the repository and report findings to the engineer.

### What to look for

**1. Old-structure indicators (confirm migration is needed):**
- `ai-dlc/onboard.md` or `ai-dlc/setup-guide.md` — confirms old bootstrap agent is present
- `ai-dlc/rules/prompt-quality-gate.md` — confirms old framework was installed
- `ai-dlc-reviewer/role-play.md` — confirms old diagnostic agent is present

**2. New-structure indicators (check for partial migration):**
- `process-onboarding-agent/` folder — already copied from base repo
- `process-diagnostic-agent/` folder — already copied from base repo
- Any `intent-execution-framework/` folder anywhere in the repo

**3. Master rule file:**
- Look for `CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md` at the repo root
- Scan it for path patterns — confirm it contains `ai-dlc/rules/`, `ai-dlc/skills/`, `ai-dlc/guidelines/` references

**4. Operational data:**
- `ai-dlc/ops/inception/intents/` — count files (excluding `_template.md`)
- `ai-dlc/ops/build/units/` — count files (excluding `_template.md`)
- `ai-dlc/ops/build/bolts/` — count files (excluding `_template.md`)
- `ai-dlc/ops/operate/retros/` — count files (excluding `_template.md`)
- Report counts to the engineer so they can verify nothing is lost

### Pre-flight report format

Present findings in this format before asking any questions:

```
Pre-Flight Scan Results
──────────────────────────────────────────
Old-structure files found:
  ✓ ai-dlc/onboard.md
  ✓ ai-dlc/setup-guide.md
  ✓ ai-dlc/rules/prompt-quality-gate.md
  ✓ ai-dlc-reviewer/role-play.md
  [list others found]

Master rule file: CLAUDE.md [or whichever was found]
  Confirmed: references ai-dlc/ paths

Operational data (will be preserved):
  Intents:  [N] files
  Units:    [N] files
  Bolts:    [N] files
  Retros:   [N] files

New agent folders already present:
  [list any of process-onboarding-agent/, process-diagnostic-agent/ already present]
  [or "None — both will need to be set up" if absent]

Migration status: Ready to proceed / Partially migrated / Issues found
──────────────────────────────────────────
```

If `process-onboarding-agent/` or `process-diagnostic-agent/` are absent, stop and tell the engineer:

> "Before we can proceed, copy `repository-agents/process-onboarding-agent/` and `repository-agents/process-diagnostic-agent/` from the AI-DLC base repo into this project root. Then restart this session."

---

## Step 1 — Confirm the New Framework Location

Ask the engineer:

> "Where do you want your process documentation to live? This is the new home for all your framework files — rules, skills, guidelines, and ops data.
>
> Common choices:
> - `docs/process/intent-execution-framework/` (recommended if you don't have a docs folder yet)
> - `{your-existing-docs-folder}/intent-execution-framework/`
>
> All your existing data will be moved there. The master rule file will be updated with the new paths."

Record the answer. Set **`NEW_FRAMEWORK_ROOT`** to the path the engineer provides (e.g. `docs/process/intent-execution-framework`).

Do not proceed until this is confirmed.

---

## Step 2 — Migrate the Onboarding Agent

**What this step does:** Moves all generated and operational content from `ai-dlc/` into the new `intent-execution-framework/` location, refreshes template skill files, and discards the old bootstrap files.

### 2a — Create the new framework folder structure

Create the following folder structure at `{NEW_FRAMEWORK_ROOT}`:

```
{NEW_FRAMEWORK_ROOT}/
  rules/
  skills/
  guidelines/
  prompts/
  ops/
    inception/
      intents/
      elaborations/
    build/
      units/
      bolts/
    operate/
      retros/
      incidents/
      improvements/
```

### 2b — Move generated files

Copy each generated file from its old location to the new location. Do not modify content during the move.

| Source (old) | Destination (new) |
|---|---|
| `ai-dlc/rules/prompt-quality-gate.md` | `{NEW_FRAMEWORK_ROOT}/rules/prompt-quality-gate.md` |
| `ai-dlc/rules/code-standards.md` | `{NEW_FRAMEWORK_ROOT}/rules/code-standards.md` |
| `ai-dlc/rules/security.md` | `{NEW_FRAMEWORK_ROOT}/rules/security.md` |
| `ai-dlc/rules/architecture.md` | `{NEW_FRAMEWORK_ROOT}/rules/architecture.md` |
| `ai-dlc/skills/mob-elab-prompts.md` | `{NEW_FRAMEWORK_ROOT}/skills/mob-elab-prompts.md` |
| `ai-dlc/skills/review-checklist.md` | `{NEW_FRAMEWORK_ROOT}/skills/review-checklist.md` |
| `ai-dlc/guidelines/` (all files) | `{NEW_FRAMEWORK_ROOT}/guidelines/` (same filenames) |
| `ai-dlc/Instructions2FDE.md` | `{NEW_FRAMEWORK_ROOT}/Instructions2FDE.md` |
| `ai-dlc/README.md` | `{NEW_FRAMEWORK_ROOT}/README.md` |

Only move a file if it exists at the source. If a file is absent (e.g. `forbidden-zones.md` only exists in mature projects), skip it without error.

### 2c — Move all operational data

Move all operational data. Preserve the complete folder structure, including any `archive/` subfolders. Do not modify file contents.

| Source (old) | Destination (new) |
|---|---|
| `ai-dlc/ops/inception/intents/` | `{NEW_FRAMEWORK_ROOT}/ops/inception/intents/` |
| `ai-dlc/ops/inception/elaborations/` | `{NEW_FRAMEWORK_ROOT}/ops/inception/elaborations/` |
| `ai-dlc/ops/inception/dependency-map.md` | `{NEW_FRAMEWORK_ROOT}/ops/inception/dependency-map.md` |
| `ai-dlc/ops/build/backlog.md` | `{NEW_FRAMEWORK_ROOT}/ops/build/backlog.md` |
| `ai-dlc/ops/build/units/` | `{NEW_FRAMEWORK_ROOT}/ops/build/units/` |
| `ai-dlc/ops/build/bolts/` | `{NEW_FRAMEWORK_ROOT}/ops/build/bolts/` |
| `ai-dlc/ops/operate/retros/` | `{NEW_FRAMEWORK_ROOT}/ops/operate/retros/` |
| `ai-dlc/ops/operate/incidents/` | `{NEW_FRAMEWORK_ROOT}/ops/operate/incidents/` |
| `ai-dlc/ops/operate/improvements/` | `{NEW_FRAMEWORK_ROOT}/ops/operate/improvements/` |

**Critical:** After moving ops files, verify that the file counts match. Report any discrepancy before proceeding.

### 2d — Refresh template skill files

Replace the old template skill files in `{NEW_FRAMEWORK_ROOT}/skills/` with the latest versions from `process-onboarding-agent/skills/`. Copy each file verbatim.

Files to refresh:

| Source (new base) | Destination |
|---|---|
| `process-onboarding-agent/skills/compact-docs.md` | `{NEW_FRAMEWORK_ROOT}/skills/compact-docs.md` |
| `process-onboarding-agent/skills/root-cause-analysis.md` | `{NEW_FRAMEWORK_ROOT}/skills/root-cause-analysis.md` |
| `process-onboarding-agent/skills/solution-shaping.md` | `{NEW_FRAMEWORK_ROOT}/skills/solution-shaping.md` |
| `process-onboarding-agent/skills/design-session.md` | `{NEW_FRAMEWORK_ROOT}/skills/design-session.md` |
| `process-onboarding-agent/skills/bolt-risk-assessment.md` | `{NEW_FRAMEWORK_ROOT}/skills/bolt-risk-assessment.md` |
| `process-onboarding-agent/skills/progress-digest.md` | `{NEW_FRAMEWORK_ROOT}/skills/progress-digest.md` |
| `process-onboarding-agent/skills/uat.md` | `{NEW_FRAMEWORK_ROOT}/skills/uat.md` |
| `process-onboarding-agent/skills/release-readiness-checklist.md` | `{NEW_FRAMEWORK_ROOT}/skills/release-readiness-checklist.md` |
| `process-onboarding-agent/skills/process-health.md` | `{NEW_FRAMEWORK_ROOT}/skills/process-health.md` |
| `process-onboarding-agent/skills/dependency-audit.md` | `{NEW_FRAMEWORK_ROOT}/skills/dependency-audit.md` |
| `process-onboarding-agent/skills/knowledge-promotion.md` | `{NEW_FRAMEWORK_ROOT}/skills/knowledge-promotion.md` |
| `process-onboarding-agent/skills/ai-hub-metrics.md` | `{NEW_FRAMEWORK_ROOT}/skills/ai-hub-metrics.md` |
| `process-onboarding-agent/skills/process-visualization.md` | `{NEW_FRAMEWORK_ROOT}/skills/process-visualization.md` |
| `process-onboarding-agent/skills/notifications.md` | `{NEW_FRAMEWORK_ROOT}/skills/notifications.md` |
| `process-onboarding-agent/skills/new-engineer-induction.md` | `{NEW_FRAMEWORK_ROOT}/skills/new-engineer-induction.md` |
| `process-onboarding-agent/skills/bug-bolt.md` | `{NEW_FRAMEWORK_ROOT}/skills/bug-bolt.md` |
| `process-onboarding-agent/skills/hotfix-bolt.md` | `{NEW_FRAMEWORK_ROOT}/skills/hotfix-bolt.md` |
| `process-onboarding-agent/skills/nfr-bolt.md` | `{NEW_FRAMEWORK_ROOT}/skills/nfr-bolt.md` |

The table above lists the pre-built skills as of this guide's last update. Treat it as a checklist, not a closed set: refresh **every** `*.md` under `process-onboarding-agent/skills/` in the new base repo. If the base repo contains a skill the table does not mention, copy it across anyway and note it in the migration report — the base repo is the source of truth, not this list.

**Skills that need a setup conversation, not just a file copy.** Copying a skill file installs the *file*, not the configuration. After refreshing, check for skills whose behaviour is driven by a master rule file section that a migrated project will not have. Currently that means two skills, each gated on its own section being present:

- **notifications** — if the project's master rule file has no Section 10, offer to run the *Onboarding setup* steps inside `{NEW_FRAMEWORK_ROOT}/skills/notifications.md` and write Section 10 from the answers. If the engineer declines, write Section 10 with `Status: Disabled` — or leave it absent, which is treated as disabled — and note it in the migration report so it is a visible choice rather than an oversight.
- **ai-hub-metrics** — if the project's master rule file has no Section 11, this project predates the skill entirely (it is a newer addition to the base repo than notifications). Offer to run the *Onboarding setup* steps inside `{NEW_FRAMEWORK_ROOT}/skills/ai-hub-metrics.md` and write Section 11 from the answers — the skill's own setup ends by showing the engineer a status summary card, which doubles as their confirmation that the integration is live. If the engineer declines or is not ready (it needs a Workflow modeled in AI Hub first), write Section 11 with `Status: Disabled` — or leave it absent — and note it in the migration report.

Both sections are project-level switchboards (whether events fire at all), not project-specific data, so writing them from scratch during migration is correct — there is nothing in the old structure to preserve for either. They differ in who the credential belongs to: notifications is per-engineer (each teammate sets their own `SLACK_WEBHOOK_URL`, so Section 10 being `Enabled` gets a migrated teammate nothing until they personally configure it), while ai-hub-metrics is project-wide (one shared credential covers everyone once Section 11 is `Enabled`). Also add the Section 6 routing line for each skill if it is missing from the migrated master rule file (compare against the templates in `process-onboarding-agent/setup-guide.md` Section 6) — an old master rule file predating a skill will not have its routing line, and a copied skill file with no routing line never fires.

Also refresh the engagement rule:

| Source (new base) | Destination |
|---|---|
| `process-onboarding-agent/rules/engagement.md` | `{NEW_FRAMEWORK_ROOT}/rules/engagement.md` |

### 2e — Refresh ops templates

Update the `_template.md` files in the ops folder with the latest versions. Do not touch any non-template files (operational records).

| Source (new base) | Destination |
|---|---|
| `process-onboarding-agent/ops/inception/intents/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/inception/intents/_template.md` |
| `process-onboarding-agent/ops/inception/elaborations/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/inception/elaborations/_template.md` |
| `process-onboarding-agent/ops/build/units/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/build/units/_template.md` |
| `process-onboarding-agent/ops/build/bolts/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/build/bolts/_template.md` |
| `process-onboarding-agent/ops/operate/retros/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/operate/retros/_template.md` |
| `process-onboarding-agent/ops/operate/incidents/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/operate/incidents/_template.md` |
| `process-onboarding-agent/ops/operate/improvements/_template.md` | `{NEW_FRAMEWORK_ROOT}/ops/operate/improvements/_template.md` |
| `process-onboarding-agent/ops/inception/dependency-map.md` | Only if no dependency-map.md was moved in 2c |

---

## Step 3 — Migrate the Experience Agent

**What this step does:** Updates every path reference in the master rule file from the old `ai-dlc/` paths to the new `{NEW_FRAMEWORK_ROOT}` paths.

### 3a — Read the master rule file

Read the full master rule file (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`). Identify every line that references an `ai-dlc/` path.

Patterns to find and replace:

| Old pattern | New pattern |
|---|---|
| `ai-dlc/rules/` | `{NEW_FRAMEWORK_ROOT}/rules/` |
| `ai-dlc/skills/` | `{NEW_FRAMEWORK_ROOT}/skills/` |
| `ai-dlc/guidelines/` | `{NEW_FRAMEWORK_ROOT}/guidelines/` |
| `ai-dlc/ops/` | `{NEW_FRAMEWORK_ROOT}/ops/` |
| `ai-dlc/Instructions2FDE.md` | `{NEW_FRAMEWORK_ROOT}/Instructions2FDE.md` |
| `../ai-dlc/` (GitHub Copilot variant) | `../{NEW_FRAMEWORK_ROOT}/` |

Replace all occurrences throughout the file. **Do not modify any other content** — only the path strings.

### 3b — Verify the update

After writing the updated master rule file, count how many path references were changed and report the count to the engineer. Scan once more to confirm no `ai-dlc/` references remain.

---

## Step 4 — Migrate the Diagnostic Agent

**What this step does:** The new `process-diagnostic-agent/` folder (already copied to the project root) replaces `ai-dlc-reviewer/`. No content from `ai-dlc-reviewer/` needs to be preserved — it was entirely template-based.

There is nothing to move. The new `process-diagnostic-agent/` folder is already present from the prerequisite copy step.

The key capability addition is `process-diagnostic-agent/skills/trend-analysis.md` — this enables the diagnostic agent to compare reviews over time and report on practice trends. The `role-play.md` in the new agent already calls this skill in Step 3.

Confirm `process-diagnostic-agent/skills/trend-analysis.md` exists. If it is absent, the prerequisite copy was incomplete — alert the engineer before proceeding.

---

## Step 5 — Clean Up

**Before removing any old folder, verify:**
1. All generated files from Step 2b are confirmed present at their new location.
2. All operational data from Step 2c is confirmed present at the new location, with matching file counts.
3. The master rule file has been updated and contains no remaining `ai-dlc/` references.

Only after all three conditions are met, proceed with cleanup:

1. Remove `ai-dlc/` from the project root.
2. Remove `ai-dlc-reviewer/` from the project root (if present).

If any condition is unmet, stop and report to the engineer before removing anything.

---

## Step 6 — Deliver the Migration Report

Present a report in this format:

```
AI-DLC Migration Report
──────────────────────────────────────────────────────
Migration completed: [date]

AGENT 1 — ONBOARDING AGENT
  Framework relocated to: {NEW_FRAMEWORK_ROOT}
  Generated files moved:  [list each file]
  Operational data moved: [N] intents, [N] units, [N] bolts, [N] retros, ...
  Template skills refreshed: [list each skill]
  Notifications (Section 10): Enabled / Disabled — [set up this session / already configured / declined]
  AI Hub Metrics (Section 11): Enabled / Disabled — [set up this session / already configured / declined]
  Old bootstrap removed:  ai-dlc/ ✓

AGENT 2 — EXPERIENCE AGENT
  Master rule file updated: [filename]
  Path references changed: [N]
  No remaining ai-dlc/ references: ✓

AGENT 3 — DIAGNOSTIC AGENT
  New agent installed: process-diagnostic-agent/ ✓
  Trend Analysis capability: present ✓
  Old diagnostic removed: ai-dlc-reviewer/ ✓

ITEMS REQUIRING YOUR REVIEW
  [List any files or sections that need engineer attention]
  [If none: "None — migration is complete and ready to use."]

NEXT STEPS
  1. Open a new session in this repository. Your AI tool will load the master rule
     file automatically — the experience agent is live.
  2. To run the diagnostic agent: say "Read process-diagnostic-agent/role-play.md
     and follow the instructions inside it."
  3. If you ever need to re-run onboarding for a new AI tool: copy
     process-onboarding-agent/ from the AI-DLC base repo and follow onboard.md.
──────────────────────────────────────────────────────
```

After presenting the report, add this note:

> "The migration is complete. Close this session and open a new one — your master rule file will load automatically and the experience agent will be ready. The `process-migration-agent/` folder can be deleted from your project root; it is no longer needed."

---

## Edge Cases

### The engineer already deleted `ai-dlc/` after the initial onboarding

Some engineers follow the old guidance (which did not recommend deletion) and some may have deleted it anyway. If `ai-dlc/` is absent but the master rule file still references `ai-dlc/` paths:

1. Check whether the installed framework files are at a non-standard location. Ask the engineer: "I can see the master rule file but `ai-dlc/` is absent. Do you know where your rules and skills files are currently stored?"
2. If found: treat that location as the source for the move.
3. If genuinely absent: the template skill files can be recovered from `process-onboarding-agent/skills/`. The generated files (rules, guidelines, mob-elab-prompts, review-checklist) cannot be auto-recovered — flag these individually as "requires engineer to recreate".

### The `ai-dlc/` folder is present but `ai-dlc-reviewer/` is absent

This is normal — some teams never used the diagnostic agent. Skip Step 4's cleanup for `ai-dlc-reviewer/` and note in the report that `process-diagnostic-agent/` has been added as a new capability.

### The engineer has already moved some files manually

Check for any existing `intent-execution-framework/` folder. If found:
1. Report what is already there.
2. Do not overwrite files that are already in place.
3. Only move files that are still at their old `ai-dlc/` location.
4. Flag any conflicts to the engineer.

### The master rule file uses `../ai-dlc/` paths (GitHub Copilot variant)

The Copilot master rule file lives at `.github/copilot-instructions.md` and uses `../ai-dlc/` as the path prefix. Replace with `../{NEW_FRAMEWORK_ROOT}/` (e.g. `../docs/process/intent-execution-framework/`).

### ops/build/backlog.md contains Reference Link Registry entries with old paths

The backlog file uses reference-style Markdown links that include file paths. After moving ops data, scan `{NEW_FRAMEWORK_ROOT}/ops/build/backlog.md` for any `ai-dlc/ops/` path references in the Reference Link Registry section and update them to `{NEW_FRAMEWORK_ROOT}/ops/`.
