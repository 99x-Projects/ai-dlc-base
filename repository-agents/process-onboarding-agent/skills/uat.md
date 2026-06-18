# Skill: User Acceptance Testing (UAT)

**Purpose:** Generate a plain-language demo script from the acceptance criteria of all units under an intent, guide the engineer through conducting a stakeholder validation session, and record the outcome in the intent file. Bridges the gap between "code merged" and "intent closed as Implemented."

**Trigger:** Runs when all units under an intent are marked Done — either prompted automatically by the AI ("All units under this intent are now complete. Would you like to run UAT before closing the intent?") or invoked directly by the engineer ("run UAT for [intent]").

**Three paths:** The engineer may choose to conduct UAT now, defer it to a later date, or mark it as not required. All three choices are recorded in the intent file. The intent can only move to status Implemented after a UAT Sign-off entry exists — regardless of which path was taken.

---

## Step 1 — Identify the Intent and Read Its Units

Ask the engineer if invoked directly:

> "Which intent should I prepare UAT for?"

Read the intent file and every unit file listed in its "Extracted Units" table. Collect all Acceptance Criteria from every unit.

Confirm what was found:

> "I've read [N] units under the intent [intent name] and collected [N] acceptance criteria in total. Before I build the demo script, do you want to conduct UAT now, defer it to a later date, or mark it as not required for this intent?"

Present the three options clearly:

| Option | What it means |
|---|---|
| **Conduct now** | Run through the UAT session in this conversation. I'll generate a demo script and record results step by step. |
| **Defer** | Record UAT as deferred. The intent can close as Implemented, but the UAT entry will flag it as outstanding. You choose a revisit date. |
| **Not required** | Record that UAT was explicitly waived for this intent. Requires a stated reason. |

Wait for the engineer to choose before proceeding.

---

## Step 2A — Conduct UAT Now

*Take this path if the engineer chose "Conduct now."*

### Build the Demo Script

Translate the collected ACs into a numbered plain-language demo script. Apply these translation rules:

- Write each step as: **Setup** (the starting state) → **Action** (what to do) → **Expected outcome** (what to look for)
- Use the stakeholder's vocabulary from the intent's "What" and "Success Looks Like" sections — not engineering terms
- Group related ACs into logical user journeys rather than listing them unit by unit
- If two ACs describe the same user action with different outcomes (happy path and error path), combine them into one step with two expected outcomes
- Never mention unit names, AC numbers, Given/When/Then syntax, or technical implementation details in the demo script

Present the demo script to the engineer before the session begins:

```
UAT Demo Script — [Intent name]
Generated: YYYY-MM-DD

[Journey name, e.g. "Creating a booking"]

Step 1 — [Plain-language action]
  Setup:    [Starting state in plain language]
  Action:   [What the tester does]
  Expected: [What should happen]

Step 2 — [Plain-language action]
  Setup:    [Starting state]
  Action:   [What the tester does]
  Expected: [What should happen]
  Also check (error case): [What should happen if X goes wrong]

[Next journey if applicable]
...
```

Ask the engineer:

> "Does this demo script cover the behavior you want to validate? Add, remove, or adjust any steps before we start the session."

Revise the script if the engineer requests changes. Proceed only when the engineer confirms the script is ready.

### Run the Session

Work through the demo script one step at a time. For each step:

1. Display the step clearly.
2. Ask: > "Run this step and tell me the result: Pass, Fail, Partial, or Skip."
3. If **Fail** or **Partial**: ask > "Describe what happened instead of the expected outcome."
4. Record the result and move to the next step.

Do not ask for more than one step at a time.

### Record the Session Outcome

After all steps are completed, calculate and display the outcome summary:

```
UAT Session Complete — [Intent name]
Date: YYYY-MM-DD

Results:
  Pass:     [N] steps
  Partial:  [N] steps
  Fail:     [N] steps
  Skipped:  [N] steps

Overall: [Passed | Passed with findings | Failed | Inconclusive]
```

**Overall outcome rules:**
- All steps Pass or Skip → **Passed**
- At least one Partial, no Fails → **Passed with findings**
- At least one Fail → **Failed**
- More than half skipped → **Inconclusive**

For any Fail or Partial result, ask the engineer:

> "Step [N] failed / was partial. Should I log this as a new unit in the backlog (a bug fix or gap), or is this already tracked elsewhere?"

If the engineer confirms a new unit is needed, create a draft unit file at `process-onboarding-agent/ops/build/units/YYYY-MM-DD-<unix_timestamp>-uat-[description].md` with:
- Status: Open
- Context: populated from the UAT failure description
- Acceptance Criteria: the step's expected outcome restated as a Given/When/Then AC
- Intent link: pointing back to this intent

Add the new unit to the Open section of the backlog.

Proceed to Step 3 to write the sign-off to the intent file.

---

## Step 2B — Defer UAT

*Take this path if the engineer chose "Defer."*

Ask:

> "When do you plan to conduct UAT for this intent? Give me a date or a condition (e.g. 'after client demo on [date]', 'before go-live')."

Record the answer. Then ask:

> "Is there anything specific that needs to be in place before UAT can happen? (e.g. test environment, stakeholder availability, data setup)"

Record the answer.

Proceed to Step 3 to write the deferred sign-off to the intent file.

---

## Step 2C — Mark as Not Required

*Take this path if the engineer chose "Not required."*

Ask:

> "What is the reason UAT is not required for this intent? This is recorded in the intent file."

Common valid reasons: internal tooling with no external stakeholders, purely technical refactor with no user-visible behavior change, stakeholder sign-off already obtained through a different channel.

Record the reason. Proceed to Step 3.

---

## Step 3 — Write the UAT Sign-off to the Intent File

Write a `## UAT Sign-off` section into the intent file, inserted between the "Extracted Units" section and the "Implementation Summary" section.

**For "Conduct now" — Passed or Passed with findings:**

```markdown
## UAT Sign-off

**Status:** Passed
**Date:** YYYY-MM-DD
**Conducted by:** [engineer name or "engineer + [stakeholder name]"]

### Session Results

| Step | Description | Result | Notes |
|---|---|---|---|
| 1 | [step description] | Pass / Partial / Skip | [notes if any] |

**Overall:** Passed — [N] of [N] steps passed. [N] partial results noted.

### Findings Logged

[List of new units created from UAT failures, or "None — all steps passed."]
```

**For "Conduct now" — Failed:**

```markdown
## UAT Sign-off

**Status:** Failed — not ready to close
**Date:** YYYY-MM-DD
**Conducted by:** [engineer name]

### Session Results

| Step | Description | Result | Notes |
|---|---|---|---|
| 1 | [step description] | Pass / Fail / Partial | [failure description] |

**Overall:** Failed — [N] step(s) did not meet the expected outcome.

### Findings Logged

[List of new units created to address failures]

> Re-run UAT after findings are resolved before closing this intent.
```

**For "Defer":**

```markdown
## UAT Sign-off

**Status:** Deferred
**Deferred on:** YYYY-MM-DD
**Planned for:** [date or condition stated by engineer]
**Prerequisites:** [what needs to be in place, or "None stated"]

> This intent may be closed as Implemented with UAT deferred. Update this section when UAT is conducted.
```

**For "Not required":**

```markdown
## UAT Sign-off

**Status:** Not required
**Date:** YYYY-MM-DD
**Reason:** [reason stated by engineer]
```

---

## Step 4 — Update Intent Status and Confirm

After writing the UAT section:

- If UAT **Passed** or **Passed with findings**: the intent may be closed. Ask > "Shall I set the intent status to Implemented and trigger the Implementation Summary?"
- If UAT **Failed**: do not close the intent. Inform the engineer: > "The intent stays In Progress until the [N] UAT findings are resolved and UAT is re-run."
- If UAT **Deferred** or **Not required**: the intent may be closed. Ask > "Shall I set the intent status to Implemented? The deferred UAT is recorded and can be revisited at any time."

Confirm completion:

```
UAT complete.

Intent:  [intent name]
Status:  [UAT status]
Outcome: [what was recorded]

[If new units were created:]
New units logged: [list]

[If intent is being closed:]
Intent status → Implemented. Ready to write the Implementation Summary.
```
