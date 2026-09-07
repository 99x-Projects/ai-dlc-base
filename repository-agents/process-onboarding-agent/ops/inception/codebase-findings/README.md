# Codebase Findings

Each file in this folder is the accumulated record of what the AI has learned by reading existing code — one file per module, service, or area of the codebase. It exists so that reverse-engineering done for one intent is never repeated for the next.

**Used during brownfield work:** whenever elaboration (or the Mature Project onboarding archaeology) requires analyzing existing code to understand how a new intent depends on, integrates with, or is constrained by a prior implementation.

**File naming:** `[module-or-area-slug].md` — one file per module/service/area, not per intent or per session. Findings about the same area accumulate in the same file over time.

**Workflow:**
1. **Before analyzing existing code** for a new intent, check the index below for a file covering the relevant module/area. If one exists, read it first — treat it as a starting point, not gospel, and verify it still matches the current code before relying on it (code drifts; findings can go stale).
2. **After analyzing code** not yet covered, or finding something that contradicts an existing entry, write or update the corresponding file. Append a new dated entry under "Findings" rather than overwriting prior entries — the history of what was true when is part of the record.
3. Update the index below whenever a file is created or an existing file's "Last updated" date changes.

Use `_template.md` to create a new finding file.

## Index

| Module / Area | File | Last updated | Status |
|---|---|---|---|
| — | — | — | — |

**Status values:** `Active` (reflects current code) · `Needs re-verification` (flagged as possibly stale) · `Superseded` (module was rewritten/removed — see linked replacement)
