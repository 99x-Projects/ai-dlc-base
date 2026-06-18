# Incident: [short title]

**Status:** Open | Investigating | Resolved | Closed
**Date detected:** YYYY-MM-DD HH:MM
**Date resolved:** YYYY-MM-DD HH:MM
**Severity:** P1 (critical) | P2 (major) | P3 (minor)
**Owner:** [Engineer name]

---

## Summary

[Two to three sentences. What broke, what the user impact was, and how it was resolved.]

---

## Timeline

| Time | Event |
|---|---|
| HH:MM | [What happened] |
| HH:MM | [Detection / alert] |
| HH:MM | [Investigation step] |
| HH:MM | [Fix deployed / resolved] |

---

## Root Cause

[What caused the incident. Be specific — not "human error" but what exactly was wrong and why it got past review.]

---

## Was AI-DLC a Contributing Factor?

- [ ] Yes — describe: [which unit, which AC was missing or wrong, which review step was skipped]
- [ ] No

---

## Immediate Fix

[What was done to resolve the incident. Include PR or commit reference if applicable.]

---

## Improvements Triggered

| # | Improvement file | Target rule/guideline |
|---|---|---|
| 1 | [ops/operate/improvements/YYYY-MM-DD-<unix_timestamp>-[slug].md](../improvements/YYYY-MM-DD-<unix_timestamp>-[slug].md) | [process-onboarding-agent/rules/...] |

---

## Root Cause Analysis

**RCA status:** Not run | In progress | Complete

**RCA report:** — *(link here once run)*

> When this incident is marked Resolved, the AI will suggest running the root-cause-analysis skill to surface deeper design, technology, or process gaps beyond the immediate fix. Run it by saying: "run root-cause-analysis on this incident."
