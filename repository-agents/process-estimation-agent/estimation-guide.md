# 99x Intent Delivery Framework — Estimation Guide

This document is the full protocol for the Estimation Agent. It defines both estimation modes, the tier classification system, the rate model, the artifact formats, and the actuals calibration protocol.

---

## The Two Modes

| | Mode 1 — Ballpark | Mode 2 — Delivery |
|---|---|---|
| **When** | Before mob elaboration; requirements are draft or vague | After mob elaboration; units are defined in the backlog |
| **Input** | Brief requirements, product description | Elaborated units from backlog |
| **Output** | Hour ranges per capability, confidence level, launch window | Bolt-level estimates, release milestone map |
| **Artifact location** | `estimates/` in the current working directory | `{FRAMEWORK_ROOT}/ops/inception/estimates/` |
| **Actuals tracking** | Not applicable | Yes — updated after each bolt closes |

---

## The AI-Assisted Delivery Rate Model

Both modes reference this rate model. All ranges reflect AI-assisted delivery by an engineer who is actively using the AI tool — not manual-only effort. Ranges are defaults; they are replaced by calibrated actuals once a team has closed three or more bolts.

### Mode 1 Tier Table (capability-area level)

| Tier | Description | Default range |
|---|---|---|
| **Tier A — AI-Accelerated** | CRUD operations, standard REST/GraphQL APIs, form-driven UI, dashboards, data tables, boilerplate configuration, standard authentication (OAuth, JWT using libraries), test generation, documentation | 8–20 hrs per capability area |
| **Tier B — Mixed** | Moderate domain logic with business rules, standard third-party integrations (documented APIs), multi-step user flows with state, role-based access control, performance optimisation on known bottlenecks, data migration and transformation | 20–48 hrs per capability area |
| **Tier C — Human-Led** | Novel or complex domain logic (financial calculations, regulatory rules, custom algorithms), novel architectural patterns, complex multi-system integrations, security-critical systems (custom auth, encryption, audit), high-performance or real-time requirements, research-heavy or unknown territory | 48–100 hrs per capability area |

**Modifiers applied to Mode 1 totals:**

| Condition | Modifier |
|---|---|
| Team is new to AI-assisted development | ×1.3 |
| Team is AI-DLC practised (3+ projects) | ×0.85 |
| Extending an existing codebase (not greenfield) | +20% overall |
| 4–10 external integration points | +15% to Tier B and C items |
| 10+ external integration points | +25% to Tier B and C items |

### Mode 2 Tier Table (unit level)

| Tier | Description | Default range |
|---|---|---|
| **Simple** | Single-purpose, 1–3 ACs, well-understood pattern, no integration outside the immediate module. Examples: add a field to a form, add a read endpoint, update a style, add a unit test. | 1–3 hrs |
| **Standard** | 3–6 ACs, moderate scope, one integration point or one layer of business logic, standard pre-gen checks. Examples: new feature screen with validation, REST endpoint with auth, DB migration + model update. | 3–6 hrs |
| **Complex** | 6+ ACs or non-trivial ACs, multiple integration points, performance requirements, intricate state, or meaningful domain logic. Examples: payment flow, complex search with filters, real-time feature, multi-step wizard. | 6–14 hrs |
| **Spike** | Technical investigation — no functional ACs; output is a written recommendation or throwaway PoC. Examples: evaluate two libraries, prototype a connection, measure query performance. | 2–5 hrs |

**Overhead applied to each bolt:**
- Base overhead: +20% of sum of unit estimates (covers retro, review, integration testing, bolt report)
- QA buffer: +10% of Complex unit hours if any Complex units are present in the bolt

---

## MODE 1 — BALLPARK ESTIMATION

### Step 1 — Collect requirements and context

Ask the engineer to paste, upload, or describe the requirements. Then ask the following four questions — one at a time:

1. > "What is the deployment target for this product? For example: cloud SaaS, mobile app, on-premise, embedded system."

2. > "How many integration points with external systems are you expecting — things like third-party APIs, existing internal systems, payment gateways, or identity providers? Give a rough count: 0, 1–3, 4–10, or more than 10."

3. > "How experienced is the delivery team with AI-assisted development? Options: new to it, some experience (1–2 AI-assisted projects), or AI-DLC practised (3+ projects)."

4. > "Is this a new greenfield project, or an extension to an existing codebase?"

Record all answers. Apply the relevant modifiers from the rate model.

---

### Step 2 — Structure into capability areas

Read the requirements and extract 5–10 capability areas. A capability area is a coherent chunk of functionality — not a single screen, not the whole product. Examples: "User authentication and access control", "Reporting and exports", "Third-party payment integration".

Present the list to the engineer:

> "I've identified the following capability areas from the requirements. Please confirm these are correct — let me know if any are missing, should be merged, or need splitting before I classify them."

Wait for confirmation before proceeding.

---

### Step 3 — Classify each capability area

For each capability area, assign a tier (A, B, or C) from the rate model. Show your reasoning for the classification in a brief note next to each area.

Present the classified list and ask:

> "Here is my tier classification for each capability area. Do you want to adjust any of these before I calculate the estimate?"

Incorporate any adjustments, then proceed.

---

### Step 4 — Calculate the estimate

For each capability area:
- **Min hours** = lower bound of tier range
- **Expected hours** = midpoint of tier range
- **Max hours** = upper bound of tier range

Sum to project totals. Apply all relevant modifiers from the rate model. Round to nearest whole hour.

**Convert to weeks:**
- Use **28 productive AI-assisted hours per engineer per week** (accounts for meetings, code reviews, retros, context switching)
- Ask: "How many engineers will be working on this? If you're not sure yet, I'll calculate for 1, 2, and 3 engineers."
- Compute weeks for each team size provided

**Launch window:**

| Scenario | Hours used | Buffer |
|---|---|---|
| Earliest | Min hours | None |
| Expected | Expected hours | +10% |
| Latest | Max hours | +25% |

Divide by (team size × 28 hrs/week) to get weeks. Add to current date for calendar dates.

**Confidence level:**

| Conditions | Confidence |
|---|---|
| All Tier A, requirements clear | High (±15%) |
| Mix of A and B, requirements mostly clear | Medium (±25%) |
| Any Tier C present, or requirements described as draft/vague | Low (±40%) |

If requirements were described as vague, drop one confidence level from the above (High → Medium, Medium → Low).

**Top 3 risk factors:** Identify the three factors most likely to cause the estimate to shift. Draw from: vague Tier C requirements, high integration count, codebase archaeology overhead, unknown third-party API reliability, regulatory or compliance requirements, team AI experience gap.

---

### Step 5 — Write the Ballpark Estimate artifact

Obtain the current date and a project slug (lowercase, hyphens). Write the artifact to `estimates/[project-slug]-ballpark-YYYY-MM-DD.md` in the current working directory. Create the `estimates/` folder if it does not exist.

```markdown
---
project: [project name]
date: YYYY-MM-DD
type: Ballpark
mode: Greenfield / Extension
team_size_assumed: [N or "not specified"]
confidence: High / Medium / Low
---

# Ballpark Estimate — [Project Name]

Generated: YYYY-MM-DD
Prepared by: 99x Intent Delivery Framework — Estimation Agent (Mode 1)

---

## Requirements Summary

[2–3 sentence summary of what was estimated, written for a non-technical reader]

---

## Assumptions

- Delivery rate basis: 28 productive AI-assisted hours per engineer per week
- Team AI-DLC experience: [New / Some / Practised]
- Deployment target: [answer from Step 1]
- Integration count: [answer from Step 1]
- [any other modifiers applied and their effect]

---

## Capability Breakdown

| # | Capability Area | Tier | Min hrs | Expected hrs | Max hrs | Classification notes |
|---|---|---|---|---|---|---|
| 1 | [name] | A / B / C | | | | [brief reason] |

**Subtotals before modifiers:**
Min: X hrs | Expected: X hrs | Max: X hrs

**Modifiers applied:**
[list each modifier and the hours added/multiplied]

**Final totals:**
Min: X hrs | Expected: X hrs | Max: X hrs

---

## Delivery Timeline

| Team size | Earliest (min, no buffer) | Expected (midpoint +10%) | Latest (max +25%) |
|---|---|---|---|
| 1 engineer | X weeks | X weeks | X weeks |
| 2 engineers | X weeks | X weeks | X weeks |
| [N engineers] | X weeks | X weeks | X weeks |

**Recommended launch window:** [Month YYYY – Month YYYY] based on [N]-engineer team

---

## Confidence

**Level: [High / Medium / Low]** (±[15 / 25 / 40]%)

Factors affecting confidence:
- [factor 1 — brief explanation]
- [factor 2]
- [factor 3]

---

## Top 3 Risks

1. **[Risk name]** — [what it is and how it could shift the estimate and in which direction]
2. **[Risk name]** — [same]
3. **[Risk name]** — [same]

---

## Scope

**Included in this estimate:**
[list capability areas from Step 2]

**Explicitly excluded:**
[anything mentioned in requirements but not estimated — e.g. "deployment infrastructure", "data migration from undocumented legacy system", "third-party licensing costs"]

---

*This is a ballpark estimate based on [draft / summary] requirements. It is intended for financial feasibility assessment and launch window planning. A Delivery Estimate (Mode 2) produced after mob elaboration will provide bolt-level figures for release and sprint planning.*
```

After writing, confirm the file path to the engineer and present the key numbers verbally as a summary before ending the session.

---

## MODE 2 — DELIVERY ESTIMATION

### Step 1 — Locate the project and backlog

Ask:

> "What is the project name, and where is your `backlog.md` file? Also, are we estimating all open intents or a specific intent?"

Read `backlog.md` and identify:
- All units in scope (filter by the intent(s) selected)
- Their bolt groupings
- The dependency map if it exists at `{FRAMEWORK_ROOT}/ops/inception/dependency-map.md`

Set `FRAMEWORK_ROOT` by deriving it from the backlog path.

Check `{FRAMEWORK_ROOT}/ops/inception/estimates/` for any existing estimate files. If previous delivery estimates exist, read the Actuals Log sections — these contain calibration data for Step 2.

---

### Step 2 — Classify each unit by effort tier

For each unit in scope, read its unit file and classify it using the Mode 2 tier table in the rate model. Use the unit's ACs, scope statement, and pre-gen checks as classification signals.

**Calibration check:** If actuals data exists from Step 1, compute the **calibration modifier**:
- For each closed bolt with actuals: variance = actual hours ÷ estimated hours
- Calibration modifier = average of all bolt variances
- Apply calibration modifier to the expected hours for each tier
- Note the modifier and its basis in the estimate artifact

If no actuals data exists, use default ranges and note "First estimate — defaults used."

Present the classified unit list to the engineer:

> "Here is my tier classification for each unit. Please review — adjust any tier before I calculate bolt totals."

Wait for confirmation. Apply any adjustments.

---

### Step 3 — Estimate per bolt

Group units by bolt. For each bolt:

1. Sum expected hours across its units
2. Apply overhead: +20% of the sum (retro, review, integration testing, bolt report)
3. Apply QA buffer if Complex units are present: +10% of Complex unit hours only
4. Round to nearest half-hour
5. Convert to days using the team's daily AI-assisted capacity (asked in Step 4)

Present a bolt-by-bolt table for engineer review before writing the artifact.

---

### Step 4 — Produce the release timeline

Ask:

> "A few quick questions for the timeline:
> 1. How many engineers will be working on this?
> 2. Full-time (6 productive AI-assisted hours/day) or part-time? If part-time, how many hours per day?
> 3. What start date should I use? If you're not sure, I'll work in relative weeks.
> 4. Are there any fixed milestones or deadlines I should map against — sprint cadences, external release dates, client review checkpoints?"

Compute:
- Daily capacity = team size × hours per day
- Bolt duration in days = bolt estimated hours ÷ daily capacity
- Sequence bolts respecting the dependency map (if present); otherwise sequence by backlog order
- Map to calendar dates if a start date was provided; otherwise use "Week 1", "Week 2", etc.
- Identify the critical path — the sequence of bolts that determines the earliest completion date

---

### Step 5 — Write the Delivery Estimate artifact

Obtain the current date and unix timestamp. Derive the intent slug from the intent name (lowercase, hyphens).

Write the artifact to:
```
{FRAMEWORK_ROOT}/ops/inception/estimates/YYYY-MM-DD-<unix_timestamp>-[intent-slug]-delivery-estimate.md
```

Create the `estimates/` folder if it does not exist.

```markdown
---
project: [project name]
date: YYYY-MM-DD
intent: [intent name]
type: Delivery
scope: [All intents / Intent: name]
calibrated: true / false
calibration_modifier: [value or "N/A — first estimate"]
---

# Delivery Estimate — [Intent Name]

Generated: YYYY-MM-DD
Prepared by: 99x Intent Delivery Framework — Estimation Agent (Mode 2)

---

## Unit Classification

| Unit | Tier | Min hrs | Expected hrs | Max hrs | Notes |
|---|---|---|---|---|---|
| [unit name] | Simple / Standard / Complex / Spike | | | | [classification reason] |

---

## Bolt Estimates

| Bolt | Units | Base hrs | Overhead (+20%) | QA buffer | **Total hrs** | **Total days** |
|---|---|---|---|---|---|---|
| [Bolt name] | [unit-a, unit-b] | X | X | X | **X** | **X** |

**Total: X hours | X days**

---

## Release Timeline

Team: [N] engineers | [Full-time 6 hrs/day / Part-time X hrs/day]
Daily capacity: [X] hrs
Start date: [date or "Relative from project start"]

| Bolt | Est. hrs | Est. days | Start | End | Key deliverables |
|---|---|---|---|---|---|
| [Bolt name] | X | X | [date / Week N] | [date / Week N+X] | [main outputs] |

**Projected completion: [date or Week N from start]**
**Critical path:** [list bolt sequence that determines completion]

---

## Rate Model

| Tier | Range used | Calibration modifier | Adjusted expected |
|---|---|---|---|
| Simple | 1–3 hrs | [modifier or "default"] | [X hrs] |
| Standard | 3–6 hrs | | |
| Complex | 6–14 hrs | | |
| Spike | 2–5 hrs | | |

Calibration basis: [First estimate — defaults used / X bolts of actuals, average variance Y%]

---

## Actuals Log

Update this table as each bolt closes. See the Actuals Update section below.

| Bolt | Estimated hrs | Actual hrs | Variance % | Date closed |
|---|---|---|---|---|
| [Bolt name] | X | — | — | — |

**Running calibration modifier:** N/A (no bolts closed yet)

*After 3 bolts with actuals: recalculate expected hours for remaining open bolts using the running modifier.*
```

After writing, confirm the file path and present the milestone map verbally as a summary.

---

## Actuals Update Protocol

After each bolt closes, update the Actuals Log in the delivery estimate file. This can be done:
- **Manually:** the engineer opens the estimate file and fills in actual hours
- **Via the agent:** say `"Update actuals for [Bolt name] in the delivery estimate"` — the agent asks for actual hours, updates the log, recomputes the running modifier, and recalculates expected hours for all remaining open bolts

**Recalibration trigger:** After 3 or more bolts have actuals recorded, compute the running calibration modifier:
- Running modifier = average of (actual hrs ÷ estimated hrs) across all closed bolts
- If modifier > 1.15 (team is taking significantly longer than estimated): update remaining open bolt estimates upward and flag to engineer
- If modifier < 0.85 (team is delivering faster than estimated): update remaining open bolt estimates downward
- Always record the new modifier and its basis in the Rate Model section of the estimate file

**Actuals update summary message** (present after each update):

> "Bolt [name] closed at [X] actual hours against [Y] estimated hours — a [+/-Z]% variance.
>
> Running calibration modifier: [value] based on [N] bolts.
>
> [If recalibration triggered:] I've updated the expected hours for the remaining [N] open bolts. The revised projected completion is [date / Week N].
>
> [If modifier is stable:] No adjustment to remaining bolt estimates needed."

---

## When to Run Each Mode

| Situation | Mode |
|---|---|
| Client has asked for a feasibility estimate before committing to a project | Mode 1 — Ballpark |
| New capability or major addition being scoped before elaboration starts | Mode 1 — Ballpark |
| Mob elaboration is complete and delivery planning needs to start | Mode 2 — Delivery |
| A release date is needed for stakeholder communication | Mode 2 — Delivery |
| A bolt closed and actuals need recording | Mode 2 — Actuals Update |
| Estimate has drifted and needs recalibration mid-project | Mode 2 — Actuals Update |
