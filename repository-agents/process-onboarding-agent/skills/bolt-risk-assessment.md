# Skill: Bolt Risk Assessment

**Purpose:** Actively interrogate a planned bolt for blast radius, cross-unit sequencing risks, rollback feasibility, and feature flag requirements before the first unit executes. Replaces the passive "Risks and Assumptions" table in the bolt file with structured, AI-driven findings that the engineer signs off on.

**Trigger:** Runs after elaboration sign-off and before the first unit in a bolt is executed. Invoked by the mob elaboration protocol automatically after the sign-off summary is confirmed, or directly by the engineer with "run risk assessment for bolt [name]."

**Never skip for mature projects.** For projects with existing code, this assessment is mandatory — no unit executes without a signed-off risk assessment in the bolt file. For fresh projects with no existing modules affected, the assessment may be brief but must still be completed.

---

## Step 1 — Read the Bolt and Its Units

Read the bolt file and every unit file listed in the bolt's Units table before saying anything.

Also read:
- `process-onboarding-agent/rules/architecture.md` — to understand existing module boundaries and ADRs
- `process-onboarding-agent/guidelines/forbidden-zones.md` — if it exists, to check whether any unit touches a protected zone
- The design artifact linked from the intent, if one exists

Confirm what you have read:

> "I've read the bolt [name] and its [N] units. I'll now assess the blast radius, sequencing risks, rollback approach, and feature flag requirements before execution begins."

Do not ask any questions yet — proceed directly to Step 2.

---

## Step 2 — Blast Radius Analysis

For each unit in the bolt, work through the following — internally, without asking questions unless something is genuinely ambiguous:

**What to assess per unit:**

| Question | What to look for |
|---|---|
| Which existing files or modules will this unit modify? | Named in the unit's Context or Scope sections |
| What existing behavior depends on those files? | Cross-references in architecture.md; other units in the bolt or backlog that link to the same modules |
| What is the worst-case impact if this unit introduces a defect? | Data loss, broken auth, degraded UI, silent failure, cascading failure in downstream modules |
| Does this unit touch any boundary defined in architecture.md? | API contracts, service boundaries, data ownership rules |
| Does this unit touch a forbidden zone? | Check forbidden-zones.md if it exists |

After assessing all units, produce a blast radius table:

```
Blast Radius — [Bolt name]

| Unit | Modules touched | Existing behavior at risk | Worst-case impact |
|---|---|---|---|
| [unit name] | [files/modules] | [behavior description] | [impact level: High / Med / Low] |
```

Flag any unit rated High immediately before presenting the full table:

> "Unit [name] has a high blast radius — it touches [module] which [existing behavior]. I'll highlight this in the assessment."

---

## Step 3 — Cross-Unit Sequencing Risks

Review the bolt's Execution Order and identify risks that arise specifically from the order units run in — not from individual units in isolation.

Look for:

- **State dependency risk:** Unit B assumes a state that Unit A creates. If Unit A fails mid-way, what is the system state?
- **Partial rollout risk:** If the bolt is interrupted after some units are done but before others, is the system in a consistent state?
- **Shared resource contention:** Two units that modify the same file, table, or config. If run in parallel, do they conflict?
- **Contract mismatch risk:** A unit that changes an interface that a later unit in the same bolt consumes. If the producer unit changes shape after the consumer unit is written, do they fall out of sync?

Produce a sequencing risk table if any risks are found:

```
Sequencing Risks

| Risk | Units involved | Condition | Mitigation |
|---|---|---|---|
| [description] | [unit A → unit B] | [when this occurs] | [how to prevent or recover] |
```

If no sequencing risks exist, state: "No cross-unit sequencing risks identified."

---

## Step 4 — Rollback Assessment

For the bolt as a whole, determine the rollback approach. Ask the engineer only if the answer cannot be determined from the unit files and architecture:

Assess each of the following:

**Code rollback:** Can reverting the commits for this bolt's units restore the previous behavior without side effects? Look for: database migrations in any unit (once applied, may not be reversible), external API calls that mutate state (webhooks sent, emails triggered, records created in third-party systems), or file system changes outside the repo.

**Data rollback:** Does any unit write to a database schema or seed data in a way that cannot be undone by reverting the code? If yes, a data rollback script or migration must be part of the unit's Definition of Done.

**Partial rollback:** If only some units in the bolt are merged when a problem is found, can those units be reverted independently, or do they form an atomic group that must be reverted together?

Produce a rollback summary:

```
Rollback Assessment

Code rollback:   [Safe — revert commits restores prior state]
                 [Unsafe — [reason]: engineer must [action] before reverting]

Data rollback:   [Not required — no schema or data changes]
                 [Required — migration needed: [description]]

Partial rollback:[Each unit is independently revertible]
                 [Units [X] and [Y] must be reverted together — [reason]]
```

---

## Step 5 — Feature Flag Requirement

Determine whether any unit in the bolt changes behavior that an existing user would observe in production — even if the change is intentional. The question is not whether the change is correct, but whether it should be introduced progressively.

Assess each unit against these criteria:

| Criterion | Feature flag required? |
|---|---|
| Changes the behavior of an existing feature visible to end users | Yes |
| Adds a new capability that replaces or competes with an existing one | Yes |
| Changes an API response shape consumed by existing clients | Yes |
| Modifies internal logic with no user-visible effect | No |
| Adds a new endpoint or screen that does not affect existing flows | No — unless explicitly required by project policy |

If a feature flag is required for any unit, state which unit(s) and what behavior the flag gates. If the project has a feature flag convention (in architecture.md or code-standards.md), reference it. If no convention is established, flag this as an open question for the engineer:

> "Unit [name] changes [behavior] which existing users will observe. A feature flag is required. Does this project have an established feature flag pattern, or should I log establishing one as an open question?"

---

## Step 6 — Risk Assessment Sign-off

Present the full assessment before writing anything to the bolt file:

```
Risk Assessment — [Bolt name]

BLAST RADIUS
[paste blast radius table from Step 2]

SEQUENCING RISKS
[paste sequencing risk table from Step 3, or "None identified"]

ROLLBACK
[paste rollback summary from Step 4]

FEATURE FLAGS
[list units requiring flags, or "None required"]

HIGH-PRIORITY ITEMS (requiring engineer decision before execution):
[list any High blast radius units, unsafe rollbacks, or open feature flag questions]
[or "None — assessment is clear to proceed"]

Sign off to proceed with unit execution?
```

If the engineer raises concerns or requests changes to the assessment, revise the relevant section and present the summary again. Do not proceed to Step 7 until the engineer explicitly confirms sign-off.

---

## Step 7 — Write to the Bolt File

Replace the bolt file's "Risks and Assumptions" section with the signed-off assessment. Do not keep the original passive table — it is superseded by this output.

Write the following structure into the bolt file:

```markdown
## Risk Assessment

**Assessed:** YYYY-MM-DD  
**Status:** Signed off by engineer before execution

### Blast Radius

| Unit | Modules touched | Existing behavior at risk | Impact |
|---|---|---|---|
| [unit] | [modules] | [behavior] | High / Med / Low |

### Sequencing Risks

| Risk | Units involved | Condition | Mitigation |
|---|---|---|---|
| [description] | [units] | [condition] | [mitigation] |

_(or: No sequencing risks identified.)_

### Rollback

- **Code rollback:** [safe / unsafe — reason and required action]
- **Data rollback:** [not required / required — description]
- **Partial rollback:** [independently revertible / must revert together — units and reason]

### Feature Flags

- [Unit name]: gates [behavior description]
_(or: No feature flags required.)_

### Open Items

- [Any items the engineer must resolve before or during execution, or "None"]
```

After writing, confirm:

> "Risk assessment written to the bolt file. All [N] open items (if any) must be resolved before the affected units execute. Ready to begin unit execution — say 'execute unit [first unit name]' to start."
