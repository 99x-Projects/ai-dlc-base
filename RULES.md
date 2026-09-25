# Repository Integrity Rules

← [Back to README](README.md) · See also [CONTRIBUTING.md](CONTRIBUTING.md) and [pr-review.md](pr-review.md)

---

## Why this file exists

`ai-dlc-base` is copied into every project that runs the framework, and its own catalogues, diagrams, and routing tables reference each other by exact file path and exact phrase. A change that looks like a one-file edit — adding a skill, renaming a rule, editing a template field — is usually a multi-file edit, because the same fact is documented in several places for several audiences (contributors, the onboarding agent, the diagnostics agent, engineers reading the README).

CONTRIBUTING.md tells you *how to raise a PR*. This file tells you *what must stay true across the repo* for that PR to be safe to merge. [pr-review.md](pr-review.md) is the checklist that turns these rules into a pass/fail review.

These rules apply to changes made **to `ai-dlc-base` itself**. They are not distributed to consumer projects — nothing here is copied by the onboarding agent.

---

## 1. The Placement Map

Every file in this repo belongs to exactly one category. Before editing, confirm which one you're in — it determines which Sync Set (§2) applies.

| Category | Path pattern | Source of truth for |
|---|---|---|
| Skill file | `repository-agents/process-onboarding-agent/skills/*.md` | One installable capability, copied verbatim into every project |
| Rule file | `repository-agents/process-onboarding-agent/rules/*.md` | A generic rule copied verbatim into every project |
| Ops template | `repository-agents/*/ops/**/_template.md` | The shape of an artifact (intent, unit, bolt, retro, etc.) generated in every project |
| Agent entry point | `repository-agents/process-*-agent/{onboard,migrate,estimate,role-play}.md` | The bootstrap trigger for one agent |
| Agent protocol/guide | `repository-agents/process-*-agent/*-guide.md` | The full protocol an entry point runs |
| Top-level reference doc | `readme-content/*.md` | Human-facing documentation of what exists and how it fits together |
| Root doc | `README.md`, `CONTRIBUTING.md`, `RULES.md`, `pr-review.md` | Entry points for a human landing on the repo |
| Base repo's own master rule file | `CLAUDE.md`, `.cursor/rules/ai-dlc-base-governance.mdc`, `.github/copilot-instructions.md` | Auto-loaded routing into RULES.md/pr-review.md/CONTRIBUTING.md when working *on* ai-dlc-base itself — all three must route an assistant to the same places and trigger the same behavior; name, location, frontmatter, and tool-specific wording may differ (see Sync Set E). **Not** the master rule file template the onboarding agent generates for consumer projects (that's `repository-agents/process-onboarding-agent/setup-guide.md`'s subject, a different artifact entirely). |

If a change doesn't fit any row — you're proposing a new category. Say so explicitly in the PR description; don't force it into an existing folder.

---

## 2. Sync Sets — files that must change together

A Sync Set is a group of files that encode the same fact from different angles. Touching one member without the others leaves the repo self-contradictory — the exact failure mode this file exists to prevent.

### Sync Set A — Adding, renaming, or removing a skill

Source: `repository-agents/process-onboarding-agent/skills/<name>.md`

When this changes, check every one of these:

1. `readme-content/repository-contents.md` — the file/folder reference table (one row per skill)
2. `readme-content/repository-contents.md` — the "What the Agent Creates" tree (`skills/` listing)
3. `repository-agents/process-skills-agent/skills-guide.md` — the full catalogue entry (name, invoke phrase, dependency classification `◆`/`◈`/`◇`, "what you get")
4. `readme-content/skills-for-bespoke-processes.md` — the skill count (currently "19 skills") and the classification lists (Standalone / Needs config / Framework-only)
5. `readme-content/agents.md` — the skill count reference and, if the skill changes agent flow, the Mermaid diagrams
6. `repository-agents/process-onboarding-agent/setup-guide.md` — the Section 6 reference-map entry (`**X skill:** read `{FRAMEWORK_ROOT}/skills/x.md` when...`) and the per-skill installation instructions (`Copy this file verbatim from process-onboarding-agent/skills/x.md to {FRAMEWORK_ROOT}/skills/x.md`)
7. CONTRIBUTING.md's "What you can contribute" table — only if this is a new *category* of skill, not a routine addition

A rename touches all seven as a path/phrase change; a removal touches all seven as a deletion; an addition touches all seven as a new entry.

### Sync Set B — Adding a new agent

Source: a new `repository-agents/process-<name>-agent/` folder

1. `README.md` — the "Agent Suite" table (name, what it does, entry point)
2. `readme-content/agents.md` — the detailed description and its place in the Mermaid flow diagrams
3. CONTRIBUTING.md's "What you can contribute" table, if the agent introduces a new kind of contribution

### Sync Set C — Changing an ops template field

Source: `repository-agents/*/ops/**/_template.md`

1. The template file itself
2. `repository-agents/process-diagnostic-agent/review-guide.md` — any rubric row that names the field (e.g. "AI Risk classification", "UAT Sign-off", "Breaking Changes Register", "Observability")
3. `repository-agents/process-onboarding-agent/setup-guide.md` — the section describing when/how the field gets populated
4. `readme-content/repository-contents.md` — the template's one-line description, if its purpose changed

### Sync Set E — Changing the base repo's own master rule file

Source: any of `CLAUDE.md`, `.cursor/rules/ai-dlc-base-governance.mdc`, `.github/copilot-instructions.md`

These three must stay **behaviorally equivalent, not textually identical**: same sections, same triggers, same routing targets (RULES.md, pr-review.md, CONTRIBUTING.md, setup-guide.md), so that Claude, Cursor, and Copilot all send their assistant to the same place and the same next action when working on this repo. Byte-identical text is the easiest way to guarantee that and remains the default — keep it unless there's a concrete, tool-specific reason to diverge. The standing example of a legitimate reason is Cursor's: its required frontmatter (`description`/`globs`/`alwaysApply`) and its repo-root-relative links (`../../RULES.md` etc., since it lives under `.cursor/rules/`). A wording change that only suits how one tool's assistant parses instructions, without changing what it's told to do, is allowed. Editing one file's routing content — a section added or removed, a target changed, a trigger changed — without carrying the same change to the other two is a Blocking finding, regardless of whether the wording ends up identical.

### Sync Set D — Changing a rule file

Source: `repository-agents/process-onboarding-agent/rules/*.md`

1. The rule file itself
2. `readme-content/repository-contents.md` — the description row
3. `repository-agents/process-onboarding-agent/setup-guide.md` — where the rule is referenced or generated
4. `repository-agents/process-diagnostic-agent/review-guide.md` — if the diagnostics agent audits this rule's presence or quality

---

## 3. Naming & structural conventions

- Agent folders: `repository-agents/process-<name>-agent/`.
- Skill and rule files: `kebab-case.md`, named after the capability, not the ticket or author.
- Every agent has one bootstrap entry point (verb-named: `onboard.md`, `migrate.md`, `estimate.md`, `role-play.md`) and, where the protocol is long, a companion `*-guide.md` the entry point routes to — don't inline a long protocol into the entry point itself.
- An "How to invoke" phrase quoted anywhere in the repo must match the file's actual path exactly. If you move a file, grep the whole repo for its old path and old invoke phrase before you're done — an orphaned instruction is worse than no instruction, because it fails silently in front of a user.
- Skill files under `process-onboarding-agent/skills/` are copied **verbatim** into every project — don't add project-specific branches or TODOs inside them.

---

## 4. Scope and classification discipline

- One change per PR — one skill, one rule, one template, one agent. This repo has no CI to catch scope creep (§ Testing in CONTRIBUTING.md), so a tight diff is the only thing that makes a Sync Set check tractable.
- Before adding content to `repository-agents/*/skills/`, `rules/`, or `ops/**/_template.md`, apply the generic test from CONTRIBUTING.md and `knowledge-promotion.md`: *if this were applied to a completely different project — different stack, domain, team size — would it still be an improvement?* If no, it belongs in a project's own generated `intent-execution-framework/`, not here.
- Match the tone and structure of the file you're editing. These are instructions an AI agent reads and follows literally; ambiguity here becomes inconsistent behavior downstream, not a matter of style preference.

---

## 5. Hard stops — never do these

- Never rename or move a file under `repository-agents/` without grepping the entire repo for every reference to its old path and updating each one in the same PR.
- Never delete a file that anything else links to without first updating or removing those references — no orphaned links.
- Never add a skill, rule, or template edit that only makes sense for one team's stack or domain (see §4's generic test).
- Never leave an "How to invoke" phrase, file count, or catalogue entry out of sync with the file it describes.
- Never bundle a skill/rule/template change with an unrelated documentation change in the same PR — split them so a Sync Set check applies cleanly.

---

## Using these rules

- **Contributing?** Read CONTRIBUTING.md first for process, then use §1–§2 here to find every file your change touches before you open the PR.
- **Reviewing?** Use [pr-review.md](pr-review.md) — it turns §1–§5 into a checklist against an actual diff and returns a Ready to merge / Changes needed verdict.
