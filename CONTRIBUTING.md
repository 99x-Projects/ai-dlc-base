# Contributing to ai-dlc-base

Thanks for helping improve the 99x Intent Delivery Framework. This repo is the **base template** — the canonical source that gets copied into every project repo — so a change here reaches every team running the framework. That reach is exactly why changes go through review like any other PR.

There are two distinct ways to contribute, depending on where the improvement came from.

---

## 1. Promoting an improvement from a project retro

Most improvements originate inside a project that's already running the framework, surfaced through a bolt retro and classified by the `knowledge-promotion.md` skill as **generic** (would still be an improvement on a completely different project) rather than project-specific.

If that's you, don't write this PR from scratch — the skill has already drafted the exact change (target file, current text, proposed replacement, and why it generalizes), recorded in the source project's improvement file.

→ Follow the full protocol in [readme-content/contributing.md](readme-content/contributing.md), which covers classification, the drafted-change format, and what to include in the PR description (source project, retro link, the "why this generalises" reasoning).

## 2. Direct improvements to the base repo

For everything else — fixing an unclear instruction, adding a new skill, correcting a template, improving a diagram, tightening an agent's protocol — open a PR directly against this repo.

### Before you start

- Skim [readme-content/repository-contents.md](readme-content/repository-contents.md) to see what already exists and where it lives. Most content is under `repository-agents/*/` — each agent's `skills/`, `rules/`, `ops/` templates, and its bootstrap `.md` entry point.
- Check whether your change belongs here at all. The rule of thumb from the knowledge-promotion protocol applies here too: **if this would still be an improvement on a completely different project — different stack, domain, team size — it belongs in the base repo.** If it depends on one team's stack, domain, or conventions, it belongs in that project's generated `intent-execution-framework/`, not here.
- Read [RULES.md](RULES.md) before editing a skill, rule, template, or agent file. This repo's catalogues, diagrams, and routing tables reference each other by exact path and phrase, so most changes are multi-file changes — RULES.md maps out exactly which files move together (its "Sync Sets") so your PR doesn't leave the repo self-contradictory.

### What you can contribute

| Change | Where |
|---|---|
| New or improved skill | `repository-agents/process-onboarding-agent/skills/` |
| Fix or clarification to a rule (e.g. engagement monitoring) | `repository-agents/process-onboarding-agent/rules/` |
| Template change (intent, unit, bolt, retro, etc.) | `repository-agents/*/ops/**/_template.md` |
| Onboarding, diagnostics, migration, skills, or estimation agent protocol | The relevant `repository-agents/process-*-agent/*.md` |
| Documentation | `README.md`, `readme-content/*.md` |
| Repository governance (cross-reference rules, PR review checklist) | `RULES.md`, `pr-review.md` |

### Style and conventions

- Match the tone and structure of the file you're editing — these are instructions an AI agent reads and follows literally, not general prose. Ambiguity here becomes inconsistent behavior downstream.
- Prefer editing an existing skill or rule over adding a new one. A new file is justified when the concern is genuinely distinct from everything already in the catalogue (see [readme-content/skills-for-bespoke-processes.md](readme-content/skills-for-bespoke-processes.md) for how the skills catalogue is organized and classified).
- If you add or rename a skill or template, update every place that references it: the agent descriptions in [readme-content/agents.md](readme-content/agents.md), the file listing in [readme-content/repository-contents.md](readme-content/repository-contents.md), and the "What the Agent Creates" tree in the same file.
- Keep Mermaid diagrams (in `readme-content/agents.md`) in sync with any change to agent flow or artifact lifecycle.

### Testing your change

There's no build or test suite — the content *is* the product, so validate it the way it will actually be used:

1. Copy the relevant agent folder (e.g. `repository-agents/process-onboarding-agent/`) into a scratch project.
2. Run it against your AI assistant exactly as a user would (`"Read process-onboarding-agent/onboard.md and follow the instructions inside it."`).
3. Confirm the behavior change is what you intended and nothing else regressed — pay particular attention to skills that are shared across multiple entry points (e.g. `knowledge-promotion.md` is invoked from every retro).

### Opening the PR

1. Keep the PR focused on one change — one skill, one rule, one template. This repo has no CI to catch scope creep, so a tight diff is the reviewer's main tool.
2. In the description, state:
   - What the change is and why it generalizes (same bar as a knowledge-promotion draft, even if this isn't a promoted change).
   - What you tested it against (step 2 above).
3. Request review as normal. Promoted changes and direct changes go through the same review bar — nothing is merged without review, since every project's framework install inherits it.
4. Before requesting a human reviewer, run [pr-review.md](pr-review.md) against your own diff — it checks the mechanical parts of RULES.md (cross-reference sync, orphaned references, scope) so the human reviewer can focus on whether the change itself is good. Reviewers should run it too if the author hasn't.

---

## Reporting a gap without a fix

If you've spotted a gap but don't have a drafted fix, open an issue describing what's missing or unclear and which agent/skill it affects. A good bug report against this repo looks like a Diagnostics Agent finding: the observed gap, which file/behavior it's in, and what a project running the framework actually experienced as a result.
