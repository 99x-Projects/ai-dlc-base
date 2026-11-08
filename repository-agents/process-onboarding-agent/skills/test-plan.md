# Skill: Test Plan

**Purpose:** Bind every acceptance criterion on a unit to exactly one test level, and record the project's existing command for `unit`, `integration`, and `system`, before any code is generated. This skill writes the plan. It does not run tests.

**Trigger:** After elaboration sign-off and bolt risk assessment, before the first code response for a unit. Invoke directly with: "Read {path}/test-plan.md and write the test plan for <unit>."

**Gate:** Do not generate code for a unit until every AC has a level. A missing level blocks code generation. Stop when the plan is written. Do not generate code, run a command, or start UAT.

**Classification:** ◆ Standalone. With the framework installed, write the plan on the unit. Without it, ask for the AC list and write a standalone markdown file where the engineer says.

---

## Levels

Exactly these five. Do not add a sixth. Performance and security stay on `nfr-bolt.md` and `dependency-audit.md`. A "component test" is recorded as `unit` or `integration`, whichever matches the command the project already uses. Do not invent a level name for it.

| Level | What it proves | Default from the AC wording |
|---|---|---|
| `unit` | One pure function or module, dependencies mocked | a pure function |
| `integration` | A boundary between modules, or an external service this unit owns | a boundary between modules, or an external service the unit owns |
| `system` | A user journey through the assembled system | a user journey through the assembled system |
| `acceptance` | A stakeholder outcome | a stakeholder outcome. Evidence is the pointer `uat.md`. Not executed here. |
| `exploratory` | Unknown behavior, usability, or "poke at it" | unknown behavior, usability, or "poke at it" |

The engineer can override the default. One AC gets one level.

---

## Step 1 — Resolve the unit

If the framework is installed, read the unit file and the intent it links, for AI Risk. Confirm before asking anything else:

> "I've read unit [name] and intent [name]. AI Risk is [Minimal | Limited | High | not recorded]. I found [N] acceptance criteria. I'll assign one level to each. I will not generate code and I will not run tests."

Acceptance criteria are the numbered Given / When / Then lines under `## Acceptance Criteria`. Do not treat edge-case rows, observability checkboxes, or Definition of Done lines as ACs.

If there is no unit file, ask and wait:

> "Paste the acceptance criteria, and tell me the markdown file path where I should write the test plan."

Do not invent ACs. Do not choose a path the engineer did not give.

---

## Step 2 — Assign one level per AC

Work one AC at a time. State the default level and the clause that triggered it, then ask:

> "AC [n]: I read this as [level] because [clause]. Say yes, or name a different level: unit, integration, system, acceptance, or exploratory."

Wait for the answer before the next AC. If the wording matches none of the defaults, do not guess. Ask which of the five levels proves it, and wait.

If the engineer will not assign a level, stop. Do not write the plan. Do not generate code. Tell them code generation stays blocked until every AC has a level.

---

## Step 3 — Store the project's command

Do this only for a level that appears on at least one row among `unit`, `integration`, and `system`. Ask once per level, not once per AC. Do not ask for a command for `acceptance` or `exploratory`.

Look for the project's code standards, in this order:

1. `{FRAMEWORK_ROOT}/rules/code-standards.md` when the framework root is known
2. `rules/code-standards.md` anywhere else in the project

A command counts only when that file states a command for **that** level. A command is an executable invocation the file already quotes or writes as a command. Copy that string exactly. Do not run it.

These are not a command:

- a coverage threshold or a percentage
- a tool name, a language, or a framework with no command string
- a directory of tests
- prose, including "integration tests are required" or "system tests run in staging"
- a command the file states for a different level

If the file has no command for a level this plan uses, ask and wait:

> "What command does this project already use for [level] tests? I'll store the string and I will not run it. If there is no command, say so."

If the engineer gives a command, store that string. If they say there is no command, store `no command`.

Never invent a command. Never fill one in from the stack, from a filename, from a coverage threshold, or by reusing another level's command.

`acceptance` rows store the pointer `uat.md` and status `planned`. Do not open `uat.md`. Do not start a UAT session. Do not write a sign-off.

`exploratory` rows store status `planned`. Put `not this unit` in Evidence only when the engineer says this unit is not where that exploratory work belongs. Otherwise leave Evidence empty. Do not charter a session and do not write a session note.

---

## Step 4 — Write the table and stop

Append `## Test Plan` to the unit when that section is absent. On the standalone path, write it to the path the engineer gave. If `## Test Plan` is already present, do not add a second section. Show the existing table and stop.

One row per AC, numbered as in the unit. Every status is `planned`. Do not write `passed`, `waived`, or `gap`. This skill does not record a waiver. An empty status is not a waiver.

```markdown
## Test Plan

| AC | Level | Evidence | Status |
|---|---|---|---|
| [n] | [level] | [command, "no command", "uat.md", "not this unit", or empty] | planned |

Waiver: who, why. Empty status is not a waiver.
```

Evidence by level:

| Level | Evidence cell |
|---|---|
| `unit`, `integration`, `system` | the stored command, or `no command` |
| `acceptance` | `uat.md` |
| `exploratory` | empty, or `not this unit` when the engineer said so |

Then stop. Do not run any command. Do not generate code. Do not start UAT.
