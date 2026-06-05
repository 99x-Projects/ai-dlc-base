# Skill: New Engineer Induction

**Purpose:** Guide a new engineer through their first AI-DLC session. Explains the framework using the project's actual files and domain language — not a generic script — and produces a personal quick-reference card they can use from their second session onward.

**Trigger:** Engineer-initiated when joining an AI-DLC project for the first time. Invoked by saying "I'm new to this project", "run new engineer induction", or "walk me through how to work here."

**Never run automatically.** Only runs when explicitly requested.

**Duration:** This session takes approximately 30–45 minutes. Tell the engineer this upfront and confirm they have the time before beginning.

---

## Step 1 — Welcome and Name

Ask:

> "Welcome. Before we start, what's your name? I'll use it to personalise the quick-reference card at the end."

Record the name. Then set expectations:

> "I'm going to walk you through how this project uses AI-DLC — the framework that governs how we work together. We'll cover: what the framework is, how the process runs, what you own versus what I do, and how to use the quality gate. At the end I'll create a quick-reference card you can keep.
>
> This takes about 30–45 minutes. If you'd prefer a faster version covering just the essentials, say 'quick tour' and I'll skip the practice section."

Wait for the engineer to confirm or request the quick tour before continuing.

---

## Step 2 — Read the Project's Framework Files

Before saying anything substantive, read the following without prompting:

- The master rule file (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`)
- `ai-dlc/guidelines/domain-glossary.md` (if it exists)
- `ai-dlc/rules/prompt-quality-gate.md`
- `ai-dlc/ops/build/backlog.md` (to understand current project state)

Do not present any of this content yet. Reading is preparation for the explanations in Steps 3–6.

---

## Step 3 — Explain the Framework

Walk the engineer through the master rule file section by section, in plain language. Do not read sections aloud — translate each one into a brief, conversational explanation. For each section, one paragraph maximum.

Structure your explanation as follows:

**What AI-DLC is (from Section 1):**
> "This project uses a framework called AI-DLC. It's not a tool — it's a set of agreements about how you and I work together. At its core: every feature starts with a written intent, every AI interaction is gated by a quality check, and every mistake feeds back into rules that prevent it from happening again. The framework lives in the `ai-dlc/` folder — you'll see it referenced throughout our conversations."

**The process loop:**
> "The work runs in a continuous loop: Inception (you write what you want to build and why) → Build (I help you break it into units and write the code) → Operate (you run retros after each batch and I turn the findings into rule improvements) → back to Inception. The loop compounds — every retro makes the next build better."

**The project's stack and domain (from Section 1 of the master rule file):**
Summarise the project identity section in two or three sentences using the actual content. Highlight the system boundaries (what must never happen architecturally).

**The domain language (from domain-glossary.md if present):**
Pick the three terms most likely to trip up a new engineer and explain them:
> "Three terms you'll encounter that have specific meaning in this project: [term 1] means [definition]. [term 2] means [definition]. [term 3] means [definition]. There are more in `ai-dlc/guidelines/domain-glossary.md` — read it before your first elaboration session."

**What you own (the engineer's responsibilities):**
> "Here's what I never do for you: I don't confirm that ACs are correct — you do. I don't decide whether the output is good enough to merge — you do. I don't run the tests — you do. I generate and assist; you verify and approve. If I produce something that passes all the checks but is still wrong for the business, that's a signal to update the rules — not to accept bad output."

Pause after this section and ask: "Any questions before we move to the quality gate?"

---

## Step 4 — Demonstrate the Quality Gate

Explain the quality gate using a concrete example from the project's domain.

**First, show a bad prompt:**
> "The quality gate has four components: Context, Constraints, Acceptance Criteria, and Output Format. If any are missing, I stop and ask — I don't guess. Here's what a bad prompt looks like in this project:"

Write an example incomplete prompt using a plausible feature for this project's domain (derived from the intent files or backlog):

```
Bad prompt example:
"Add a [domain-relevant noun] form."

What's missing:
- Context: Who needs this? Where does it sit in the system?
- Constraints: Which rules and conventions apply? (read code-standards.md)
- Acceptance Criteria: What must be true when it's done?
- Output Format: Which files? New component or update existing?
```

**Then show a complete prompt using the same feature:**

```
Complete prompt example:
Context:    [Engineer role] needs a [feature] on the [screen/module] so that [outcome].
            Stack: [project stack from master rule file]. Follows patterns in [relevant file].
Constraints: Read ai-dlc/rules/code-standards.md and ai-dlc/rules/security.md before generating.
             No inline styles. Auth check on every new endpoint.
ACs:        1. Given [precondition], when [action], then [outcome].
            2. Given [error condition], when [action], then [error handling outcome].
Output:     New file at [path]. Update [other file] to register it.
```

> "If you send me the bad version, I'll stop and ask for what's missing — every time, no exceptions. The gate exists because incomplete prompts produce output that looks correct but isn't."

---

## Step 5 — Practice Elaboration (skip if quick tour)

Run a short practice elaboration using a real feature from the project's intent files or backlog, or a simple hypothetical if nothing suitable exists.

Explain what you're about to do:

> "I'll now run a compressed elaboration session. This is how every feature starts. I'll propose one unit at a time — name and purpose — then its acceptance criteria, then edge cases, then observability. You respond as you would in a real session: confirm, push back, or ask questions. We'll do two units."

Run the elaboration turn structure for two units:
1. Propose unit name and one-sentence purpose. Stop.
2. Propose ACs as a numbered list. Stop.
3. Surface one or two edge cases. Stop.
4. Ask the observability questions (success signal, failure signal, alert threshold). Stop.
5. Move to the second unit and repeat.

After both units, present the sign-off summary table:

```
Unit | Purpose | ACs | Edge cases | Observability
[1]  | [...]   | [N] | [N]        | [status]
[2]  | [...]   | [N] | [N]        | [status]

This is what sign-off looks like. In a real session: confirm this and I create the unit files.
```

Ask: "What questions do you have about the elaboration process?"

---

## Step 6 — Explain the Retro and Improvement Loop

Give a brief explanation of the retro cycle:

> "After every bolt — a planned batch of units — we run a retrospective. I drive it: I synthesise the findings, propose rule changes, and apply approved changes to the framework files immediately. The important thing for you to know is: every finding that doesn't become a rule change will recur. The retro is not a feelings exercise — it's the mechanism that makes the AI get better at working in this specific project over time."

Point to the relevant files:
> "Retro template: `ai-dlc/ops/operate/retros/_template.md`. Improvement template: `ai-dlc/ops/operate/improvements/_template.md`. You don't need to fill these in manually — I do it during the retro session."

---

## Step 7 — Produce the Quick-Reference Card

Write a quick-reference card to `ai-dlc/guidelines/[engineer-name-slug]-quick-ref.md`. Use the engineer's name from Step 1 to personalise it. Populate every section from the project's actual files — not generic placeholders.

```markdown
# AI-DLC Quick Reference — [Engineer Name]
Generated: YYYY-MM-DD

---

## The Process Loop

Inception → Build → Operate → Improvements → (repeat)

- **Inception:** Write an intent file. Run mob elaboration.
- **Build:** Execute units one at a time. Review each output before moving on.
- **Operate:** Run retro after every bolt. Improvements are applied immediately.

---

## How to Invoke Each Ceremony

| What you want to do | What to say |
|---|---|
| Start a new feature | "Run a mob elaboration for the [intent name] intent" |
| Execute a unit | "Execute unit [name] from bolt [name]" |
| Plan a batch of work | "Plan a bolt from the open units in the backlog" |
| Run a retro | "Run a retro for bolt [name]" |
| Check project health | "Run process health" |
| Get a stakeholder update | "Generate a progress digest for [intent]" |
| Run UAT | "Run UAT for [intent]" |
| Audit dependencies | "Run dependency audit" |
| Get help (new) | "Run new engineer induction" |

---

## The Quality Gate — Four Components Required

Every prompt must have all four or I will stop and ask:

1. **Context** — who needs this, what it does, where it sits
2. **Constraints** — which rules apply (read `ai-dlc/rules/code-standards.md` first)
3. **Acceptance Criteria** — testable Given/When/Then statements
4. **Output Format** — which files, new or update existing

---

## What You Own (I Never Do These for You)

- Confirming ACs are correct before elaboration sign-off
- Reviewing output before merging — tests passing is not sufficient
- Running the tests
- Deciding whether a retro finding should become a rule change

---

## Key Domain Terms

[Top 5 terms from domain-glossary.md with one-line definitions, using the project's actual glossary]

- **[Term]:** [definition]
- **[Term]:** [definition]
- **[Term]:** [definition]
- **[Term]:** [definition]
- **[Term]:** [definition]

Full glossary: `ai-dlc/guidelines/domain-glossary.md`

---

## System Boundaries — Never Break These

[List the system boundary rules from the master rule file Section 1, verbatim]

- [boundary rule]
- [boundary rule]

---

## Key Files

| File | Purpose |
|---|---|
| `CLAUDE.md` (or `.cursorrules`) | Master rule file — governs every session |
| `ai-dlc/ops/build/backlog.md` | All units by status |
| `ai-dlc/rules/prompt-quality-gate.md` | Quality gate definition |
| `ai-dlc/rules/code-standards.md` | Coding conventions for this project |
| `ai-dlc/guidelines/domain-glossary.md` | Business term definitions |
| `ai-dlc/guidelines/edge-cases.md` | Known failure modes |
| `ai-dlc/ops/inception/dependency-map.md` | Intent dependency map |

---

## Common Mistakes

| Mistake | What happens | What to do instead |
|---|---|---|
| Sending an incomplete prompt | I stop and ask for the missing component | Include all four gate components |
| Accepting output without reviewing | Bugs reach the codebase | Run the review checklist before merging |
| Skipping the retro | Mistakes recur in the next bolt | Run the retro immediately after every bolt |
| Writing ACs that test implementation | Tests pass but behavior is wrong | ACs describe outcomes, not how the code works |
```

Confirm completion:

> "Quick-reference card written to `ai-dlc/guidelines/[engineer-name-slug]-quick-ref.md`. That's everything for the induction. You're ready to run a real elaboration session — say 'Run a mob elaboration for [intent name]' to begin. Come back to this card whenever you need a reminder."
