# Diagnostics and Review Guide

← [Back to README](../README.md)

---

## AI-DLC Reviewer (Diagnostics)

The `repository-agents/process-diagnostic-agent/` folder provides a **pull** mode audit that works at any stage — whether the project is fully onboarded, partially adopted, or has no AI-DLC installation at all. The agent works with whatever process artifacts the team already has and scores them against AI-DLC principles, making it equally useful as a pre-onboarding readiness assessment or a periodic health check on a running project.

| File | Purpose |
|---|---|
| `repository-agents/process-diagnostic-agent/role-play.md` | Bootstrap trigger. Copy into the target repo and read it to the AI to start a review session. |
| `repository-agents/process-diagnostic-agent/review-guide.md` | Full review protocol — eight domains, rubrics for every artifact type, scoring system, and report format. |

---

## How to Use

1. Copy the `process-diagnostic-agent/` folder from `repository-agents/` in this repo into the root of the project repo being audited.
2. Open your AI assistant inside that repo.
3. Say: `"Read process-diagnostic-agent/role-play.md and follow the instructions inside it."`
4. The AI adopts the reviewer persona and runs through the selected review domains — requesting artifacts or conducting conversations per domain, scoring against rubrics, and delivering a comprehensive report with gap analysis, a People-Process-Tools Alignment section, and a prioritised remediation plan.

The reviewer never asks for files by specific path — the engineer shares whatever they have from wherever they keep it. For teams without AI-DLC installed, that means sharing current-process artifacts (tickets, PRs, retro notes) and the agent scores those against AI-DLC principles. The resulting gap report is a practical brief for an onboarding session.

The reviewer never generates code or creates files unprompted. It is a diagnostic tool, not an onboarding tool.

---

## Review Scope

The session begins with a scope selection step. Process review always runs; People and Tools are optional.

**Always included (6 domains):**
1. **Foundation** — master rule file structure and completeness
2. **Inception** — intent and elaboration session quality
3. **Build** — unit and bolt file quality
4. **Operate** — retro, improvement, and RCA discipline
5. **Process Adherence** — quality gate enforcement and review usage, conversation-based
6. **Organization & Structure** — assessed passively from the locations you provide artifacts from

**Optional (selected at session start):**
7. **People** — FDE skills assessment across five capability areas, conversation-based. Choose one team member or several — the agent assesses each individually and compiles a team aggregate.
8. **Tools** — SDLC automation posture across nine pipeline stages, conversation-based; produces a classified automation tier table with gap challenges and suggestions.

---

## Review Output

The agent delivers a comprehensive Review Report written to a dated file in your `diagnostics-reports/` folder. The report contains:

- **Review Scorecard** — domain-by-domain scores (Aligned / Partially Aligned / Not Aligned / Skipped)
- **Findings** — every finding by severity (Critical → Important → Advisory), each with the observed gap, the principle it violates, and the specific change recommended
- **Organization Assessment** — scored across five dimensions
- **People-Process-Tools Alignment** — FDE skills profile (People), process adherence summary (Process), and SDLC automation posture table (Tools); skipped sections noted
- **Gap Analysis** — table mapping each finding to the AI-DLC principle it violates, with current state, target state, and severity
- **Trend Analysis** — domain score timeline across all past reviews, finding classifications (New / Recurring / Resolved / Worsened), and a trajectory summary (or first-review baseline if no prior reports exist)
- **Remediation Plan** — Immediate / Short-term / Long-term actions (produced only when Critical or Important findings exist), each with the exact change required and a verification signal
- **First Action** — the single highest-impact change from the Immediate tier
- **Patterns Worth Monitoring** — recurring themes that did not reach Critical severity but signal process drift

---

## Running a Pre-Onboarding Assessment

The diagnostics agent works equally well before AI-DLC is installed. Share your current process artifacts — Jira tickets, PR descriptions, retro notes, incident records, or anything that represents how your team currently works — and the agent scores them against AI-DLC principles.

The resulting gap report tells you exactly what is missing and why, and becomes the brief for your onboarding session. It is a faster path to a high-quality onboarding than starting from scratch.

You do not need to copy `process-onboarding-agent/` for this — the diagnostics agent never reads from it.
