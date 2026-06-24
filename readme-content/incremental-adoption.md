# Incremental Adoption Activities

← [Back to README](../README.md)

---

## Not Ready to Onboard Yet? Start Here.

Full adoption of the 99x Intent Delivery Framework — with onboarding, a master rule file, and governance ceremonies — delivers the most value, but it requires a deliberate commitment that not every team is ready to make. If your team is still evaluating, or wants to prove value before committing, the activities below are a low-risk starting point.

Each activity plugs into a ceremony you already run. No new framework is imposed, no files need to be installed, and no onboarding session is required. Pick two activities from Phase 1, run them for a sprint or two, and measure the lift. Then add the next.

Phase 3 activities are the natural signal that a team is ready to graduate to full 99x Intent Delivery Framework adoption — by the time you reach them, you will have already reproduced most of the framework's core behaviors organically.

---

### Phase 1 — Easy wins: add AI to what you already do

No process change required. These activities slot into existing ceremonies and produce immediate, visible output.

| # | Activity | What the AI does | Builds toward |
|---|---|---|---|
| 1 | **Meeting → stories and tasks** | Turns a planning or discovery transcript into draft stories, tasks, and open questions. | Intent capture |
| 2 | **Story polish** | Rewrites a rough story in intent style — what it is, why it matters, and what success looks like. | Intent-style tickets |
| 3 | **AC and NFR enhancement** | Expands a story into testable acceptance criteria plus non-functional requirements: performance, security, accessibility, and observability. | Testable ACs |
| 4 | **Status digest** | Turns ticket moves and commits into a plain-language stakeholder update. | progress-digest skill |
| 5 | **PR summarizer** | Writes the "what changed and why" on each pull request so reviewers start faster and with full context. | Build hygiene |

---

### Phase 2 — Strengthen the ceremony: AI adds judgment, not just text

These activities deepen the quality of decisions your team already makes. The AI does not just produce text — it challenges assumptions and surfaces risk.

| # | Activity | What the AI does | Builds toward |
|---|---|---|---|
| 6 | **Edge-case surfacing** | Before story sign-off, asks "what could break or go unhandled?" on each story. | Elaboration discipline |
| 7 | **Code-review gate** | Checks diffs for security gaps, missing error handling, and test coverage before merge. | Quality gate |
| 8 | **Test cases from ACs** | Drafts test cases and a UAT demo script directly from the acceptance criteria. | UAT sign-off |
| 9 | **AI-assisted retro** | Takes retro notes, clusters themes by type, ranks them, and proposes concrete actions with suggested owners. | Closed-loop retros |
| 10 | **5-Whys on incidents** | Runs root-cause analysis on a production incident and classifies the cause as design / technology / process. | root-cause-analysis skill |
| 11 | **Backlog grooming assist** | Flags vague, duplicate, or oversized tickets and suggests how to split or sharpen them. | Unit decomposition |

---

### Phase 3 — Approaching AI-DLC discipline: the loop starts to close

By this phase, the team is running a recognizable version of the AI-DLC cycle. Completing items 12–17 is the practical signal that formal onboarding will land on fertile ground rather than on an unfamiliar process.

| # | Activity | What the AI does | Builds toward |
|---|---|---|---|
| 12 | **Retro → applied change** | Drafts the actual rule or checklist update so a retro finding becomes an applied change within the same sprint. | Improvement workflow |
| 13 | **Design-session assist** | Helps the team lock API contracts, data models, and architectural constraints before any building begins. | Design session |
| 14 | **Risk and blast-radius check** | Before a sizeable change, assesses impact on other components, rollback options, and whether a feature flag is needed. | Bolt risk assessment |
| 15 | **Dependency and security audit** | Reads package manifests monthly, lists risky or outdated packages, and suggests remediation steps. | dependency-audit skill |
| 16 | **New-joiner induction** | Walks a new engineer through the codebase and produces a personalized quick-reference card. | new-engineer-induction skill |
| 17 | **Capture conventions as a prompt** | Seeds a `CLAUDE.md` or `.cursorrules` with the team's actual coding standards and domain terms — the first step of a real onboarding. | Master rule file → full framework adoption |

---

When your team reaches item 17 and has a conventions file in place, you are one session away from full AI-DLC adoption. At that point, copy `repository-agents/process-onboarding-agent/` into your repo and follow the [onboarding instructions](../README.md#onboarding-a-new-project) — the archaeology path will read what you have already built and extend it rather than starting from scratch.
