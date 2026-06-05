# Intent Dependency Map

> Maintained by the AI during every mob elaboration session.
> Read before planning a bolt — see usage rules below.
>
> **Last updated:** —

---

## How to Read This Map

**Intent Dependencies** records which intents must be complete before another can begin. An intent depends on another when it requires a data model, API contract, or shared service that the prerequisite introduces.

**Shared Interfaces** records API endpoints, data entities, and service boundaries that cross intent boundaries — things owned by one intent but consumed by another.

Before planning a bolt, the AI reads this map and flags:
- Any prerequisite intent that is not yet Implemented
- Any units in the planned bolt that touch a shared interface owned by a different intent

---

## Intent Dependencies

| Intent | Depends on | Dependency type | Prerequisite status |
|---|---|---|---|
| — | — | — | — |

**Dependency types:**
- `Data model` — the intent requires an entity or schema introduced by the prerequisite
- `API contract` — the intent consumes an endpoint or interface defined by the prerequisite
- `Shared service` — the intent uses a module, utility, or service owned by the prerequisite

---

## Shared Interfaces

| Interface / Entity | Owned by intent | Consumed by intents | Notes |
|---|---|---|---|
| — | — | — | — |

---

## Update Log

| Date | Change | Elaboration session |
|---|---|---|
| — | — | — |
