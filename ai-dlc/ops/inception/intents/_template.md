# Intent: [Name]

**Status:** Draft | Ready | In Elaboration | Elaborated | In Progress | Implemented | Deferred | Cancelled
**Date:** YYYY-MM-DD
**Owner:** [Engineer name]

---

## What

[One paragraph describing what this capability is. Written from the user's perspective — what they can do, not how the system does it.]

## Why

[Why does this matter? What problem does it solve or what opportunity does it capture? Include any business or user context.]

## Success Looks Like

[Describe the observable outcome when this intent is fully delivered. Not technical metrics — describe what is true for the user or business.]

## Assumptions

- [Assumption 1]
- [Assumption 2]

## Open Questions

- [Question 1]
- [Question 2]

## Out of Scope

- [Thing explicitly not included]
- [Thing explicitly not included]

---

## Elaboration Sessions

| Session | Date | File |
|---|---|---|
| 1 | YYYY-MM-DD | [elaborations/YYYY-MM-DD-[slug]-session-1.md](../elaborations/YYYY-MM-DD-[slug]-session-1.md) |

---

## Extracted Units

| Unit | Status | Bolt |
|---|---|---|
| [unit-name](../../build/units/[unit-file].md) | Open | — |

---

## UAT Sign-off

> **Complete this section before setting the intent status to Implemented.** Run `ai-dlc/skills/uat.md` when all units are Done. The engineer may choose to conduct UAT now, defer it, or waive it — all three choices are recorded here.

**Status:** Pending | Passed | Passed with findings | Failed | Deferred | Not required

---

## Implementation Summary

> **Complete this section when all units under this intent have been delivered and merged. Do not fill it in until the intent status is set to Implemented.**

### What Was Built

[Describe the features delivered under this intent from the user's perspective — what they can now do that they could not before. Write one paragraph per distinct feature if the intent contained multiple. Do not describe implementation details.]

### How It Works (Key Design Decisions)

[Explain the key design decisions made during elaboration and build that a future engineer must understand before modifying this feature. Include: data model choices, API contract decisions, edge cases that were explicitly handled, and anything non-obvious about the implementation.]

### Scope Delivered vs. Original Intent

[State any deviations from the original intent: features descoped, assumptions that changed, or out-of-scope items that were included. If the implementation matches the intent exactly, write "Delivered as specified."]

### Known Limitations and Future Considerations

[List anything deliberately left out that may need to be addressed later, or constraints the current implementation imposes on future changes. If none, write "None identified."]

### Bolts That Delivered This Intent

| Bolt | Completed | Retro |
|---|---|---|
| [bolt-name](../../build/bolts/[bolt-file].md) | YYYY-MM-DD | [retro](../../operate/retros/[retro-file].md) |
