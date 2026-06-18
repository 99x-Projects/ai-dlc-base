# Unit: [name]

**Status:** Open | Planned | In Progress | Done | Blocked | Deferred
**Intent:** [ops/inception/intents/YYYY-MM-DD-<unix_timestamp>-[slug].md](../../inception/intents/YYYY-MM-DD-<unix_timestamp>-[slug].md)
**Elaboration:** [ops/inception/elaborations/YYYY-MM-DD-<unix_timestamp>-[slug]-session-N.md](../../inception/elaborations/YYYY-MM-DD-<unix_timestamp>-[slug]-session-N.md)
**Bolt:** [ops/build/bolts/YYYY-MM-DD-<unix_timestamp>-[bolt-slug].md](../bolts/YYYY-MM-DD-<unix_timestamp>-[bolt-slug].md)
**Priority:** High | Medium | Low

---

## Context

[One paragraph. Who needs this, what it does, and why it is a discrete unit. Written for an engineer who has not read the intent.]

---

## Acceptance Criteria

1. Given [precondition], when [action], then [outcome].
2. Given [precondition], when [action], then [outcome].
3. Given [precondition — unhappy path], when [action], then [outcome].

---

## Scope

**In scope:**
- [What this unit covers]

**Out of scope:**
- [What is explicitly excluded — handled by another unit or not planned]

---

## Dependencies

| Dependency | Type | Status |
|---|---|---|
| [Unit or external system] | Unit / API / Config | Done / Pending |

---

## Pre-generation Checks

Before generating code for this unit, the agent must run these checks:

- [ ] Grep for existing implementations of [pattern] across the codebase to avoid duplication
- [ ] Confirm the module entry point is listed in `process-onboarding-agent/guidelines/entry-points.md`
- [ ] Confirm no files in scope appear in `process-onboarding-agent/guidelines/forbidden-zones.md`
- [ ] Verify test coverage for affected module meets the gate threshold

---

## Edge Cases to Handle

| ID | Scenario | Required behavior |
|---|---|---|
| EC-001 | [scenario] | [what the code must do] |

---

## Observability

> What evidence must exist in production to know this unit is working correctly? Any entry that represents code behavior the AI must implement should be expressed as an AC in the Acceptance Criteria section above.

**Success signal** — what metric or log entry confirms this unit is operating normally?
- [ ] [e.g. "A counter increments on every successful call to /api/bookings"]
- [ ] [or: "Not applicable — this unit has no production traffic or async process to monitor"]

**Failure signal** — what log entry confirms something has gone wrong?
- [ ] [e.g. "A WARNING-level log entry is written when payment processing fails, including the order ID and error code"]

**Alert threshold** — at what point should an on-call engineer be notified?
- [ ] [e.g. "Alert fires if the error rate for /api/bookings exceeds 5% over a 5-minute window"]
- [ ] [or: "Not applicable"]

---

## Breaking Changes Register

> **Complete this section only for Migration or Remediation Bolts that change contract boundaries. Leave blank (write "N/A — not a contract-change Bolt") for all other Bolts. This section must be completed and approved before any code is generated.**

| # | Contract boundary changed | Reason it must change | Approved by | Date |
|---|---|---|---|---|
| 1 | [API endpoint / data schema / inter-module interface] | [why the change is necessary] | [engineer name] | YYYY-MM-DD |

---

## Definition of Done

- [ ] All ACs implemented and traceable to code
- [ ] Unit tests written for each AC
- [ ] Integration tests for affected module pass without modification *(or: all breaking changes listed in the Breaking Changes Register have updated tests and are approved)*
- [ ] No secrets or hardcoded environment values
- [ ] Auth checked on every new endpoint
- [ ] Reviewed against `process-onboarding-agent/skills/review-checklist.md`
- [ ] Prompt log updated in `process-onboarding-agent/prompts/`
- [ ] Unit status set to Done in backlog

---

## Prompt Log

[process-onboarding-agent/prompts/[unit-name].md](../../../prompts/[unit-name].md)

---

## Notes

[Any decisions made during execution that future engineers should know about.]
