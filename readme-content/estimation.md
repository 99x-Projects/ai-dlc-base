# Estimation Guide

← [Back to README](../README.md)

---

## Overview

The Estimation Agent produces structured, AI-assisted delivery estimates at two different stages of a project.

| Mode | When to use | Input | Output |
|---|---|---|---|
| **Mode 1 — Ballpark** | Pre-sales or feasibility; requirements are brief or draft | Product description or draft requirements | Hour ranges per capability area, confidence level, launch window |
| **Mode 2 — Delivery** | After mob elaboration; units are defined | Elaborated units from the project backlog | Bolt-level estimates, release milestone map, calibration log |

---

## How to Use

**Ballpark (Mode 1):**
1. Copy `process-estimation-agent/` from `repository-agents/` into your working folder.
2. Open your AI assistant.
3. Say: `"Read process-estimation-agent/estimate.md and follow the instructions inside it."` — select **Mode 1** when prompted.
4. Paste or describe the requirements. The agent structures them into capability areas, classifies each by delivery tier, and produces a range estimate with confidence level and risk factors.
5. The agent writes `estimates/[project-slug]-ballpark-YYYY-MM-DD.md` to your working folder.

**Delivery (Mode 2):**
1. The 99x Intent Delivery Framework must already be installed and mob elaboration complete.
2. Copy `process-estimation-agent/` from `repository-agents/` into your project root.
3. Say: `"Read process-estimation-agent/estimate.md and follow the instructions inside it."` — select **Mode 2** when prompted.
4. The agent reads your backlog, classifies each unit by effort tier, sums estimates per bolt, and maps bolts to a release timeline.
5. The agent writes the estimate artifact to `{FRAMEWORK_ROOT}/ops/inception/estimates/`.

---

## The Tier Classification System

### Mode 1 — Capability-area tiers

| Tier | What it covers | Default range |
|---|---|---|
| **Tier A — AI-Accelerated** | CRUD, standard APIs, form-driven UI, dashboards, boilerplate, standard authentication, test generation | 8–20 hrs per capability area |
| **Tier B — Mixed** | Moderate domain logic, standard third-party integrations, multi-step flows, role-based access, data migration | 20–48 hrs per capability area |
| **Tier C — Human-Led** | Novel domain logic, custom algorithms, complex multi-system integrations, security-critical systems, real-time or high-performance requirements | 48–100 hrs per capability area |

Modifiers adjust the totals based on team AI experience, integration count, and whether the project is greenfield or an extension.

### Mode 2 — Unit-level tiers

| Tier | What it covers | Default range |
|---|---|---|
| **Simple** | 1–3 ACs, single-purpose, well-understood pattern, no external integration | 1–3 hrs |
| **Standard** | 3–6 ACs, one integration point or one layer of business logic | 3–6 hrs |
| **Complex** | 6+ ACs, multiple integration points, performance requirements, or intricate domain logic | 6–14 hrs |
| **Spike** | Technical investigation — no functional deliverable, produces a written recommendation or proof of concept | 2–5 hrs |

Overhead of 20% is added per bolt; a QA buffer of 10% of Complex unit hours is added if any Complex units are present.

---

## Calibration from Recorded Hours

After each bolt closes, the actual hours taken are recorded in the estimate artifact's log. Once three or more bolts have recorded hours, the agent computes a **calibration modifier** — the ratio of actual to estimated hours averaged across closed bolts — and applies it to all remaining open bolt estimates.

**To update after a bolt closes:**
> "Update the estimate for [Bolt name] — it took [X] actual hours."

The agent updates the log, recomputes the modifier, adjusts remaining bolt estimates if needed, and reports the revised projected completion date.

This means estimates improve in accuracy as the project progresses. The first estimate uses default tier ranges; by the third bolt, it reflects the team's actual AI-assisted delivery pace.

---

## How the Calculations Work

### Mode 1 — Ballpark calculation

**Step 1: Assign hours to each capability area**

Each capability area is placed into a tier. The tier defines a min/max range of hours. The agent always works with three numbers per area:

```
Min hours     = lower bound of the tier range
Expected hrs  = midpoint of the tier range
Max hours     = upper bound of the tier range
```

Examples using the default ranges:

| Tier | Min | Expected | Max |
|---|---|---|---|
| A (AI-Accelerated) | 8 hrs | 14 hrs | 20 hrs |
| B (Mixed) | 20 hrs | 34 hrs | 48 hrs |
| C (Human-Led) | 48 hrs | 74 hrs | 100 hrs |

**Step 2: Sum across all capability areas**

```
total_min      = sum of all min values
total_expected = sum of all expected values
total_max      = sum of all max values
```

**Step 3: Apply modifiers**

Modifiers are applied to the totals. Multiple modifiers stack — they are applied in sequence, not averaged:

```
New to AI-assisted development  → multiply all three totals by 1.3
AI-DLC experienced team         → multiply all three totals by 0.85
Extending an existing codebase  → add 20% to all three totals
4–10 external integrations      → add 15% to the Tier B and C subtotals only
10+ external integrations       → add 25% to the Tier B and C subtotals only
```

Example: A project with one Tier A area (20 hrs expected) and one Tier B area (34 hrs expected), extending an existing codebase, team new to AI:

```
Raw expected  = 20 + 34           = 54 hrs
Extension +20% = 54 × 1.20        = 64.8 hrs
New to AI ×1.3 = 64.8 × 1.30     = 84.2 hrs  ← final expected hours
```

**Step 4: Convert to delivery weeks**

The agent uses **28 productive AI-assisted hours per engineer per week** — this accounts for meetings, code reviews, retros, and context switching (not a full 40-hour week of coding).

```
weeks = total_hours ÷ (team_size × 28)
```

**Step 5: Apply scenario buffers**

Three delivery scenarios are always produced:

```
Earliest  = min_hours   ÷ (team_size × 28)         no buffer
Expected  = expected_hours × 1.10 ÷ (team_size × 28)   +10% buffer
Latest    = max_hours   × 1.25 ÷ (team_size × 28)   +25% buffer
```

The +10% and +25% buffers account for integration risk, unplanned scope clarifications, and the uncertainty inherent in draft requirements.

---

### Mode 2 — Delivery calculation

**Step 1: Assign expected hours to each unit**

Each unit is classified into a tier. The agent uses the **expected value** (midpoint of the range) unless a calibration modifier is active:

```
Simple expected   = 2 hrs   (midpoint of 1–3)
Standard expected = 4.5 hrs (midpoint of 3–6)
Complex expected  = 10 hrs  (midpoint of 6–14)
Spike expected    = 3.5 hrs (midpoint of 2–5)
```

If a calibration modifier exists (see below), the expected hours for each unit are adjusted:

```
adjusted expected = default expected × calibration_modifier
```

**Step 2: Sum units per bolt and add overhead**

For each bolt:

```
base_hours  = sum of expected hours for all units in the bolt

overhead    = base_hours × 0.20
              (covers retro, review session, integration testing, bolt summary)

qa_buffer   = (sum of expected hours for Complex units only) × 0.10
              (only applied if the bolt contains at least one Complex unit;
               zero otherwise)

total_hours = base_hours + overhead + qa_buffer
```

Example: a bolt with two Standard units (4.5 hrs each) and one Complex unit (10 hrs):

```
base      = 4.5 + 4.5 + 10   = 19 hrs
overhead  = 19 × 0.20         = 3.8 hrs
qa_buffer = 10 × 0.10         = 1 hr    (Complex unit only)
total     = 19 + 3.8 + 1      = 23.8 hrs → rounded to 24 hrs
```

**Step 3: Convert to days**

```
daily_capacity = team_size × hours_per_day
                 (full-time = 6 hrs/day; part-time = whatever the engineer states)

bolt_days = total_bolt_hours ÷ daily_capacity
```

Bolts are then sequenced in dependency order to produce the milestone map.

---

### Calibration from recorded hours — the feedback loop

This is the mechanism that makes estimates get more accurate over time.

**After each bolt closes**, the engineer records how many hours it actually took. The agent computes the **variance ratio** for that bolt:

```
bolt_variance = actual_hours ÷ estimated_hours
```

A variance of 1.0 means the estimate was exact. Above 1.0 means it took longer than estimated. Below 1.0 means the team delivered faster.

**Running calibration modifier** — after each bolt closes, the modifier is recomputed as the straight average of all bolt variances so far:

```
calibration_modifier = (bolt_1_variance + bolt_2_variance + ... + bolt_N_variance) ÷ N
```

**When the modifier triggers a recalibration:**

The modifier is applied to all remaining open bolt estimates only when it diverges meaningfully from 1.0 — specifically when it is above 1.15 (team is running slower than estimated) or below 0.85 (team is running faster). Minor variance within that band does not trigger a recalculation.

```
if calibration_modifier > 1.15 or calibration_modifier < 0.85:
    for each open unit:
        new_expected_hours = original_expected_hours × calibration_modifier
    recompute all open bolt totals
    recompute projected completion date
```

**Worked example:**

| Bolt | Estimated | Actual | Variance |
|---|---|---|---|
| Bolt 1 | 20 hrs | 24 hrs | 1.20 |
| Bolt 2 | 18 hrs | 20 hrs | 1.11 |
| Bolt 3 | 22 hrs | 25 hrs | 1.14 |

```
calibration_modifier = (1.20 + 1.11 + 1.14) ÷ 3 = 1.15
```

Exactly on the threshold — the agent flags this but does not recalibrate. After Bolt 4 closes, if the modifier rises above 1.15, all remaining open bolt estimates are multiplied by the new modifier and the projected completion date shifts accordingly.

---

## Estimate Artifacts

**Mode 1 artifact:** `estimates/[project-slug]-ballpark-YYYY-MM-DD.md`
Written to the current working directory. Contains: requirements summary, capability breakdown with tier classifications, hour ranges with modifiers applied, delivery timeline by team size, confidence level, top 3 risks, and scope boundary.

**Mode 2 artifact:** `{FRAMEWORK_ROOT}/ops/inception/estimates/YYYY-MM-DD-<timestamp>-[intent-slug]-delivery-estimate.md`
Written into the installed framework. Contains: unit classification table, bolt estimate table (base + overhead + QA buffer), release milestone map, rate model with calibration modifier, and the recorded-hours log updated after each bolt closes.
