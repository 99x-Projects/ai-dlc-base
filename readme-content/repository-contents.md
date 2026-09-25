# Repository Contents Reference

← [Back to README](../README.md)

---

## What Is This Repository?

This repo is the **base template** — the source of truth that gets copied into each project repo. It contains:

| File / Folder | Purpose |
|---|---|
| `repository-agents/process-onboarding-agent/setup-guide.md` | The complete framework specification. Describes every file, rule, and ceremony. |
| `repository-agents/process-onboarding-agent/onboard.md` | The bootstrap trigger. Drop this into a target repo and follow the instructions to start onboarding. |
| `repository-agents/process-onboarding-agent/skills/compact-docs.md` | Engineer-triggered skill to archive operational documents older than the project's configured threshold. |
| `repository-agents/process-onboarding-agent/skills/root-cause-analysis.md` | Skill to run 5-Whys analysis on incidents and improvements, classify root causes (Solution Design / Technology / Process), surface cross-cutting patterns, and produce linked intent or improvement artifacts. |
| `repository-agents/process-onboarding-agent/skills/solution-shaping.md` | Pre-elaboration solution shaping. Decides generic vs feature-specific scope, expected usage, the simplest viable approach, extend-vs-build-vs-buy, and reversibility before any design work; records the signed-off shape on the intent as binding context for the design session. |
| `repository-agents/process-onboarding-agent/skills/design-session.md` | Phase 0 of mob elaboration. Runs at the start of every elaboration session to lock down API contracts, data models, and architectural patterns before unit decomposition begins. |
| `repository-agents/process-onboarding-agent/skills/bolt-risk-assessment.md` | Pre-bolt risk assessment. Analyzes blast radius, cross-unit sequencing risks, rollback options, and feature flag requirements before the first unit executes. Mandatory for mature projects. |
| `repository-agents/process-onboarding-agent/skills/product-engineering-essentials.md` | Optional ten-pillar checklist of product/engineering foundations (vision, domain understanding, requirements, UX, architecture, engineering practices, DevOps, quality engineering, security/compliance, delivery feedback loop). Reads the actual repository for evidence per item and reports In place / Partial / Missing / N/A — never a single score. Engineer-invoked only, never a gate. |
| `repository-agents/process-onboarding-agent/skills/progress-digest.md` | Stakeholder communication artifact. Translates technical progress (units, bolts, statuses) into plain-language summaries for non-technical stakeholders. |
| `repository-agents/process-onboarding-agent/skills/uat.md` | Acceptance testing protocol. Guides the engineer through UAT using the intent's ACs as the test script; records pass/fail/deferred outcomes; blocks intent from being marked Implemented without sign-off. |
| `repository-agents/process-onboarding-agent/skills/release-readiness-checklist.md` | Self-attestation UAT/production release checklist. Walks the engineer through the checklist one item at a time (Confirmed / Not confirmed / N/A), records a blocker note for anything not confirmed, and produces a readiness report. Never inspects the repo for evidence and never gates the release — that decision stays with whoever holds sign-off. Engineer-invoked only. |
| `repository-agents/process-onboarding-agent/skills/process-health.md` | Process health metrics. Computes four metrics (improvement adoption, quality gate failure rate, AC revision rate, bolt velocity) and surfaces decay signals; saves a dated health report automatically. |
| `repository-agents/process-onboarding-agent/skills/dependency-audit.md` | Monthly dependency and security posture audit. Reads manifests, classifies findings by severity, and creates remediation bolts for high/critical issues. Scheduled via Section 9 of the master rule file. |
| `repository-agents/process-onboarding-agent/skills/knowledge-promotion.md` | Cross-project learning protocol. Runs as the final step of every retro; classifies each improvement as generic (to be contributed back to this base repo) or project-specific. |
| `repository-agents/process-onboarding-agent/skills/notifications.md` | Slack notifications for delivery moments that need a human. A local, per-engineer setup — each teammate creates their own channel and webhook on their own machine; nothing is shared. Two layers: deterministic tool hooks (Claude Code, Cursor and Copilot) that fire whenever the agent stops or needs attention, and agent-driven, event-specific alerts (bolt complete, UAT sign-off, incident, circuit breaker, audit due) for any tool that can run a terminal command. Sends go through a tool-neutral `scripts/notify.sh`; the webhook URL comes from the `SLACK_WEBHOOK_URL` env var — no webhook URL is ever committed. |
| `repository-agents/process-onboarding-agent/skills/ai-hub-metrics.md` | Pushes delivery activity events to 99x AI Hub so AI-assisted work is visible in one dashboard. Project-level, not per engineer: one Team, one Workflow, and one shared credential cover the whole project. Fires at unit Done, bolt complete, UAT sign-off, and intent Implemented; optional `tokens`/`costUsd`/`model` dimensions are passed only when the project has a real source for them. Endpoint, credential, and workflow IDs come from a gitignored `scripts/ai-hub.env` — nothing is ever committed. |
| `repository-agents/process-onboarding-agent/skills/process-visualization.md` | Retro-time delivery reconstruction. Mines git history (or falls back to recorded artifact dates) to render Mermaid diagrams of how a bolt actually got delivered, plus a plan-vs-actual deviation table; offered at the start of every retro. |
| `repository-agents/process-onboarding-agent/skills/new-engineer-induction.md` | New engineer onboarding session. Walks a new team member through the project's framework using actual project files; prompts them to set up notifications on their own machine if the project uses them; produces a personalized quick-reference card. |
| `repository-agents/process-onboarding-agent/skills/bug-bolt.md` | Lightweight bolt workflow for fixing a specific, reproducible bug. Triggered by "fix a bug in X". Skips design session and elaboration; replaces them with a four-question intake, recurrence check, and a single focused unit. Runs RCA automatically if the bug is recurring. |
| `repository-agents/process-onboarding-agent/skills/hotfix-bolt.md` | Emergency bolt for production incidents. Triggered by "hotfix" or "prod is down". Runs a three-question intake (symptom, severity, rollback), creates a minimal unit, mandates a retro and RCA within 24 hours. |
| `repository-agents/process-onboarding-agent/skills/nfr-bolt.md` | Non-functional quality attribute bolt. Triggered by "improve performance of X", "NFR bolt for X", etc. Requires a measurable threshold AC, establishes a before/after baseline, and cross-references affected intents. Does not create a new intent. |
| `repository-agents/process-onboarding-agent/ops/inception/dependency-map.md` | Intent dependency map. Records which intents depend on others and which interfaces are shared across intent boundaries; read before bolt planning, updated after every elaboration sign-off. |
| `repository-agents/process-onboarding-agent/rules/engagement.md` | Engineer engagement monitoring — signals of disengagement, intervention script, and escalation protocol. Copied verbatim into every project. |
| `repository-agents/process-onboarding-agent/ops/inception/intents/_template.md` | Template for writing a feature intent (includes Implementation Summary section, written when all units under the intent are delivered) |
| `repository-agents/process-onboarding-agent/ops/inception/elaborations/_template.md` | Template for logging a mob elaboration session |
| `repository-agents/process-onboarding-agent/ops/build/units/_template.md` | Template for an atomic unit of work |
| `repository-agents/process-onboarding-agent/ops/build/bolts/_template.md` | Template for a planned batch of units |
| `repository-agents/process-onboarding-agent/ops/build/backlog.md` | Starter backlog file with Reference Link Registry — all unit/bolt links use reference-style Markdown so compact-docs only updates the registry, never the table rows |
| `repository-agents/process-onboarding-agent/ops/operate/retros/_template.md` | Template for a bolt retrospective (includes Post-Retro Improvement Workflow — AI-driven, runs immediately after every retro) |
| `repository-agents/process-onboarding-agent/ops/operate/incidents/_template.md` | Template for a production incident |
| `repository-agents/process-onboarding-agent/ops/operate/improvements/_template.md` | Template for a process improvement triggered by a retro or incident |

The remaining files (most of `rules/`, all of `guidelines/`, and the master rule file) are **generated per project** by the AI agent during onboarding — they cannot be shared across projects because they encode each project's specific stack, domain, and conventions.

---

## What the Agent Creates in Your Project Repo

The onboarding agent first asks where your process documentation lives, then installs the framework at `{your-docs-folder}/intent-execution-framework/` (or `docs/process/intent-execution-framework/` if no folder exists yet). After onboarding, your target repo will contain:

```
<your-project>/
  CLAUDE.md (or .cursor/rules/project-rules.mdc / .github/copilot-instructions.md)
  process-onboarding-agent/        ← delete this after onboarding is complete
    onboard.md
    setup-guide.md
    rules/  skills/  ops/          ← source files used during onboarding only
  {your-docs-folder}/              ← e.g. docs/process/ or an existing docs folder
    intent-execution-framework/    ← the installed framework lives here
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
        solution-shaping.md
        design-session.md
        bolt-risk-assessment.md
        product-engineering-essentials.md
        progress-digest.md
        uat.md
        release-readiness-checklist.md
        process-health.md
        dependency-audit.md
        knowledge-promotion.md
        notifications.md
        ai-hub-metrics.md
        process-visualization.md
        new-engineer-induction.md
        bug-bolt.md
        hotfix-bolt.md
        nfr-bolt.md
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
        inception/dependency-map.md
        build/backlog.md
        build/units/
        build/bolts/
        operate/retros/
        operate/incidents/
        operate/improvements/
      Instructions2FDE.md
      README.md
```
