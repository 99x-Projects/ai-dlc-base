# ai-dlc-base — Master Rule File

**This governs work on `ai-dlc-base` itself** — the base template repo. It is not one of the master rule files the onboarding agent generates for a *consumer* project; this repo doesn't run the framework, it publishes it. If you were sent here expecting the full 11-section project master rule file, see `repository-agents/process-onboarding-agent/setup-guide.md` instead — that's the specification for the file this one is not.

---

## 1. What this repo is

The source of truth for the 99x Intent Delivery Framework, copied into every project that adopts it. Its own catalogues, diagrams, and routing tables cross-reference each other by exact file path and phrase — see [RULES.md](RULES.md) for why that makes most changes here multi-file changes.

## 2. Before editing anything under `repository-agents/`, `README.md`, or `readme-content/`

Read [RULES.md](RULES.md) first. It defines the Placement Map (which category a file belongs to) and the Sync Sets (which other files must change alongside it). Skipping this step is the most common way a change here silently breaks the repo — a file gets renamed or a skill gets added without updating the four or five other places that reference it.

## 3. Before finalizing or approving a PR to this repo

Run [pr-review.md](pr-review.md) against the diff. It checks Sync Set consistency, orphaned references, and scope discipline, and returns a Ready to merge / Changes needed verdict. Do this even if you also plan a substantive content review — the two are complementary, not redundant.

## 4. How to contribute

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the full process (direct changes vs. promoted improvements from a project retro, PR description requirements, testing expectations).

## 5. Reference map

| Need | Read |
|---|---|
| What exists and where | `readme-content/repository-contents.md` |
| Cross-reference rules for editing this repo | `RULES.md` |
| PR review checklist | `pr-review.md` |
| Contribution process | `CONTRIBUTING.md` |
| Agent descriptions and diagrams | `readme-content/agents.md` |
| The spec this repo distributes | `repository-agents/process-onboarding-agent/setup-guide.md` |
