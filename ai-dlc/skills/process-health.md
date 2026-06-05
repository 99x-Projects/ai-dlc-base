# Skill: Process Health

**Purpose:** Analyse the project's AI-DLC operational artifacts and produce a quantitative report on how well the process is functioning. Surfaces improvement adoption gaps, recurring quality gate failures, AC quality drift, and bolt velocity trends — the signals that indicate process decay before it becomes visible in code quality.

**Trigger:** Engineer-initiated at any time. Invoked by saying something like "run process health", "how is our AI-DLC process doing?", or "generate a process health report."

**Never run automatically.** The engineer decides when to run this check. Recommended cadence: after every third or fourth bolt, or whenever the team suspects the process has drifted.

---

## Step 1 — Define the Scope

Ask the engineer:

> "Should I analyse all artifacts in the project, or limit the report to a specific time range? (e.g. 'last 3 months', 'since [date]', or 'all time')"

Record the answer. If the engineer says "all time" or does not specify, analyse everything in `ai-dlc/ops/`.

Then read the following without further prompting — do not ask for files individually:

| Artifact type | Location | What to extract |
|---|---|---|
| Retro files | `ai-dlc/ops/operate/retros/` | Date, bolt link, Improvements Triggered table, AI-Specific Observations section, What Didn't Go Well section |
| Improvement files | `ai-dlc/ops/operate/improvements/` | Status, Triggered by, Applied date |
| Bolt files | `ai-dlc/ops/build/bolts/` | Status, Start date, Target date, Completed date, Retrospective link |
| Intent files | `ai-dlc/ops/inception/intents/` | Status, Date, UAT Sign-off status |

Read all files within scope before calculating any metric. Do not calculate or report while still reading.

---

## Step 2 — Calculate Metric 1: Improvement Adoption Rate

**What it measures:** Whether improvements filed after retros are actually being applied to the target files, or are sitting Open and having no effect on future bolts.

**Source:** All improvement files in scope.

**Calculation:**

1. Count all improvement files by status:
   - `Applied` — improvement was completed
   - `Open` — improvement was filed but not yet applied
   - `Rejected` — engineer deliberately chose not to apply

2. Adoption rate = (Applied) ÷ (Applied + Open) × 100
   *Exclude Rejected from both numerator and denominator — a deliberate rejection is not a gap.*

3. For each Open improvement, calculate how many days it has been open (today's date minus the date in the file name or Triggered by link date).

4. Flag any improvement Open for more than 30 days as stale.

5. If more than one retro is in scope, calculate the trend: compare adoption rate for the first half of retros versus the second half. A declining rate signals worsening process discipline.

**Thresholds:**

| Rate | Rating |
|---|---|
| ≥ 80% | Healthy |
| 60 – 79% | Warning |
| < 60% | Critical |

**Stale improvements threshold:** Any single improvement Open > 30 days is a Warning signal regardless of the overall rate.

---

## Step 3 — Calculate Metric 2: Quality Gate Failure Rate

**What it measures:** How often the Prompt Quality Gate is failing — meaning the engineer is submitting incomplete prompts and the AI is having to stop and ask for missing context, constraints, ACs, or output format.

**Source:** The "Quality gate failures" subsection of "AI-Specific Observations" in every retro file in scope.

**Calculation:**

1. For each retro, check the "Quality gate failures" subsection. A retro counts as having a QG failure if the subsection contains any entry other than "—", a blank line, or an explicit "None."

2. QG failure rate = retros with at least one QG failure ÷ total retros in scope × 100.

3. List each distinct failure description (do not deduplicate across retros — recurring failures are a separate signal). Look for failures that appear in more than one retro: these are systematic gaps, not one-off mistakes.

4. Trend: if four or more retros are in scope, split them chronologically in half. Is the rate in the recent half lower (improving) or higher (worsening) than the earlier half?

**Thresholds:**

| Rate | Rating |
|---|---|
| ≤ 20% | Healthy |
| 21 – 40% | Warning |
| > 40% | Critical |

**Recurring failure signal:** Any specific QG failure type appearing in two or more retros is flagged as a recurring gap regardless of the overall rate.

---

## Step 4 — Calculate Metric 3: AC Revision Rate

**What it measures:** How often acceptance criteria are found to be wrong, incomplete, or untestable — either during the bolt or discovered in retro — requiring revision after elaboration sign-off.

**Source:** "What Didn't Go Well" and "Prompts that needed revision" subsection of retro files in scope.

**Signal detection:** Scan each retro for language indicating AC problems. Flag a retro as having AC issues if any of the following appear in the relevant sections:

- References to ACs being wrong, incomplete, changed, or missing after elaboration
- References to acceptance criteria not covering a scenario that was discovered during build
- References to ACs being untestable or too vague to implement against
- References to a unit requiring rework because the original criteria were misunderstood

Do not flag retros where prompt revision was purely about context or output format — only flag AC-specific findings.

**Calculation:**

1. AC revision rate = retros with AC issues ÷ total retros in scope × 100.

2. Note any AC issues that appear across multiple retros: these are patterns, not one-off incidents. Common recurring patterns include: missing unhappy-path ACs, ACs written at the wrong level of abstraction, or ACs that mix behavior and implementation.

3. Trend: same split as Metric 2 — is the rate improving or worsening in recent retros?

**Thresholds:**

| Rate | Rating |
|---|---|
| ≤ 15% | Healthy |
| 16 – 30% | Warning |
| > 30% | Critical |

---

## Step 5 — Calculate Metric 4: Bolt Velocity Trend

**What it measures:** Whether bolts are consistently completing on time, running late, or accelerating — and whether the trend is improving or worsening.

**Source:** All bolt files in scope with status Done and a Completed date recorded.

**Calculation:**

1. For each completed bolt, calculate the variance:
   - Variance = Completed date − Target date (in calendar days)
   - Positive variance = bolt completed late
   - Negative variance = bolt completed early
   - Zero = on time

2. Compute the average variance across all completed bolts in scope.

3. Identify the distribution:
   - How many bolts completed on time (variance −2 to +2 days)?
   - How many completed early (variance < −2)?
   - How many completed late (variance > +2)?

4. Trend: if four or more completed bolts are in scope, compare the average variance of the first half versus the second half. Increasing average variance = worsening delivery predictability.

5. Flag any bolt that overran by more than 50% of its planned duration — these are outliers that warrant a separate note.

**Thresholds (average variance):**

| Average variance | Rating |
|---|---|
| −2 to +3 days | Healthy |
| +4 to +7 days | Warning |
| > +7 days, or trend worsening | Critical |

If fewer than two completed bolts are in scope, write: "Insufficient data — fewer than two completed bolts in scope. Run again after more bolts complete."

---

## Step 6 — Identify Additional Decay Signals

After calculating the four metrics, scan the artifacts for the following additional signals. Each one found is noted in the report as a decay signal.

| Signal | How to detect | Severity |
|---|---|---|
| Retro with no improvements and no stated reason | "Improvements Triggered" table is blank or contains "—" with no explanatory note | Warning |
| Bolt with no retro filed | Bolt status is Done but Retrospective field links to a non-existent file | Warning |
| Intent In Progress with no active bolt | Intent status is In Progress but no bolt links back to it and no unit under it is In Progress | Warning |
| Improvement Open > 30 days | Calculated in Metric 1 — list each stale improvement | Warning |
| UAT consistently deferred | More than half of Implemented intents have UAT status Deferred with no follow-up record | Warning |
| Same QG failure type in 3+ retros | Identified in Metric 2 — list the recurring failure | Critical |
| Same AC issue type in 3+ retros | Identified in Metric 3 — list the recurring pattern | Critical |

---

## Step 7 — Produce and Save the Report

Present the full report in the conversation, then write it to `ai-dlc/ops/operate/process-health-YYYY-MM-DD.md` automatically. Do not ask for permission — saving is mandatory so that health reports accumulate over time and trends can be compared across runs.

```
─────────────────────────────────────────────────────
AI-DLC Process Health Report
Project: [project name from master rule file]
Scope:   [time range or "All time"]
Date:    YYYY-MM-DD
Artifacts read: [N] retros, [N] improvements, [N] bolts, [N] intents
─────────────────────────────────────────────────────

METRIC 1 — Improvement Adoption Rate
  Rate:     [X]%  ([Applied] applied of [Applied+Open] filed; [Rejected] rejected)
  Rating:   [Healthy / Warning / Critical]
  Stale:    [N] improvements Open > 30 days
             [list: improvement title — open since YYYY-MM-DD]
  Trend:    [Improving / Stable / Worsening — or "Insufficient data"]

METRIC 2 — Quality Gate Failure Rate
  Rate:     [X]%  ([N] of [N] retros had at least one QG failure)
  Rating:   [Healthy / Warning / Critical]
  Recurring failures:
             [list recurring failure types, or "None"]
  Trend:    [Improving / Stable / Worsening — or "Insufficient data"]

METRIC 3 — AC Revision Rate
  Rate:     [X]%  ([N] of [N] retros flagged AC issues)
  Rating:   [Healthy / Warning / Critical]
  Recurring patterns:
             [list recurring AC issue types, or "None"]
  Trend:    [Improving / Stable / Worsening — or "Insufficient data"]

METRIC 4 — Bolt Velocity
  Completed bolts in scope: [N]
  Average variance:  [+/− X days]
  On time / Early / Late:  [N] / [N] / [N]
  Rating:   [Healthy / Warning / Critical]
  Outliers: [bolts that overran > 50% of planned duration, or "None"]
  Trend:    [Improving / Stable / Worsening — or "Insufficient data"]

─────────────────────────────────────────────────────
DECAY SIGNALS
[List each signal found, or "None detected"]

  ⚠ [signal description]
  ⚠ [signal description]

─────────────────────────────────────────────────────
OVERALL HEALTH

  [Healthy / Warning / Critical]

  Rules:
  - Healthy:  all four metrics Healthy, no Critical decay signals
  - Warning:  one or more metrics Warning, or one or more Warning decay signals
  - Critical: any metric Critical, or any Critical decay signal

─────────────────────────────────────────────────────
FIRST ACTION

[One sentence. The single most impactful thing the team should address first,
derived from the highest-severity finding. Examples:
"Apply the [N] stale Open improvements before the next bolt begins."
"Address the recurring quality gate failure '[type]' — it has appeared in [N] retros."
"Run retros consistently — [N] completed bolts have no retro on record."]
─────────────────────────────────────────────────────
```

After presenting the report, write it to `ai-dlc/ops/operate/process-health-YYYY-MM-DD.md` and confirm:

> "Report saved to `ai-dlc/ops/operate/process-health-YYYY-MM-DD.md`. Run this skill again after the next few bolts to track whether the health trend is improving."
