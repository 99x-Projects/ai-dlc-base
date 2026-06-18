# AI-DLC Review Guide

This document is the protocol for the AI-DLC Process Reviewer. It defines the eight review domains, the rubric for each artifact type, the scoring system, and the format for the final Review Report.

---

## Artifact Log

Throughout the session, maintain a running log of every artifact the engineer shares. Record the artifact type, the location the engineer provided it from, and a one-word status after review. Do not tell the engineer you are doing this — it is an internal tracking step.

```
| Artifact type       | Location provided by engineer | Status after review |
|---------------------|-------------------------------|---------------------|
| Master rule file    | [path or description]         | Reviewed            |
| Intent file         | [path or description]         | Reviewed            |
| ...                 | ...                           | ...                 |
```

At the end of the session, use this log to assess whether the team's artifacts are organized in a logical, findable, and consistent structure. This becomes the Organization Assessment in the Review Report.

---

## Review Scorecard

Maintain this table throughout the session. Update it after each domain completes.

| Domain | Score | Critical findings | Important findings | Advisory findings |
|---|---|---|---|---|
| 1. Foundation | — | — | — | — |
| 2. Inception | — | — | — | — |
| 3. Build | — | — | — | — |
| 4. Operate | — | — | — | — |
| 5. Process Adherence | — | — | — | — |
| 6. Organization & Structure | — | — | — | — |
| 7. People | — | — | — | — |
| 8. Tools | — | — | — | — |
| **Overall** | — | — | — | — |

**Scoring scale per domain:**

| Score | Meaning |
|---|---|
| **Aligned** | Artifacts and practice match AI-DLC principles with no critical gaps |
| **Partially Aligned** | Core structure is present but key elements are missing or weak |
| **Not Aligned** | Fundamental principles are not being followed; process is generating risk |

---

## Finding Severity

Every finding must be classified before it goes into the report.

| Severity | Meaning |
|---|---|
| **Critical** | The gap is actively undermining output quality or hiding risk — must be fixed before the next bolt |
| **Important** | The gap weakens the process over time and should be addressed in the next retro |
| **Advisory** | A refinement that would improve quality but is not urgent |

---

## Domain 1 — Foundation

**What you are reviewing:** The master rule file (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`)

**How to request it:**
> "Please share your master rule file — the file your AI tool loads automatically at the start of every session."

**What to look for and how to score it:**

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Project Identity (Section 1)** | Stack, boundaries, and system constraints are specific and accurate | Present but vague or incomplete | Missing or generic placeholder |
| **Prompt Quality Gate (Section 2)** | Routes to `rules/prompt-quality-gate.md` with a clear mandatory-read instruction | Gate referenced but embedded inline as a wall of text | Missing entirely |
| **Code Rules (Section 3)** | 3–5 hard-stop prohibitions in working memory; routes to rules files for the rest | Long lists embedded inline; no routing | Missing or only generic rules |
| **Domain Language (Section 4)** | Routes to `guidelines/domain-glossary.md`; 2–3 critical terms inline | Full glossary embedded inline (not routed) | Domain terms not defined |
| **Edge Cases (Section 5)** | Routes to `guidelines/edge-cases.md` with a mandatory-read instruction | Some edge cases inline; no routing | Missing |
| **Workflow (Section 6)** | Compact turn structure; routes to skills files; includes engagement monitoring reference | Turn structure present but long; no routing | Missing or not enforced |
| **Review Behavior (Section 7)** | Routes to `skills/review-checklist.md` with a mandatory-run instruction | Checklist embedded inline | Missing |
| **Reference Map (Section 8)** | Complete routing table covering all common needs | Partial | Missing |
| **Process Configuration (Section 9)** | Archive threshold set; `Next dependency audit` date populated | Section present but one or both values not set | Missing section |

**Common findings:**
- Master rule file is a wall of text rather than a routing table — AI loads it but cannot act on it efficiently
- Domain language section uses generic examples rather than the project's actual terms
- Hard-stop list has more than five items — items beyond five belong in `code-standards.md`
- Engagement monitoring reference is missing — disengagement will go undetected

---

## Domain 2 — Inception

**What you are reviewing:** Intent files and elaboration session files

**How to request it:**
> "Please share two or three intent files for recently delivered features — wherever you keep them in your project — and at least one elaboration session file for one of those intents."

Note the location each file comes from in the Artifact Log. Review the intent files first, then the elaboration files.

### Intent File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **What section** | User-perspective description; no implementation detail | Partially user-facing; some technical leak | Written from a technical perspective |
| **Why section** | Business or user reason clearly stated | Vague or missing motivation | Missing |
| **Success Looks Like** | Observable, non-technical outcome | Technical metric used | Missing |
| **Assumptions and Open Questions** | Specific and relevant | Generic or templated | Missing or empty |
| **Out of Scope** | Explicit list of excluded items | Vague | Missing |
| **AI Risk classification** | Field is set to Minimal, Limited, or High; not left blank | Present but appears defaulted without thought (all intents are Minimal) | Missing entirely |
| **UAT Sign-off** | Status recorded (Passed / Passed with findings / Failed / Deferred / Not required) before intent marked Implemented | Status is still Pending on an Implemented intent | Missing entirely on an Implemented intent |
| **Implementation Summary** | Completed for Implemented intents; not filled in for open intents | Partially filled | Missing on Implemented intents, or filled prematurely |
| **Extracted Units** | Units listed with links | Present but unlinked | Missing |

### Elaboration Session File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Design Session run (Phase 0)** | Design artifact created or design decisions explicitly captured before first unit was proposed; binding constraints stated | Some design discussion present but no formal artifact or binding constraints | No evidence of a design session — units proposed immediately with no API, data model, or architecture discussion |
| **Turn structure followed** | One unit proposed per turn; AC confirmation before edge cases | Multiple units proposed at once | No evidence of turn structure |
| **ACs are testable** | Every AC has Given/When/Then structure; no compound ACs | Some ACs are testable | ACs are vague or describe implementation |
| **Unhappy path covered** | At least one AC per unit covers a failure case | One unhappy path for the whole session | No unhappy paths |
| **Edge cases surfaced** | Referenced `edge-cases.md`; new ones identified | Edge cases discussed informally | Not addressed |
| **Sign-off recorded** | Final summary table and explicit sign-off before file creation | Partial | Files created without sign-off |
| **Dependency map updated** | `ops/inception/dependency-map.md` updated after sign-off; cross-intent dependencies and shared interfaces recorded | Map exists but not updated after this session | Map missing or never updated |

**Common findings:**
- "Success Looks Like" describes a technical metric (e.g. "API returns 200") rather than a user outcome
- ACs say "should" instead of "then" — "the system should X" is not a testable criterion
- Unhappy paths are missing — every unit needs at least one failure-case AC
- Elaboration session has no turn structure evidence — the AI proposed all units at once
- No evidence of a design session at the start of elaboration — units were proposed without first locking down API contracts or data model, so ACs were written against a vague interface
- AI Risk field is missing or all intents are marked Minimal regardless of whether they process personal data — risk classification was not applied
- UAT Sign-off is missing or still Pending on intents marked Implemented — the acceptance loop was not closed
- `ops/inception/dependency-map.md` was never updated after elaboration sign-off — cross-intent dependencies are invisible to bolt planning

---

## Domain 3 — Build

**What you are reviewing:** Unit files and bolt files

**How to request it:**
> "Please share the planning file for a recently completed batch of work and three or four of the individual work-item files that were part of it — wherever you keep them."

Note the location each file comes from in the Artifact Log. Review the planning file first to understand the scope, then review the individual work-item files.

### Bolt File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Bolt Type** | Field set to Feature, Bug, Hotfix, or NFR; correct skill was used for the declared type | Field present but type does not match the actual work (e.g. Bug bolt run as Feature) | Field missing entirely |
| **Goal is bounded** | One clear outcome; maps to one intent | Multiple intents mixed | Vague or missing goal |
| **Units table** | All units listed with status and links | Present but incomplete | Missing |
| **Execution order** | Dependencies between units are documented | Implied but not stated | Missing — units executed in arbitrary order |
| **Risk Assessment** | Blast radius table present; rollback assessment and feature flag decision recorded (mandatory for mature projects) | "Risks and Assumptions" present but no blast radius or rollback analysis | Missing entirely |
| **Retrospective link** | Links to completed retro for Done bolts | Missing on Done bolts | — |

### Unit File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Context paragraph** | Self-contained; readable without the intent | Requires reading the intent | Missing |
| **ACs** | Testable Given/When/Then; at least one unhappy path; no compound ACs | Partially testable | Vague or missing |
| **Scope** | Explicit in-scope and out-of-scope lists | Only one of the two | Missing |
| **Pre-generation Checks** | Grep patterns listed; forbidden zones checked | Partial | Missing |
| **Observability** | Success signal, failure signal, and alert threshold defined; "Not applicable" explicitly stated if none | One or two fields present but incomplete | Missing entirely |
| **Breaking Changes Register** | Present and approved for Migration/Remediation bolts; "N/A" for others | Present but not approved | Missing on contract-change units |
| **Definition of Done** | All items checked off; prompt log linked | Partially checked | Not completed |
| **Integration test AC** | Standard or contract-change form present | Missing | Missing |

**Common findings:**
- Units have no out-of-scope section — AI scope-creeps silently
- Pre-generation checks are empty — duplication goes undetected
- Observability section is missing — no success or failure signal defined, so production behavior is invisible
- Definition of Done not completed after execution — no evidence the unit was properly closed
- Breaking Changes Register missing on Remediation units that changed API contracts
- Bolt has no Risk Assessment — blast radius and rollback options were never analyzed before execution began
- Bolt Type field is missing or set to Feature for work that is clearly a bug fix or hotfix — the abbreviated workflow was not used, adding unnecessary ceremony overhead
- Bug bolt did not trigger RCA for a recurring bug — the recurrence check was skipped or its output was ignored

---

## Domain 4 — Operate

**What you are reviewing:** Retro files, improvement files, and incident files (if any)

**How to request it:**
> "Please share your last two retrospective write-ups and any improvement records they triggered — wherever you keep these in your project. If you have had any production incidents, share one of those too."

Note the location each file comes from in the Artifact Log.

### Retro File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **What Went Well is specific** | Named prompts, decisions, or patterns that worked | Generic observations | Missing |
| **What Didn't Go Well is specific** | Named failures with enough detail to trace to a root cause | Vague observations | Missing or "nothing went wrong" |
| **AI-Specific Observations** | Prompts analysed; quality gate failures logged; output-without-review cases named | Partially filled | Missing or empty |
| **Improvements Triggered** | At least one improvement per retro; reason stated if none | Present but improvements not applied | Left blank with no reason |
| **Post-Retro Workflow completed** | Improvements applied to target files; retro closed only after | Started but not finished | Not run |
| **New Intents Triggered** | Populated or explicitly "None — reason stated" | Missing | — |

### Improvement File Rubric

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Current text and replacement both present** | Exact text shown for both | One is vague | Missing either |
| **Reason traces to the retro** | Explicit link and explanation | Link present, no explanation | Missing |
| **Validation criteria set** | Observable signal defined | Generic | Missing |
| **Status is Applied** | Change is applied; date recorded | Open for more than one bolt | Rejected without reason |
| **Affected open units** | Field populated or "None" recorded; impact check was run before applying the change | Field missing on improvements that changed shared rules | Field absent from improvement template entirely |
| **Knowledge Promotion** | Classified as Promoted, Project-specific, or Declined (with reason); not left as Pending | Pending on an Applied improvement older than one bolt | Field missing entirely |

### Incident File Rubric (if applicable)

| Check | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **Timeline is complete** | Detection, investigation, and resolution all timestamped | Partial | Missing |
| **Root cause is specific** | Names what exactly was wrong, not "human error" | Semi-specific | Vague |
| **AI-DLC contributing factor assessed** | Explicitly checked yes or no with detail | Checked but no detail | Not filled in |
| **Improvements triggered** | At least one improvement filed | None with no reason | Missing section |
| **RCA run** | Root-cause-analysis skill has been run; linked | Not run | — |

**Common findings:**
- Retros produce observations but not improvements — the feedback loop is not closing
- Improvement files are created but never applied — the target files are unchanged
- Knowledge Promotion field is missing or still Pending on Applied improvements — generic improvements never reach the base repository
- "What Didn't Go Well" is left blank or says "nothing" — this is a signal the retro was not run seriously
- Incidents have no AI-DLC contributing factor assessment — the framework cannot learn from production failures

---

## Domain 5 — Process Adherence

**What you are reviewing:** Observable practice — not files, but how the team actually works. This domain is assessed through conversation, not artifact inspection.

**How to conduct it:**
Ask the following questions one at a time. Wait for each answer before continuing.

1. > "Walk me through what happens from the moment an engineer decides to build a new feature to the moment the first line of code is generated. What steps happen in between?"

   *Looking for:* intent written → mob elaboration run → ACs confirmed → quality gate checked before code generated.

2. > "When the AI proposes an acceptance criterion during elaboration, what does your team typically do with it? Has anyone ever pushed back on one?"

   *Looking for:* active challenge and modification of ACs, not passive acceptance.

3. > "After the AI generates code, what does the review step look like? Who does it and what are they checking?"

   *Looking for:* review-checklist.md used systematically, not a casual glance at the diff.

4. > "Has the AI ever flagged that the quality gate components were missing before generating code? Can you give me an example?"

   *Looking for:* evidence the gate is actually enforced, not just documented.

5. > "After a retro, how do the improvements get applied? Who does it and how do you know it happened?"

   *Looking for:* improvements applied to the actual files, not just filed and forgotten.

6. > "Have you ever noticed a pattern of recurring issues — the same type of problem appearing across multiple bolts?"

   *Looking for:* awareness of systemic gaps; use of root-cause-analysis skill.

7. > "Has the AI ever stopped mid-unit and said it couldn't proceed — either because output kept failing review or because it reached a decision it couldn't make without more context? What happened?"

   *Looking for:* evidence the circuit breaker in `engagement.md` is firing correctly; units blocked with a recorded reason rather than endlessly retried; failure patterns surfaced in the retro.

8. > "When a feature is finished and all units are done, what happens before you mark the intent as Implemented? Is there a testing or acceptance step with the actual user or stakeholder?"

   *Looking for:* UAT is conducted using the uat.md skill; sign-off status is recorded in the intent file before it is closed; intent is not marked Implemented solely because tests pass.

9. > "Does your project have a scheduled dependency audit? When did you last check your package manifests for outdated or vulnerable dependencies?"

   *Looking for:* `Next dependency audit` date set in Section 9 of the master rule file; dependency-audit.md skill has been run at least once; remediation bolts were created for high/critical findings.

**Scoring:**

| Score | Signal |
|---|---|
| **Aligned** | Engineer describes all steps fluently; evidence of gate enforcement and active AC challenge |
| **Partially Aligned** | Steps are known but shortcuts are taken; review is informal |
| **Not Aligned** | Process is not followed; AI generates code without elaboration or gate; retro loop is not closing |

**Common findings:**
- Engineers describe the process correctly but admit the quality gate is skipped when they are in a hurry
- AC review is passive — the engineer accepts whatever the AI proposes
- Improvements are filed but the target files are never actually updated
- The retro is run but the Post-Retro Improvement Workflow is not — the loop does not close
- Rule changes are applied without checking which open units reference the changed section — existing work is silently invalidated
- The circuit breaker in `engagement.md` has never fired — either output quality is consistently high, or failing units are being endlessly retried without escalation
- Intents are marked Implemented without UAT — tests passing is treated as sufficient, but the user outcome was never verified
- No dependency audit has been run and no date is set — the project's security posture is unknown

---

## Domain 7 — People: FDE Skills

**What you are reviewing:** The team's capability to operate the AI-DLC process effectively. This domain is assessed through conversation — not files, but how team members actually work.

**How to conduct it:**

Ask the following questions one at a time — one per FDE Focused Area. Wait for each answer before continuing.

1. > "How well do you understand the customer's business domain? Can you walk me through the key business processes, the main user groups, their pain points, and any compliance or regulatory constraints that shape what you build?"

   *Looking for:* ability to name specific processes, user groups with distinct needs, domain terminology, and at least one compliance or regulatory constraint. Not generic descriptions like "they're a finance company."

2. > "How close is your relationship with the customer — not just professionally, but personally? Do you understand how they prefer to communicate and make decisions? Do you get early visibility into real problems before they become formal requirements?"

   *Looking for:* established trust; knowledge of the customer's communication style and decision-making preferences; engineer hears about real problems directly, not filtered through a PM.

3. > "When something blocks your progress — a missing requirement, a technical risk, a dependency — what do you do? Can you give me a recent example where you saw a problem coming and got ahead of it?"

   *Looking for:* proactive risk identification before escalation is forced; clear ownership of outcomes; knows when and how to involve the customer without over-escalating.

4. > "Can you describe a recent decision where your area of expertise shaped how the team approached a problem? How did you connect that technical decision to a business goal the customer cares about?"

   *Looking for:* deliberate application of expertise grounded in business goals; ability to articulate the value created, not just the technical output; evidence of design thinking or problem framing before solution.

5. > "How do you keep stakeholders informed? What does your written communication look like — do you document decisions and risks proactively, or do you tend to rely on verbal updates and meetings?"

   *Looking for:* proactive, fact-based written communication; no surprises on status or risk; decisions documented; escalation happens before deadlines slip, not after.

### FDE Skills Rubric

| FDE Focused Area | Aligned | Partially Aligned | Not Aligned |
|---|---|---|---|
| **FA01 — Business Domain Knowledge** | Names key processes, user groups, pain points, domain terms, and at least one compliance constraint without prompting | Knows the surface domain but cannot explain user pain points or regulatory constraints | Cannot describe the customer's business beyond the product category or tech stack |
| **FA02 — Customer Closeness** | Trust is established; understands the customer's communication style and decision-making preferences; receives early visibility into real problems | Has regular contact but the relationship is formal and transactional; problems arrive as formal requests | Interaction is infrequent or mediated through a project manager; no direct access to the customer's real priorities |
| **FA03 — Self-Motivation and Ownership** | Proactively identifies risks and blockers before they escalate; takes full ownership of outcomes; involves the customer at the right moment | Takes ownership when reminded; escalates reactively after problems are visible | Waits for instructions; treats work as task execution; doesn't distinguish between "it's done" and "the outcome is achieved" |
| **FA04 — Value Through Expertise** | Connects technical decisions to business goals explicitly; can articulate what value was created; applies critical thinking and design thinking before proposing a solution | Applies expertise competently but the link to business outcomes is implicit or unstated | Executes requirements without questioning whether they solve the right problem; solutions are technically correct but business-disconnected |
| **FA05 — Effective Communication** | Communicates fact-based and proactively; documents decisions and risks in writing; no surprises; escalates early; maintains visibility across all stakeholders | Communicates when asked; documentation is inconsistent or after-the-fact | Communication is reactive; stakeholders are surprised by status changes, risks, or decisions already made |

**Scoring:**

| Score | Signal |
|---|---|
| **Aligned** | Engineer demonstrates all five focused areas fluently; customer outcomes drive decisions, not just technical execution |
| **Partially Aligned** | Two or three areas are strong; one or two are weak or dependent on team support to compensate |
| **Not Aligned** | Multiple focused areas are missing; engineer operates as a technical executor rather than a forward-deployed partner |

**Common findings:**
- Engineer knows the tech stack but cannot describe customer pain points or business processes — solutions risk being technically correct but user-wrong
- Customer interaction is mediated through a PM — the engineer lacks direct access to real priorities and gets sanitised requirements
- Work is executed as a task list rather than owned as an outcome — risks and blockers surface late or not at all
- Technical decisions are made without an explicit link to business goals — the connection between code and customer value is assumed, not stated
- Communication is verbal and undocumented — decisions made in meetings are not written down, creating misalignment over time

---

## Domain 8 — Tools: Automation Posture

**What you are reviewing:** How much of the team's SDLC is automated, where the gaps are, and how those gaps affect AI-DLC execution.

**How to conduct it:**

Ask the following questions one at a time. Record the classification for each stage before continuing.

1. > "How do you enforce code style and quality? Does it run automatically on every commit or PR, or is it done manually?"

   *Looking for:* linters, formatters, static analysis running as a pre-commit hook or CI check; not "we have ESLint installed but run it manually."

2. > "How does unit testing work? Does it run automatically, and does a failing test block a merge?"

   *Looking for:* tests run on every PR; failures block merge; coverage threshold enforced.

3. > "What does integration or end-to-end testing look like? Is it automated?"

   *Looking for:* automated suite running in CI; not "we test manually before release."

4. > "What does a code review look like? Are there automated checks that run before a human reviewer looks at it?"

   *Looking for:* CI, linting, and type-checking gates before human review; evidence that `review-checklist.md` is used.

5. > "How does your build and CI pipeline work? What triggers it and what does it verify?"

   *Looking for:* every PR triggers build; artifact produced and verified before merge.

6. > "How does deployment to staging and production work — automated, semi-automated, or manual?"

   *Looking for:* automated staging deploy on merge to main; production deploy via pipeline with approvals, not SSH sessions.

7. > "After you deploy, how do you know if something goes wrong? What monitoring and alerting do you have?"

   *Looking for:* APM, error tracking, uptime monitoring with alert routing; not "we check the logs if someone reports a bug."

8. > "Do you have automated security scanning — SAST, DAST, container scanning, or similar?"

   *Looking for:* at least one automated security scan in CI; findings reviewed and actioned, not ignored.

9. > "How are dependency updates and vulnerability alerts handled? Is it automated?"

   *Looking for:* Dependabot or equivalent raising PRs automatically; vulnerability alerts routed to a named owner.

### Automation Posture Rubric

For each SDLC stage, classify using one of three tiers:

| Tier | Definition |
|---|---|
| **Full Automation** | Process runs without human intervention; enforced as a gate (blocks merge or deploy on failure) |
| **Co-pilot Automation** | Tool or AI assists, but a human must approve, trigger, or act; not a hard gate |
| **Not Automated** | Fully manual or no process exists |

After classifying each stage, assess the AI-DLC impact for any stage that is Not Automated:

| SDLC Stage | AI-DLC Impact if Not Automated |
|---|---|
| **Code quality / linting** | AI-generated code style violations accumulate silently — pre-generation checks become the only quality gate |
| **Unit testing** | Definition of Done cannot be verified objectively — units are closed on trust, not evidence |
| **Integration testing** | Cross-unit regressions introduced by AI output go undetected until staging or production |
| **Code review** | Reviewers spend time on formatting rather than logic — AI output review quality degrades |
| **Build / CI** | No reproducible environment to verify AI-generated code — "works on my machine" is the only signal |
| **Deployment** | Manual deployment introduces human error into AI-generated changes; rollback capability is unclear |
| **Monitoring / alerting** | Production failures from AI output go undetected until user reports — circuit breaker failure data is invisible |
| **Security scanning** | AI-generated code with security vulnerabilities ships undetected; dependency audit skill has no safety net |
| **Dependency management** | Dependency audit skill operates reactively — findings surface late and remediation bolts are unplanned |

For every stage classified as **Not Automated**, record in the report:
- **Challenge:** What risk does this gap create given this project's context?
- **Suggestion:** What specific automation would reduce this risk and lower cognitive load on the AI-DLC process?

**Scoring:**

| Score | Signal |
|---|---|
| **Aligned** | Six or more stages at Full Automation; no critical stages (CI, unit testing, security scanning) are unautomated |
| **Partially Aligned** | Three to five stages at Full Automation; critical stages are at Co-pilot level or better |
| **Not Aligned** | Fewer than three stages at Full Automation; critical stages are manual or absent |

**Common findings:**
- Build and CI are automated but testing is not — the pipeline verifies compilation, not correctness
- Monitoring exists but alerts are not routed to anyone — failures are noticed by users before engineers
- Security scanning is absent — AI-generated code receives no automated security review
- Deployment is semi-automated to staging but fully manual to production — rollback is ad hoc
- Dependency management relies on human memory rather than automated alerts — the dependency-audit skill runs against an unknown baseline

---

## Review Report Format

Present this report at the end of the session.

---

### AI-DLC Process Review — [Project Name]

**Date:** YYYY-MM-DD
**Reviewer:** AI-DLC Process Reviewer
**Scope:** Full review / [Domain(s) reviewed]

---

#### Review Scorecard

[Insert completed scorecard table]

---

#### Findings

List all findings grouped by severity. For each finding:

```
[CRITICAL / IMPORTANT / ADVISORY] — [Domain]

Observed: [What was seen in the artifact or conversation]
Principle violated: [Which AI-DLC principle this breaks and why it matters]
Recommended change: [Specific, actionable change — name the file, section, or practice to modify]
```

---

#### Organization Assessment

Using the completed Artifact Log, evaluate whether the team's artifacts are organized in a logical, consistent, and findable structure. Score each dimension:

| Dimension | Score | Observation |
|---|---|---|
| **Colocation** | Aligned / Partial / Poor | Are like artifacts stored together? (all intents in one place, all retros in one place, etc.) |
| **Naming consistency** | Aligned / Partial / Poor | Do file names follow a consistent, date-prefixed or slug-based convention that makes chronological order and topic obvious? |
| **Discoverability** | Aligned / Partial / Poor | Could a new team member find any artifact type within 30 seconds without asking? |
| **Cross-linking** | Aligned / Partial / Poor | Are related artifacts linked to each other (intent → units → bolt → retro)? |
| **Orphaned artifacts** | None found / Some found / Many found | Are there files that exist but are not linked from anywhere? |

If the team's structure differs significantly from the AI-DLC recommended layout, note it here and explain whether the deviation is intentional and defensible or accidental and worth correcting.

---

#### People-Process-Tools Alignment

Summarise the findings across the three alignment dimensions. Present as three sub-sections.

**People**

Summarise the team's capability profile across the five FDE Focused Areas. Note which areas are strong, which are weak, and whether gaps are individual or team-wide. Flag any bus-factor risks (capability concentrated in one person).

| FDE Focused Area | Assessment | Finding |
|---|---|---|
| FA01 — Business Domain Knowledge | Aligned / Partially Aligned / Not Aligned | [Observation] |
| FA02 — Customer Closeness | Aligned / Partially Aligned / Not Aligned | [Observation] |
| FA03 — Self-Motivation and Ownership | Aligned / Partially Aligned / Not Aligned | [Observation] |
| FA04 — Value Through Expertise | Aligned / Partially Aligned / Not Aligned | [Observation] |
| FA05 — Effective Communication | Aligned / Partially Aligned / Not Aligned | [Observation] |

**Process**

[Drawn from Domain 5 findings. Summarise in 2–3 sentences: which ceremonies are consistently followed, where shortcuts are taken, and whether the feedback loop (retro → improvement → applied change) is closing.]

**Tools**

Present the automation posture table. For each stage classified as Not Automated, include the challenge and suggestion.

| SDLC Stage | Automation Tier | Challenge (if gap) | Suggestion (if gap) |
|---|---|---|---|
| Code quality / linting | Full / Co-pilot / Not Automated | — | — |
| Unit testing | Full / Co-pilot / Not Automated | — | — |
| Integration testing | Full / Co-pilot / Not Automated | — | — |
| Code review | Full / Co-pilot / Not Automated | — | — |
| Build / CI | Full / Co-pilot / Not Automated | — | — |
| Deployment | Full / Co-pilot / Not Automated | — | — |
| Monitoring / alerting | Full / Co-pilot / Not Automated | — | — |
| Security scanning | Full / Co-pilot / Not Automated | — | — |
| Dependency management | Full / Co-pilot / Not Automated | — | — |

Close with a 2–3 sentence summary: overall automation posture, which gaps have the highest AI-DLC impact, and the single automation investment that would most reduce process risk.

---

#### Gap Analysis

Map every finding to the specific AI-DLC principle it violates and state the distance between the team's current practice and the target state. Present as a table.

| # | Domain | AI-DLC Principle | Current State | Target State | Gap Severity |
|---|---|---|---|---|---|
| 1 | [Domain] | [The principle being violated — e.g. "Every AC must have Given/When/Then structure"] | [What the team is doing now] | [What aligned practice looks like] | Critical / Important / Advisory |

After the table, write a short paragraph (3–5 sentences) summarising the overall gap pattern: which domains are most misaligned, whether the gaps are concentrated in a single area or spread across the process, and whether the issues suggest a systemic problem (e.g. the process is documented but not followed) or isolated gaps in specific artifacts.

---

#### Remediation Plan

> **Produce this section only if there are Critical or Important findings. If all findings are Advisory, write: "No remediation plan required — all gaps are advisory refinements. Address them at the next retro."**

Group actions into three time horizons. For each action, name the exact change required, the specific file or practice to modify, and how the team will know the action is complete.

**Immediate — before the next bolt runs**

These are Critical findings that are actively putting output quality or process integrity at risk.

| # | Action | What to change | Verification |
|---|---|---|---|
| 1 | [What to do — specific and imperative] | [File name / section / practice] | [How you know it is done — observable signal] |

**Short-term — address at the next retro**

These are Important findings that weaken the process over time but do not block the next bolt.

| # | Action | What to change | Verification |
|---|---|---|---|
| 1 | [What to do] | [File name / section / practice] | [Observable signal] |

**Long-term — over the next two to three bolts**

Structural changes that require deliberate iteration — new intent files, architectural adjustments, or process habits that take time to establish.

| # | Action | What to change | Verification |
|---|---|---|---|
| 1 | [What to do] | [File name / section / practice] | [Observable signal] |

---

#### First Action

> [One sentence: the single highest-impact change the team should make before the next bolt runs — drawn from the Immediate tier of the Remediation Plan. Name the specific file or practice, not a general improvement.]

---

#### Patterns Worth Monitoring

[Any recurring themes across domains that did not reach Critical severity but suggest a direction the process is drifting. Flag these so the team can watch for them over the next few bolts. Optional — omit if none.]
