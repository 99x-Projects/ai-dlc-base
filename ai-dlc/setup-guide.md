# AI-DLC Setup Guide

This document explains how to set up the AI-DLC framework in a repository. It works with Claude Code, Cursor, and GitHub Copilot. Everything here is generic — substitute your project's technology stack, domain terms, and rules where indicated.

---

## What You Are Building

AI-DLC is a structured operating system for building software with AI assistance. It is not a tool — it is a set of files, conventions, and ceremonies that govern how your team and your AI work together. The output is a repo where:

- Every feature starts with a written intent and testable acceptance criteria
- Every AI interaction is gated by a quality check and logged for audit
- Every failure feeds back into rules that prevent recurrence
- Any engineer (or AI tool) opening the repo knows exactly how to work in it

---

## Before You Begin

### Question 1 — Which AI tool are you using?

This guide uses the term **master rule file** to refer to the file that governs AI behaviour in your repository. Each tool has a different name and location for this file:

| AI Tool | Master rule file | Location |
|---|---|---|
| **Claude Code** | `CLAUDE.md` | Repo root |
| **Cursor** | `.cursorrules` | Repo root |
| **GitHub Copilot** | `copilot-instructions.md` | `.github/` folder |

The content of the master rule file is identical across tools. The only differences are the file name, the location, and — for GitHub Copilot — internal links to `ai-dlc/` files must use the prefix `../ai-dlc/` since the file lives inside `.github/`.

All subsequent steps in this guide refer to the "master rule file." Substitute the correct name and path for your chosen tool.

> If you want to support more than one AI tool in the same repo, write the master rule file once and create copies for each additional tool. See Step 8.

---

### Question 2 — Fresh or Mature Project?

Before following this guide, the AI must ask the engineer one question:

> **"Is this a fresh project (no significant codebase yet) or a mature project (existing code, conventions, and team practices already in place)?"**

- **Fresh project** → proceed directly to Step 1 below.
- **Mature project** → complete the Mature Project Onboarding phases (Phase M1–M3) first, then continue to Step 1.

---

## Mature Project Onboarding

Onboarding AI-DLC into an existing codebase requires three preparatory phases before any Bolt is written. The goal is to extract what the system already is, encode it as rules, and establish safety controls — so that AI-DLC enhances the project without disrupting what already works.

---

### Phase M1 — Codebase Archaeology Analysis

#### M1.0 — Scope Agreement (Required Before Any Analysis)

Large codebases cannot be analysed in a single context window. Before reading any code, the agent must ask the engineer to define the analysis scope.

Ask the engineer:

> "This codebase may be too large to analyse in one pass without overloading context. To keep each analysis pass focused and accurate, I'd like to work through it in segments.
>
> Please tell me:
> 1. Which modules, services, or folders are the highest priority for AI-DLC onboarding?
> 2. Are there any areas I should skip entirely for now (e.g. legacy code not being actively worked on, third-party code, generated files)?
> 3. Should I analyse one segment at a time and report findings before moving to the next, or would you prefer a summary across all agreed segments?"

Record the engineer's answers. Use them to define **analysis segments** — named, bounded slices of the codebase (e.g. "auth service", "payments module", "shared UI components"). Each segment is analysed independently across M1.1–M1.4, with findings reported to the engineer before moving to the next segment.

**Rules for segmented analysis:**
- Never read beyond the agreed segment boundary in a single pass
- After completing each segment, present a findings summary and ask the engineer to confirm before continuing to the next segment
- If a segment itself is too large for one context window, ask the engineer to break it down further before proceeding
- Keep a running segment log: `Segment | Status | Key findings` — update it after each segment completes

Only proceed to M1.1 once at least one segment is agreed.

---

#### M1.1 — Architecture Mapping

- Read the codebase module by module (within the agreed segment)
- Generate business-language descriptions of each module (what it does, not how)
- Produce a capability map: what the system does as a whole
- Identify service boundaries, integration points, and data flows
- Output becomes the foundation for the Bolt Backlog

#### M1.2 — Pattern Extraction

- Read 10–20 representative files across modules
- Extract: naming conventions, error handling style, ORM usage, API response shapes, authentication patterns, test patterns and coverage conventions
- Document all extracted patterns — these go into the master rule file and the `rules/` folder before any Bolt is written

#### M1.3 — Due Diligence Audit

Before classifying work, audit the existing codebase for defects and structural problems that new AI-generated code could inherit. This step must complete before any Bolt is planned.

**What to look for:**

| Category | Examples |
|---|---|
| **Logic defects** | Incorrect business logic, off-by-one errors, wrong conditional branches, silent data loss |
| **Design violations** | Responsibilities mixed across layers, circular dependencies, God classes or functions doing too much |
| **Security gaps** | Unvalidated input, missing auth checks, secrets in code, direct DB calls from the wrong layer |
| **Fragile patterns** | Catch-all error suppression, hardcoded values that should be config, mutable shared state |
| **Test blind spots** | Code paths with no test coverage; tests that assert implementation details rather than behaviour |
| **Consistency breaks** | Naming or structural conventions that differ across modules with no documented reason |

**Output:** A ranked list of findings. For each finding, record:
- Location (file/module/function)
- Category from the table above
- Impact if inherited by new code
- Recommended correction (fix-in-place, quarantine, or encode as a prohibition in the master rule file)

Present the list to the engineer and agree on which items must be corrected before the first Bolt runs and which can be logged as Remediation Bolts.

#### M1.4 — Debt and Gap Mapping

Classify all identified work — including findings from M1.3 — into three Bolt types:

| Bolt Type | Description |
|---|---|
| **Enhancement Bolt** | New capabilities not yet in the system |
| **Remediation Bolt** | Tech debt, coverage gaps, refactoring, and defects found in M1.3 |
| **Migration Bolt** | Architectural changes that enable future work |

**Prioritisation order:** Remediation of blocking defects found in M1.3 first → Enhancement (shows value) → Remediation of non-blocking debt → Migration last.

---

### Phase M2 — Repository Overlay

Create the governance layer on top of the existing codebase. All four artefacts must exist before the first Bolt runs.

#### M2.1 — Master Rule File

The most important file in the overlay. See **Before You Begin** for the correct file name and location for your AI tool. Must contain:
- Architecture context derived from M1.1
- Coding conventions extracted in M1.2
- Forbidden zones (see M2.2)
- Bolt workflow rules
- Entry points (see M2.3)

Follow the full master rule file authoring instructions in Step 2 of this guide, but populate each section from the archaeology output rather than from scratch.

#### M2.2 — Forbidden Zones

Create `ai-dlc/guidelines/forbidden-zones.md` containing an explicit list of files, modules, or patterns that the AI must **not** modify without senior engineer approval. This protects stable, business-critical code from accidental change.

```markdown
| Zone | Path / Pattern | Reason | Approval required from |
|---|---|---|---|
| <name> | <path or glob> | <why it is protected> | <role> |
```

Reference this file in the master rule file under Code Rules (Section 3) so it is enforced in every session.

#### M2.3 — Entry Point Registry

Create `ai-dlc/guidelines/entry-points.md` containing the approved list of modules and features where AI-DLC Bolts may begin. This controls the expansion boundary. Update the list as the team gains confidence with the process.

```markdown
| Module / Feature | Status | Notes |
|---|---|---|
| <name> | Approved / Pending / Blocked | <any constraints> |
```

#### M2.4 — Coding Conventions File

Create `ai-dlc/rules/code-standards.md` (or populate it if it already exists) entirely from the patterns extracted in M1.2. Every AI session must match the style of the existing codebase — this file is the authoritative source injected into each session.

---

### Phase M3 — Blast Radius Management

Before any Bolt is executed, three safety controls must be in place. These apply to every Bolt that touches existing code.

#### M3.1 — Test Coverage Gate

Verify that adequate test coverage exists for the target feature or module before a Bolt begins. If coverage is insufficient, a Remediation Bolt to add tests must be planned and executed first. No Bolt modifying existing code starts without this gate passing.

#### M3.2 — Feature Flag Requirement

Every change introduced by AI-DLC into an existing module must be wrapped in a feature flag. This limits production impact if a change behaves unexpectedly and allows rollback without a deployment.

#### M3.3 — Default Acceptance Criterion for Existing-Code Bolts

Every Bolt that affects existing code must carry the following AC by default. It has two forms depending on Bolt classification:

**Standard form** (Enhancement Bolts and all Bolts not classified as Migration or Remediation):
> **"All integration tests for [affected module] pass without modification."**

This AC is non-negotiable and cannot be removed during elaboration.

**Contract-change form** (Migration Bolts and Remediation Bolts that explicitly change contract boundaries — e.g. API shapes, data schemas, inter-module interfaces):
> **"All integration tests for [affected module] pass without modification, except for tests that cover the contract boundaries listed as breaking changes below. Each breaking change must be detailed and approved in the elaboration session before any code is generated."**

When the contract-change form applies, the elaboration session must produce a **Breaking Changes Register** — a table attached to the unit file listing every changed contract boundary, the reason it must change, and the name of the engineer who approved it. No unit using the contract-change form may be executed without a completed and approved Breaking Changes Register.

Add both forms to the master rule file Section 6 (AI-DLC Workflow) and to `ai-dlc/rules/code-standards.md` so the correct form is enforced automatically based on Bolt classification.

---

### Mature Project Onboarding Checklist

- [ ] Architecture map produced (capability map, service boundaries, data flows)
- [ ] Pattern extraction complete (naming, error handling, ORM, API shapes, test conventions)
- [ ] Debt and gap map produced; work classified into Enhancement / Remediation / Migration Bolts
- [ ] Master rule file written from archaeology output (see Before You Begin for file name)
- [ ] `ai-dlc/guidelines/forbidden-zones.md` created and referenced in the master rule file
- [ ] `ai-dlc/guidelines/entry-points.md` created
- [ ] `ai-dlc/rules/code-standards.md` populated from extracted patterns
- [ ] Test coverage gate verified for first target module
- [ ] Feature flag approach confirmed with team
- [ ] Default AC for existing-code Bolts added to the master rule file and `code-standards.md`

Once all items above are checked, continue to **Step 1** of this guide to create the full folder structure and remaining artifacts.

---

## Fresh Project — Structured Interview

For fresh projects the agent cannot read a codebase to populate the master rule file. Instead, it must run a structured interview before creating any files. The interview is strictly one question at a time, in this order. The agent must not proceed to the next question until the engineer has answered the current one.

**Do not present all questions at once. Ask them one at a time and wait for the answer.**

---

**Interview sequence:**

1. **Product identity**
   > "In one sentence, what does this product do and who uses it?"

2. **Technology stack — backend**
   > "What language and framework will the backend use? (e.g., Node/Express, Python/FastAPI, .NET/ASP.NET Core)"

3. **Technology stack — frontend**
   > "What language and framework will the frontend use? (e.g., React/TypeScript, Vue, server-rendered only)"

4. **Database and auth**
   > "What database will you use, and how will authentication work? (e.g., PostgreSQL with Supabase Auth, MongoDB with JWT)"

5. **System boundaries**
   > "Are there any hard rules about what must never happen architecturally? (e.g., 'the frontend must never call the database directly', 'the mobile app communicates only through the REST API')"

6. **Domain language**
   > "List the key business terms this system uses — the words that appear in your domain, not generic tech terms. For each one, give a one-sentence definition. (e.g., 'Booking — a confirmed reservation between a guest and a host')"
   >
   > *Keep asking "any more?" until the engineer says done.*

7. **Known constraints and prohibitions**
   > "Are there any technical choices that are already decided and must not be changed by the AI? (e.g., 'we must use REST, not GraphQL', 'no ORM — raw SQL only', 'all prices stored as integers in cents')"

8. **First capability**
   > "What is the first feature or capability you want to build? Give it a name and one sentence describing what it does for the user."

9. **Documentation archive threshold**
   > "AI-DLC generates operational documents over time (retros, improvement files, unit files, bolt files). These can be compacted and archived periodically using the compact-docs skill. How many months old must a document be before it qualifies for archiving? (Common choices: 3, 6, or 12 months. This can be changed later.)"

---

Once all nine questions are answered, the agent has enough to:
- Create the folder structure (Step 1)
- Write the master rule file with Sections 1–5, Section 8, and Process Configuration fully populated
- Write an initial first intent file from the answer to question 8
- Flag Sections 6 and 7 (workflow and review) as pre-populated from the guide defaults

---

## Step 1 — Create the Folder Structure

Create this directory tree at the root of your repository:

```
ai-dlc/
  Instructions2FDE.md        ← main guide for engineers
  README.md                  ← how the process achieves quality
  rules/
    prompt-quality-gate.md   ← the 4-component gate run before every code generation
    code-standards.md        ← language/framework conventions and anti-patterns
    security.md              ← never/always security rules
    architecture.md          ← ADRs — decisions and their rationale
    engagement.md            ← engineer engagement monitoring signals and intervention protocol
  skills/
    mob-elab-prompts.md      ← interactive protocol and prompts for elaboration sessions
    review-checklist.md      ← structured lens for reviewing AI output
    unit-template.md         ← how to write a unit (reference doc)
    compact-docs.md          ← engineer-triggered skill to archive old operational documents
  guidelines/
    domain-glossary.md       ← canonical business terms used in code and prompts
    edge-cases.md            ← known failure modes to check before generating code
    acceptance-patterns.md   ← rules and anti-patterns for writing ACs
    dev-setup.md             ← environment setup checklist for new engineers
    team-rollout.md          ← scaling to multiple engineers
  prompts/                   ← one file per feature; every AI session logged here
  ops/
    inception/
      intents/               ← one file per feature intent
        _template.md
        README.md
      elaborations/          ← one folder per intent; one file per session
        _template.md
    build/
      backlog.md             ← master status of all units
      units/                 ← one file per atomic unit of work
        _template.md
      bolts/                 ← one file per planned batch of units
        _template.md
    operate/
      retros/                ← one file per completed bolt
        _template.md
      incidents/             ← one file per production issue
        _template.md
      improvements/          ← one file per process change triggered by retro/incident
        _template.md
```

---

## Step 2 — Write the Master Rule File (The Behavioral Contract)

The master rule file is loaded automatically by your AI tool every session. See **Before You Begin** for the correct file name and location for your tool — the content is the same regardless of which tool you use. Every section below is required.

### Section 1 — Project Identity

State what the system is and its technology stack. Be specific — the AI needs to know:
- What the product does (one sentence)
- Backend language/framework and key patterns
- Frontend language/framework
- Database and auth approach
- **System boundaries** — what calls what; what must never happen (e.g. "frontend never calls the database directly")

```markdown
## 1. Project Identity

**<ProjectName>** is a <one-sentence description>.
- Backend: <language/framework, key patterns>
- Frontend: <language/framework>
- Database: <database, auth approach>

**System boundaries:**
- <boundary rule 1>
- <boundary rule 2>
```

### Section 2 — Prompt Quality Gate

A two-line routing entry. Do not embed the gate definition — it lives in `rules/prompt-quality-gate.md`.

```markdown
## 2. Prompt Quality Gate

Before every code response: read and enforce `ai-dlc/rules/prompt-quality-gate.md`.
If any of the four components (Context · Constraints · Acceptance Criteria · Output Format) is missing, stop and ask for it. Do not generate code.
```

### Section 3 — Code Rules

Three to five **hard-stop prohibitions** that the AI must have in working memory (no file lookup tolerated) — these are the rules where a single violation causes immediate, serious harm. Then a mandatory-read routing line for everything else.

```markdown
## 3. Code Rules

**Hard stops — memorise, never look up:**
- Never commit secrets, API keys, or connection strings
- Never trust client-supplied IDs without server-side ownership verification
- Never expose internal stack traces to the client
- [1–2 stack-specific absolute prohibitions from the project interview]

**Full rules (read before writing any code):**
- Conventions and patterns: `ai-dlc/rules/code-standards.md`
- Security rules: `ai-dlc/rules/security.md`
- Architecture decisions: `ai-dlc/rules/architecture.md`
```

Keep the hard-stop list to five items maximum. Anything beyond five belongs in `code-standards.md`, not here.

### Section 4 — Domain Language

A mandatory-read instruction plus the two or three terms most likely to cause logic errors if misused. The full glossary lives in `guidelines/domain-glossary.md` — do not reproduce it here.

```markdown
## 4. Domain Language

Read `ai-dlc/guidelines/domain-glossary.md` before every elaboration session and before generating any business logic. Use only the terms defined there — do not substitute synonyms.

**Critical terms (load immediately):**
- **[Term]:** [one-line definition]
- **[Term]:** [one-line definition]
```

Limit inline terms to three. If the project has more critical terms, add them to the glossary file, not to this section.

### Section 5 — Known Edge Cases

A single mandatory-read routing line. No inline table — edge cases are added continuously via the retro loop and must stay in their dedicated file to remain current.

```markdown
## 5. Known Edge Cases

Read `ai-dlc/guidelines/edge-cases.md` before generating code for any unit. Do not skip this step — new edge cases are added after every retro.
```

### Section 6 — AI-DLC Workflow

The turn structure is short enough to hold in working memory — keep it inline. Everything else routes to files.

```markdown
## 6. AI-DLC Workflow

**Elaboration turn structure (strictly one unit per turn):**
1. Propose one unit — name and one-sentence purpose only. Stop.
2. Propose ACs as a numbered list. Stop.
3. Surface edge cases and open questions. Stop.
4. Move to next unit. Repeat.
5. After all units agreed, present summary table and ask for sign-off before writing any files.

**Full elaboration protocol:** read `ai-dlc/skills/mob-elab-prompts.md` before every elaboration session.
**Compact-docs skill:** read `ai-dlc/skills/compact-docs.md` when the engineer invokes it.
**Engagement monitoring:** read and apply `ai-dlc/rules/engagement.md` throughout all ceremonies.
```

### Section 7 — Review Behavior

A single routing line. The checklist lives in its file.

```markdown
## 7. Review Behavior

Before presenting any output, run every item in `ai-dlc/skills/review-checklist.md`. Do not present output that has not passed this checklist.
```

### Section 8 — Reference Map

A table mapping common needs to their files. Engineers and the AI both use this to navigate the framework.

### Section 9 — Process Configuration

A single table of project-level process settings that govern AI-DLC behaviour. Populate from the setup interview answers. Every value here can be changed by the project lead at any time by editing this section.

```markdown
## 9. Process Configuration

| Setting | Value | Notes |
|---|---|---|
| **Archive threshold** | [X] months | Documents older than this qualify for archiving via the compact-docs skill |
```

The archive threshold is read by the `compact-docs` skill at runtime. If this section is absent, the skill will ask the engineer for the value before proceeding.

---

## Step 3 — Write the Rules Files

### `rules/prompt-quality-gate.md`

Defines the four components in detail with examples of complete and incomplete requests. Include:
- What each component means
- The order to ask for missing components
- An example of an incomplete request and the correct response
- An example of a complete request

### `rules/code-standards.md`

Document your stack's patterns and anti-patterns. Key sections:
- Languages & runtimes
- Naming conventions (per language)
- Framework patterns (backend and frontend)
- Formatting & linting tooling
- Testing requirements and coverage thresholds

**Critically:** add anti-patterns discovered through actual failures — e.g. framework methods that look correct but have unit-testing limitations. These turn retro findings into permanent rules.

### `rules/security.md`

Two sections:
- **Never do these** — injection, auth gaps, secrets, data exposure
- **Always do these** — input validation, auth on every endpoint, least privilege

Add a section for your specific platform's security quirks (e.g. Supabase RLS, AWS IAM patterns, OAuth flows).

### `rules/architecture.md`

One ADR per significant decision. Format:

```markdown
### ADR-001 — <Decision title>
**Decision:** <what was decided>
**Why:** <the reasoning>
**Trade-off:** <what you gave up>
```

Add an ADR whenever a new cross-cutting decision is made — especially ones where the AI might make a different choice if not told (e.g. symmetric vs asymmetric JWT signing, SSR vs SPA, monorepo vs polyrepo).

### `rules/engagement.md`

Copy this file verbatim from the base repo (`ai-dlc/rules/engagement.md`). It defines the signals of engineer disengagement, when to intervene, what to say, and how to handle continued non-engagement. The master rule file Section 6 routes to it with a single mandatory-read line — do not inline its contents.

---

## Step 4 — Write the Skills Files

### `skills/mob-elab-prompts.md`

The mob elaboration reference. Must include:
- The mandatory interactive protocol (turn structure + never-do rules)
- Facilitation prompts for: opening a session, proposing ACs, edge case check, generating API contract, generating implementation scaffold, reviewing output

### `skills/review-checklist.md`

Structured review sections covering all five AI failure modes:

| Section | Failure mode covered |
|---|---|
| Functional Correctness | Logic errors that look correct |
| Code Quality | Over-engineering; dead code |
| Security | Security vulnerabilities |
| Architecture | Architectural drift |
| Tests | Logic errors; hallucinations |
| AI-Specific Checks | Hallucinated library calls; prompt log; scope creep |
| Deployment Readiness | Configuration errors; breaking changes |

Key items that must be present:
- Feature verified in a real environment — tests passing alone is not sufficient
- Nothing in the diff beyond what the ACs required (no extra abstractions or future-proofing)
- Diff checked against system boundaries in `architecture.md`
- Behavioral trade-offs confirmed before accepting output
- For wrapper/layout components: existing files grepped for patterns the new component will duplicate before generation

### `skills/compact-docs.md`

The compact-docs skill is engineer-triggered and must never run automatically. It archives operational documents older than the project's configured threshold to keep the active workspace manageable without losing institutional memory.

Copy this file verbatim from the base repo (`ai-dlc/skills/compact-docs.md`). No customisation is needed — the archive threshold is read from the master rule file Process Configuration section at runtime.

---

## Step 5 — Write the Guidelines Files

### `guidelines/domain-glossary.md`

Full definitions for every domain term. Each entry: term, definition, usage notes, and related terms. The glossary in the master rule file is a summary — this file is the authoritative source.

### `guidelines/edge-cases.md`

Full descriptions of each known edge case: the scenario, the required behavior, and which parts of the system must handle it. Start with the ones most relevant to your domain. Add entries from retros and incidents.

### `guidelines/acceptance-patterns.md`

Rules for writing good Given/When/Then ACs:
- One behavior per criterion (no compound ACs)
- Name the actor in every Given
- Cover at least one unhappy path per unit
- No implementation details in ACs
- Anti-patterns to avoid (vague outcomes, testing implementation not behavior)


### `guidelines/dev-setup.md`

Step-by-step environment setup for a new engineer:
- All prerequisites with version checks
- Configuration files needed and what goes in them (no actual secrets — explain how to get them)
- How to start each service
- Auth verification steps
- Secrets hygiene checklist

### `guidelines/team-rollout.md`

Covers: git branching conventions, environment isolation, secrets management, backlog ownership, multi-FDE Bolt rhythm, PR checklist, merge conflict resolution (including AI-assisted resolution), team leader review guidelines, and common anti-patterns.

---

## Step 6 — Write the Ops Templates

Each template file defines the structure for its artifact type. The key templates:

### `ops/inception/intents/_template.md`
Fields: Status, Date, Owner, What, Why, Success Looks Like, Assumptions, Open Questions, Out of Scope, Elaboration Sessions, Extracted Units, Implementation Summary.

The **Implementation Summary** section is written by the AI once all units under the intent have been delivered and merged. It must not be filled in earlier. It contains four subsections:

1. **What Was Built** — user-facing description of every feature delivered under the intent; one paragraph per distinct feature
2. **How It Works (Key Design Decisions)** — data model choices, API contracts, edge cases explicitly handled, and non-obvious implementation facts a future engineer must know before modifying the feature
3. **Scope Delivered vs. Original Intent** — any deviations from the original intent (descoped items, changed assumptions, additions); write "Delivered as specified" if none
4. **Known Limitations and Future Considerations** — constraints the current implementation imposes on future changes; write "None identified" if none

When the last unit of an intent is confirmed done, the AI must:
1. Set the intent status to **Implemented**
2. Write the Implementation Summary by reading the elaboration session files, unit files, and bolt retros for this intent
3. Ask the engineer to review the summary before closing the intent

The Implementation Summary is the authoritative reference for future Bolts that modify or extend this feature. Any Bolt touching a feature covered by an intent must read its Implementation Summary before elaboration begins.

### `ops/build/units/_template.md`
Fields: Status, Intent link, Elaboration link, Bolt link, Priority, Context, Acceptance Criteria, Scope (in/out), Dependencies, Pre-generation Checks, Edge Cases to Handle, Definition of Done, Prompt Log link, Notes.

The **Pre-generation Checks** section is critical for wrapper/layout units — list grep patterns to run across existing files before generating to surface duplication.

### `ops/build/bolts/_template.md`
Fields: Status, Goal, Start/Target/Completed dates, Units table, Execution Order diagram, Risks & Assumptions, Definition of Done, Retrospective link.

### `ops/operate/retros/_template.md`
Sections: What Went Well, What Didn't Go Well, AI-Specific Observations (prompts that worked / needed revision / quality gate failures / output accepted without enough review), Actions table, Improvements Triggered (**required** — cannot be left blank without a stated reason), New Intents Triggered, Post-Retro Improvement Workflow.

**The Post-Retro Improvement Workflow is mandatory and AI-driven.** Immediately after the retro document is complete, the AI must:
1. Synthesise every finding in "What Didn't Go Well" and "AI-Specific Observations" into concrete improvement proposals — one per finding — identifying the exact file and text to change
2. Present all proposals to the engineer for approval, rejection, or revision before touching any file
3. For each approved proposal: create an improvement file, apply the change to the target file, and update mirror files if the master rule file was modified
4. Mark each improvement Applied in the retro and close the retro only when all approved improvements are applied

The intent is that every retro automatically tightens the rules, skills, and guidelines that govern the next bolt. No finding should require the engineer to manually translate it into a file change.

### `ops/operate/improvements/_template.md`
Fields: Triggered by (retro/incident link), Target file, Current text, Proposed replacement, Reason, Validation criteria, Status, Applied date.

---

## Step 7 — Write Instructions2FDE.md

This is the main onboarding document for every engineer. Sections:
1. What AI-DLC is (the loop diagram: Inception → Build → Operate → Improvements)
2. How to invoke each ceremony by talking to the AI (not by following manual steps)
3. What the engineer still owns (review, AC confirmation, running tests, edge case checks)
4. Phase 1 — Inception (how intents and mob elaboration work)
5. Phase 2 — Build (bolts, unit execution order, review before merge)
6. Phase 3 — Operate (retros, incidents, improvements)
7. The Three Non-Negotiables (quality gate, review checklist, prompt log)
8. Using a different AI tool (Cursor → `.cursorrules`; GitHub Copilot → `.github/copilot-instructions.md`)
9. Common mistakes table
10. Quick reference table (ceremony → what to say to the AI)

---

## Step 8 — Confirm Tool Setup and Add Mirror Files

### Primary tool setup

By this point your master rule file should exist at the correct path for your chosen tool (see **Before You Begin**). Verify:

| Tool | Expected path | Loaded automatically? |
|---|---|---|
| Claude Code | `CLAUDE.md` at repo root | Yes — every session |
| Cursor | `.cursorrules` at repo root | Yes — every session |
| GitHub Copilot | `.github/copilot-instructions.md` | Yes — every session |

### Supporting multiple tools in the same repo

If your team uses more than one AI tool, create copies of the master rule file for each additional tool. The content is identical — only the file name, location, and internal link prefixes differ.

**Add Cursor support** (if your primary tool is Claude Code or Copilot):
```bash
cp CLAUDE.md .cursorrules
```
Open `.cursorrules` and update the opening line to reference Cursor.

**Add GitHub Copilot support** (if your primary tool is Claude Code or Cursor):
```bash
mkdir -p .github
cp CLAUDE.md .github/copilot-instructions.md
```
Open `.github/copilot-instructions.md`, update the opening line to reference GitHub Copilot, and change all `ai-dlc/` link prefixes to `../ai-dlc/`.

**Add Claude Code support** (if your primary tool is Cursor or Copilot):
```bash
cp .cursorrules CLAUDE.md   # or copy from .github/copilot-instructions.md
```
Open `CLAUDE.md`, update the opening line to reference Claude Code, and if copying from Copilot change all `../ai-dlc/` prefixes back to `ai-dlc/`.

### Sync discipline

Whenever the master rule file is updated, all mirror files must be updated in the same PR. Add this to your PR checklist — it is a team discipline, not an automated process.

---

## Step 9 — Initialize the Backlog and First Intent

1. Create `ops/build/backlog.md` with empty Open / Planned / In Progress / Done / Blocked / Deferred sections
2. Identify the first capability to build and write an intent: `ops/inception/intents/YYYY-MM-DD-<slug>.md`
3. Say to your AI assistant: "Run a mob elaboration for the [intent name] intent"
4. After sign-off, the AI creates unit files and updates the backlog
5. Say: "Plan a bolt from the open units in the backlog"
6. Say: "Execute unit [name] from bolt [name]"

---

## What Makes This Work

The quality of the framework depends entirely on two things:

**1. The specificity of the master rule file.** Generic rules produce generic output. The more your master rule file encodes your actual domain language, your actual architectural decisions, and your actual learned anti-patterns, the better every AI interaction will be. A master rule file written on day one will be much weaker than one shaped by three bolts of retros.

**2. The discipline of the retro loop.** The retro is the compiler for the process. Every failure that is not encoded into a rule will recur. Every retro that produces no improvement file means the next bolt starts from the same baseline. Run the retro after every bolt, file the improvements, and the framework compounds.

---

## Onboarding Completion — What the Agent Must Produce

When the agent has finished executing this guide, it must output a structured completion report before handing back to the engineer. The report must contain:

### 1. Files Created
A table of every file written during onboarding, grouped by folder.

| File | Status | Notes |
|---|---|---|
| `CLAUDE.md` (or tool equivalent) | Created | Sections 1–8 populated |
| `ai-dlc/rules/...` | Created | … |
| *(etc.)* | | |

### 2. Sections Requiring Engineer Review
List every section or field in the master rule file that the agent could not populate from the codebase and left as a placeholder. The engineer must fill these before the first Bolt runs.

### 3. Open Questions
Any ambiguity the agent encountered that the engineer must resolve — e.g., conflicting patterns found in the codebase, modules where ownership was unclear, or test coverage below the gate threshold.

### 4. First Recommended Action
One sentence: what the engineer should do next (e.g., "Review the domain glossary placeholders in Section 4 of the master rule file, then run a mob elaboration for the first intent in the backlog.").

> The agent must not mark onboarding as complete until all checklist items below are checked and this report has been presented.

---

## Checklist: Ready to Start

**Tool setup**
- [ ] AI tool identified (Claude Code / Cursor / GitHub Copilot)
- [ ] Master rule file created at the correct path for your tool (see Before You Begin)
- [ ] Mirror files created for any additional tools in use (see Step 8)

**Framework files**
- [ ] Folder structure created (`ai-dlc/` tree from Step 1)
- [ ] Master rule file written with all 8 sections (Step 2)
- [ ] `rules/` files written (prompt-quality-gate, code-standards, security, architecture)
- [ ] `skills/` files written (mob-elab-prompts, review-checklist)
- [ ] `guidelines/` files written (domain-glossary, edge-cases, acceptance-patterns, dev-setup)
- [ ] Ops templates written (intent, unit, bolt, retro, incident, improvement)
- [ ] `Instructions2FDE.md` written
- [ ] `ai-dlc/README.md` written

**First iteration**
- [ ] First intent written and ready for mob elaboration
