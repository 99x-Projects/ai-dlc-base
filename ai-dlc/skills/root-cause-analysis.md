# Skill: Root Cause Analysis

**Purpose:** Analyse resolved incidents and filed improvements to surface deeper root causes beyond the immediate fix. Identifies structural gaps in solution design, technology selection, and process that make the system or team vulnerable to a class of failures — not just the specific instance that was fixed.

**Trigger:** Engineer-initiated, or triggered by the AI agent when:
- An incident is marked Resolved and the engineer has not already run this skill on it
- The Post-Retro Improvement Workflow surfaces a pattern that points to a systemic issue
- The engineer says something like "run root-cause-analysis on [incident/improvement]" or "analyse what's behind these recurring failures"

**Never run automatically.** The agent may suggest running this skill, but must wait for the engineer to confirm before starting.

---

## Step 1 — Define the Scope

Ask the engineer:

> "Should I analyse a single incident or improvement file, or look across multiple to surface patterns? If multiple, tell me which files or which time range to cover."

Read each file in scope fully before proceeding. Do not begin analysis on partial information.

---

## Step 2 — Extract Surface Findings

For each file in scope, extract:
- **Symptom** — what broke or what gap was identified
- **Immediate fix** — what was done to resolve it
- **Improvement filed** — what rule/guideline change was triggered (if any)

Present a summary table before proceeding:

```
| File | Symptom | Immediate fix | Improvement filed |
```

Confirm with the engineer that the surface findings are accurate before moving to root cause analysis.

---

## Step 3 — Apply 5-Whys Analysis

For each distinct failure, build a Why chain. Start from the symptom and ask "Why did this happen?" of each answer until you reach a cause that cannot be explained by another "because" within the team's control — typically 3–5 levels deep.

Document each chain explicitly:

```
Symptom: [what broke]
  Why 1: [symptom] happened because [cause 1]
  Why 2: [cause 1] happened because [cause 2]
  Why 3: [cause 2] happened because [root cause]
Root cause: [statement]
```

Stop the chain when you reach one of the three root cause categories defined in Step 4. Do not continue past the team's control boundary (e.g. "the cloud provider had an outage" is a boundary; "we had no fallback when the provider failed" is not).

---

## Step 4 — Classify Root Causes

Assign each root cause to one or more of the following categories:

| Category | What it means | Example signals |
|---|---|---|
| **Solution Design** | The architecture, data model, boundary, or feature design made this class of failure structurally likely | Missing validation layer, wrong responsibility boundary, tightly coupled modules, missing idempotency, incorrect API contract assumed |
| **Technology Selection** | A chosen technology has fundamental characteristics that contributed to the failure and were not accounted for | ORM behaviour under concurrent load, framework routing edge cases, third-party API rate limits or reliability, library version constraint conflicts |
| **Process** | A gap in the team's workflow, ceremonies, or standards allowed the failure to be introduced or reach production | Missing AC type, quality gate not enforced, review step skipped, test coverage blind spot, no rollback plan, unclear ownership |

A single root cause may span more than one category — record all that apply.

---

## Step 5 — Pattern Analysis (multi-file scope only)

If analysing more than one file, group root causes across all chains and identify:
- Root causes that appear in more than one incident or improvement (recurring vulnerability)
- Root cause categories that dominate (signals a systemic weakness in design, technology choices, or process)
- Failures where the same root cause produced different surface symptoms (masked recurrence)

Present a pattern table:

```
| Root cause | Category | Files affected | Risk if unaddressed |
```

Flag any pattern where the same root cause has already produced two or more distinct failures — these are the highest priority recommendations.

---

## Step 6 — Produce the RCA Report

Write a structured report containing:

1. **Scope** — files analysed, date range covered
2. **Surface findings** — table from Step 2
3. **Root cause chains** — one Why chain per distinct failure
4. **Classification** — root causes grouped by category (Solution Design / Technology / Process)
5. **Patterns** — cross-cutting vulnerabilities if multi-file scope; write "Single file — no pattern analysis" otherwise
6. **Recommendations** — for each root cause, one of three types:

| Recommendation type | What it means | Artifact to create |
|---|---|---|
| **Redesign** | A design or architectural decision must change | New intent file; update `ai-dlc/rules/architecture.md` with an ADR |
| **Technology mitigation** | A technology limitation must be documented and worked around | ADR in `ai-dlc/rules/architecture.md`; new intent if a replacement is warranted |
| **Process fix** | A workflow, gate, or standard must change | Improvement file via the standard Post-Retro Improvement Workflow |

Present the report to the engineer and get explicit approval on each recommendation before creating any artifact.

---

## Step 7 — Create Linked Artifacts

For each approved recommendation:

**Redesign or technology mitigation:**
1. Create a new intent file at `ai-dlc/ops/inception/intents/YYYY-MM-DD-rca-[slug].md`
2. Populate the "Why" section with the root cause statement from Step 3
3. Populate "What" with the recommended design or technology change
4. Add to the Open section of `ai-dlc/ops/build/backlog.md` using a reference-style link

**Process fix:**
1. Create an improvement file at `ai-dlc/ops/operate/improvements/YYYY-MM-DD-rca-[slug].md`
2. Apply the change to the target file immediately (`rules/`, `skills/`, or `guidelines/`)
3. Mark the improvement Applied

**For all artifacts:**
- Add a link back to the RCA report in the source incident or improvement file under a new "RCA" field
- If a root cause spans multiple source files, link the RCA report from each of them

---

## Step 8 — Report

Present a completion summary:

```
Root Cause Analysis complete.

Files analysed: [N]
Root cause chains identified: [N]
  - Solution Design: [N]
  - Technology Selection: [N]
  - Process: [N]

Recommendations approved: [N]
  - New intent files created: [list]
  - Improvement files created and applied: [list]
  - ADRs added to architecture.md: [N]

Patterns flagged for monitoring: [N] (recurring root causes)
```
