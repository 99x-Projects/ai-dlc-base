# Framework Values

← [Back to README](README.md)

---

## The founding objective

The 99x Intent Delivery Framework was built on one governing objective:

> **Maximize value flow and feedback. Minimize ambiguity, unnecessary effort, and unmanaged risk.**

Every mechanic in this repo — the intent format, the quality gates, the bolt types, the retro-to-improvement pipeline — is a specific, load-bearing answer to one of these five forces. None of them is decorative process for its own sake; each exists because its absence was observed to cost a team either speed, learning, clarity, time, or safety. This document names the five forces and traces each to the concrete mechanisms in the framework that enforce it, so a contributor can tell *why* a rule exists before deciding whether to change it.

These five forces are not independent — several are in direct tension (see [Where the forces trade off](#where-the-forces-trade-off)). The framework's design is the set of specific trade-off decisions described below, not a claim that all five can be maximized/minimized simultaneously.

---

## 1. Maximize value flow

**Principle:** Work should move from intent to production in small, bounded, sequenced batches — never stalled, never blocked on a large undifferentiated chunk of unclear scope.

- **Small batch size.** One unit proposed per turn during elaboration; one bounded goal per bolt, mapped to exactly one intent — never several intents bundled into one unit of work.
- **Ceremony proportionate to the work, not uniform.** Bolt Type (Feature / Bug / Hotfix / NFR) selects the workflow weight — a hotfix or bug fix runs an abbreviated path instead of full elaboration, so low-risk flow isn't throttled by process built for high-risk flow.
- **The process is a loop, not a project.** `Inception → Build → Operate → Improvements → (back to Inception)` — delivery doesn't stop at "shipped"; operate and improvements are first-class phases that feed the next inception, keeping the whole system in continuous motion rather than resetting for each feature.

## 2. Maximize feedback

**Principle:** Every gate, unit of work, and failure must produce a signal that actually reaches the next decision — a feedback loop that isn't closed is treated as a defect, not a missing nicety.

- **Retro → Improvement → Applied is one loop, not a suggestion box.** A retro that produces observations but no applied improvement is scored *Not Aligned* — the loop only counts once the target file has actually changed.
- **Observability is designed in, not bolted on after.** Every unit must define a success signal, a failure signal, and an alert threshold *before* execution, so production behavior is visible the moment it ships, not discovered later.
- **Knowledge compounds across projects, not just within one.** The Knowledge Promotion step classifies every applied improvement as Promoted, Project-specific, or Declined-with-reason — a fix that would help *any* project graduates back into this base repo instead of staying trapped in one team's copy.
- **Estimates recalibrate from what actually happened.** The estimation agent's delivery mode closes the loop by calibrating future estimates from recorded hours, not by re-guessing each time.
- **The framework can be audited without being installed.** The diagnostics agent runs pull-mode against whatever artifacts a team already has, so the feedback loop between "what the framework prescribes" and "what a team actually does" exists even before full adoption.

## 3. Minimize ambiguity

**Principle:** Every artifact must be resolvable by its reader — human or AI — without inference or guesswork. Ambiguity in an instruction an AI reads and follows literally becomes inconsistent behavior downstream, not a style problem.

- **Intent before code, with testable acceptance criteria.** Every feature starts with a written intent; every acceptance criterion is Given/When/Then, never compound, never phrased as "the system should…" — a criterion that can't be mechanically checked isn't a criterion.
- **Explicit scope boundaries.** Every intent and every unit carries an explicit in-scope and out-of-scope list — silence is not treated as "out of scope by implication."
- **Contracts are locked before they're built against.** A design session (Phase 0) fixes API shape and data model *before* elaboration proceeds, so acceptance criteria are written against a stable interface instead of one still being decided.
- **Self-contained artifacts.** A unit must be readable on its own, without requiring the reader to first read its parent intent.
- **Routing over prose.** A master rule file is a compact routing table to the relevant rule/skill file, not a wall of embedded text — an AI that has to parse paragraphs to find an instruction has effectively lost it.
- **Findings are evidence-based, never assumed.** Status is read from what actually exists in the repository — a file's mere presence is never enough to infer "in place."

## 4. Minimize unnecessary effort

**Principle:** Ceremony costs time; knowledge that already exists should never be re-derived. Effort is only justified by the risk or ambiguity it's removing.

- **The lightest workflow that fits.** Bug and hotfix bolts are deliberately abbreviated relative to the feature bolt — running the full ceremony on a one-line hotfix is itself a process failure, not diligence.
- **Check before you build.** Pre-generation checks (grep for existing patterns, forbidden-zone checks) exist specifically to catch duplicated work before it's written.
- **One physical source of truth.** Skill and rule files are authored once in this base repo and copied verbatim or routed to — never restated per project — so a fix is made in exactly one place.
- **A generality test gates what becomes shared.** Before anything is added to the shared rules/skills, it must pass: *"if this were applied to a completely different project — different stack, domain, team size — would it still be an improvement?"* If not, it belongs in that project's own local rules, not in everyone's.
- **Small, single-purpose changes to the framework itself.** One skill, one rule, one template, one agent per PR to this repo — with no CI to catch scope creep, a tight diff is the only thing that keeps a consistency review tractable.

## 5. Minimize unmanaged risk

**Principle:** Risk must be classified and bounded explicitly, before work proceeds — never left implicit, never discovered only after something breaks.

- **Risk is classified at the point of intent.** Every intent carries an AI Risk classification (Minimal / Limited / High) — defaulting every intent to the same value regardless of what it touches is itself flagged as a failure to classify.
- **Blast radius is assessed before execution, not after an incident.** Every bolt requires a blast-radius table, a rollback assessment, and an explicit feature-flag decision before work begins.
- **Guardrails are structural, not just advisory.** Test coverage gates on existing-code changes, forbidden zones the AI must never touch, and a Breaking Changes Register for any contract-altering unit all exist so risk is bounded by the system, not by an individual's judgment in the moment.
- **Every AI interaction is gated and logged.** Output is checked against a quality gate and the interaction is logged for audit — trust is earned per interaction, not assumed by default.
- **Sign-off is explicit, not implied by silence.** Elaboration sign-off and UAT sign-off are both recorded fields with real states (including *Failed* and *Deferred*) — an intent isn't "done" because nobody objected, it's done because someone recorded that it is.

---

## Where the forces trade off

These five forces don't all pull the same direction, and the framework's specific mechanics are its answer to the conflicts:

- **Flow vs. risk management.** More gates slow flow; the framework's answer is *proportionate* ceremony (Bolt Type) rather than either extreme — full rigor for what's genuinely risky, an abbreviated path for what isn't.
- **Feedback vs. effort.** Closing every loop (retro → improvement → promoted knowledge) costs time the team could spend building; the framework's answer is to make the loop cheap to run (structured templates, a generality test that filters most improvements out of the shared repo) rather than skip it.
- **Ambiguity reduction vs. effort.** Writing testable ACs and locking contracts up front is slower than starting to code; the framework's bet is that the time is repaid by not re-deriving intent mid-build or re-litigating scope after the fact.

When a change to this repo seems to serve one force at the direct expense of another, that tension is worth naming explicitly in the PR description rather than resolved silently — see [CONTRIBUTING.md](CONTRIBUTING.md) and [RULES.md](RULES.md).

---

## Guarding the quality of human interaction

The five forces above describe what the framework optimizes *for*. None of it survives contact with reality if the human half of "team + AI" disengages, rubber-stamps proposals, or hands the AI an underspecified request — an artifact template can't rescue a session where the person filling it in wasn't really participating, and AI output is bounded by the quality of what it was given to work with. So the framework has a second layer of measures aimed not at what an artifact must contain, but at how well a human is actually interacting with the process while it's produced:

- **Nothing gets generated on underspecified input.** The Prompt Quality Gate blocks code generation until four components are present — Context, Constraints, Acceptance Criteria, Output Format — so a vague request is caught and completed *before* the AI guesses at what was meant, not corrected after a wrong output.
- **Sensitive information isn't propagated just because it was pasted.** The same gate's Sensitive Data Check scans every request for credentials, personal data, and other sensitive content regardless of whether the four components are complete; when found, the AI stops, names the category without repeating the value back, and asks the engineer to confirm it's intentional or redact it — the same stop-and-ask discipline as an underspecified request, applied to what a request exposes rather than what it omits.
- **Disengagement is actively monitored, not assumed away.** The engagement-monitoring rule watches for rubber-stamping across consecutive turns — single-word approvals, decisions approved without challenge, vague answers to open questions, deferring domain calls with no reason — and on three or more signals, the AI must stop, name the pattern directly, and ask one substantive diagnostic question before resuming. If the pattern continues, the ceremony pauses entirely rather than producing artifacts nobody actually validated.
- **Repeated iteration failure triggers diagnosis, not more guessing.** A circuit breaker trips after three consecutive same-cause output rejections on one unit — the AI stops iterating blindly, asks a targeted diagnostic question, and either incorporates new context, flags the acceptance criterion itself as wrong, or marks the unit Blocked. The outcome is logged in the Prompt Log and feeds directly into the next retro's AI-Specific Observations.
- **Review happens in digestible units, never a firehose.** Mob elaboration proposes one unit at a time with explicit AC confirmation before the next (interactive mode) — or, in plan-first mode, compresses this into a full draft the engineer reviews and confirms as a distinct step. Either way, sign-off is an explicit, confirmed action, never an inferred one from silence or a skim.
- **Shape decisions are surfaced before they're buried in code.** Before elaboration begins on an intent that introduces a new capability or an expensive-to-reverse decision, Solution Shaping asks once whether to lock generic-vs-specific, simplest-viable, and extend-vs-build decisions first — catching over-engineering or accidental platform-building while it's still a conversation, not a refactor.
- **Contracts are settled by dialogue, not inferred by the AI.** The Phase 0 Design Session forces API shape, data model, and architectural pattern decisions through direct back-and-forth with the engineer before any unit or AC is proposed — the binding constraints come from the human's domain knowledge, not the AI's assumption of it.
- **New engineers are shown the bar, not just told where it is.** New Engineer Induction personalizes the walkthrough with a project-specific quality-gate example and a personal quick-reference card, so a new team member's first real interaction with the framework already meets the target quality bar instead of being a learn-by-failure ramp.
- **Three Non-Negotiables are named as a set, not scattered as tips.** The engineer-facing onboarding document names the quality gate, the review checklist, and the prompt log together as the three things that are never skipped — regardless of how confident either side feels about a given change in the moment.

The measures above are what the framework *does* to protect interaction quality — gates, monitors, and escalation paths built into the process itself. But no process-side mechanism can manufacture a quality it depends on the human to bring in the first place. Engagement monitoring can detect that someone is rubber-stamping; it cannot make their answers true, complete, or evidence-based once they do engage. That half of the equation isn't a mechanism the framework can enforce — it's a set of dispositions it depends on and is designed to reward.

---

## Human quality attributes for framework users

If the five forces (§1–5) are what the framework optimizes for, and the interaction guards (above) are what it enforces structurally, this is the third layer: the character the framework assumes in the person on the other side of the conversation. None of these are checked by a gate the way an acceptance criterion is — they are what makes every gate, sign-off, and retro produce a true picture of the work rather than a well-formatted fiction.

1. **Honesty & Evidence Mindset**
   - Answer truthfully and accurately; state explicitly when something is unknown, uncertain, or incomplete rather than guessing to satisfy the process.
   - Support important statements with evidence, and distinguish "I know" from "I believe," "I assume," and "I need to verify."

2. **Openness & Transparency**
   - Share relevant assumptions, constraints, concerns, and risks rather than hiding inconvenient facts or known problems.
   - Make the actual state of the work visible — blockers, dependencies, deviations, and failures included, not just progress.

3. **Engagement & Responsiveness**
   - Participate actively rather than treating the framework as a passive compliance exercise; give considered responses, not reflexive approvals.
   - Provide required information promptly, and don't leave important questions unanswered without explanation.

4. **Precision**
   - Give specific, factual answers rather than vague or ambiguous ones.
   - Distinguish facts from assumptions, opinions, and estimates.

5. **Patience & Respect**
   - Stay constructive when the framework or AI asks clarifying or follow-up questions — extra questions establish context, they aren't an obstacle.
   - Keep disagreement focused on the problem, evidence, and outcome, never the person.

6. **Accountability & Closure Discipline**
   - Own decisions, commitments, deliverables, and identified issues rather than using the framework to shift responsibility.
   - Work actively to resolve open questions and actions, and leave work in a clear state of completion rather than indefinitely open-ended.

7. **Constructive Challenge & Courage**
   - Question a requirement, question, or control when it appears unnecessary, incorrect, or inappropriate — with reasoning and evidence, not just resistance.
   - Raise uncomfortable issues, risks, defects, and disagreements early, rather than letting them surface later at higher cost.

8. **Curiosity & Learning Mindset**
   - Seek to understand why a question or control exists, and be willing to explore assumptions and uncover hidden risks.
   - Reconsider an earlier answer when new information emerges; treat feedback as a chance to improve the solution, not a critique to deflect.

9. **Adaptability & Pragmatism**
   - Adjust when new information, risks, or constraints emerge, instead of treating an earlier answer as permanently correct.
   - Apply depth and rigor proportionate to the risk and importance of the work — avoiding both over-engineering and superficial compliance.

10. **Focus & Outcome Orientation**
    - Keep interactions relevant to the delivery objective, without unnecessary discussion, repetition, or detail that doesn't contribute to a decision.
    - Aim to complete the interaction and achieve the intended delivery outcome, not merely to complete a process.

**Why this belongs next to the mechanisms.** Several of the framework's structural gates only function as intended if the human meets them with the matching disposition from this list — the gate creates the opening; the attribute is what has to walk through it:

| Framework mechanism | Depends on |
|---|---|
| Prompt Quality Gate (Context · Constraints · ACs · Output Format · Sensitive Data Check) | Precision, Honesty & Evidence Mindset, Openness & Transparency |
| Engagement monitoring & disengagement intervention | Engagement & Responsiveness, Patience & Respect |
| Failed-output circuit breaker | Curiosity & Learning Mindset, Honesty & Evidence Mindset |
| AI Risk classification, blast radius, rollback assessment | Openness & Transparency, Constructive Challenge & Courage |
| "Success Looks Like" / non-technical outcome framing | Focus & Outcome Orientation |
| Elaboration and UAT sign-off | Accountability & Closure Discipline |
| Retro "What Didn't Go Well" and incident root cause | Honesty & Evidence Mindset, Openness & Transparency, Constructive Challenge & Courage |
| Solution Shaping and challenge of scope | Constructive Challenge & Courage, Adaptability & Pragmatism |
| Bolt Type / proportionate ceremony selection | Adaptability & Pragmatism |

A gate can catch a missing field. It cannot catch a confidently wrong answer, a risk left unmentioned, or a "looks good" that was never really considered — that is what this list is for.

---

## Evaluation against software delivery framework quality attributes

The five forces above are the framework's *design intent*. The table below evaluates how well the actual mechanisms in this repo deliver twenty standard quality attributes of a software delivery framework — where the evidence is strong, and where the attribute is only partially delivered because it depends on optional configuration or on a human/agent following through rather than something the system enforces structurally.

Ratings: **Strong** (the mechanism exists, is specific, and is checked), **Moderate** (the mechanism exists but relies on opt-in configuration, manual diligence, or judgment calls the base framework doesn't fully constrain), **Limited** (the attribute is only addressed indirectly or left to the consumer project).

| # | Attribute | Rating | How the framework delivers it | Where it falls short |
|---|---|---|---|---|
| 1 | Clarity | Strong | Intent → elaboration → unit → bolt → retro is a fixed artifact chain with named fields (Context, ACs, Scope, DoD); the master rule file routes every decision point to a specific rule or skill file. | Roles referenced in gates (senior engineer, technical owner, QA) are generic labels — a consumer project must still map them onto its real org chart; the base repo doesn't do this for them. |
| 2 | Precision | Strong | Acceptance criteria must be Given/When/Then, never compound, with at least one unhappy path; Definition of Done is a checklist, not a paragraph. | AI Risk classification (Minimal/Limited/High) and Bolt Type selection have no scoring rubric — the boundary between tiers is a judgment call, so two teams can classify the same kind of change differently. |
| 3 | Consistency | Moderate | Skills and rules are authored once and copied verbatim into every project; Sync Sets (`RULES.md`) name exactly which files must move together when something changes. | Nothing in this repo enforces the Sync Sets automatically — there is no CI (RULES.md §4); consistency depends on `pr-review.md` being run and taken seriously on every PR. |
| 4 | Agility | Strong | Bolt Type gives Bug/Hotfix/NFR work an abbreviated path instead of full elaboration; one unit proposed per turn keeps re-scoping cheap mid-session. | The Phase 0 design session and full archaeology onboarding are front-loaded investments — genuinely exploratory, pre-product-market-fit work can find that ceremony heavy before the abbreviated paths have anything to abbreviate. |
| 5 | Adaptability | Strong | Same content ships identically for Claude Code, Cursor, and Copilot; fresh-project interview and mature-project archaeology are separate onboarding paths; skills can be adopted standalone without the full framework. | The framework's core loop assumes an AI coding assistant is the primary interface to delivery work — teams whose delivery process isn't centered on one don't have a modeled adoption path. |
| 6 | Simplicity | Moderate | The generality test (`if this were applied to a completely different project, would it still be an improvement?`) keeps the shared rule set from bloating; Product Engineering Essentials and Release Readiness are explicitly "not a gate." | The full artifact set per feature (intent, elaboration, dependency-map entry, unit, bolt, retro, improvement, knowledge-promotion record) is still meaningfully more paperwork than ad hoc delivery, even where each piece is individually justified. |
| 7 | Traceability | Strong | Intents link to extracted units, units link to bolts, bolts link to retros, retros link to improvements, and each unit carries a Prompt Log — an unbroken document chain from "why" to "what changed." | The chain is document-level, not code-level: nothing in the base framework requires a machine-checkable link (e.g. a commit trailer or PR ID) tying a unit file to the actual commit that closed it. |
| 8 | Transparency | Moderate | Progress Digest, Notifications, and AI Hub Metrics exist specifically to surface status, risk, and delivery data to people who aren't at the keyboard. | Both Notifications and AI Hub Metrics are opt-in (`◈ Needs config`) — a project can run fully "aligned" to every other attribute with zero real-time visibility to anyone but the engineer driving the session. |
| 9 | Measurability | Moderate | Process Health computes four named metrics (improvement adoption rate, quality gate failure rate, AC revision rate, bolt velocity); dependency-audit classifies findings by severity. | These are on-demand skills, not always-on instrumentation — a metric only exists for a period in which someone remembered to invoke the skill. |
| 10 | Predictability | Moderate | The estimation agent has two modes (ballpark and delivery-level) and a calibration loop that recalibrates future estimates from recorded hours instead of re-guessing. | Nothing in the default flow requires estimation to run before a bolt starts, and calibration only works if recorded-hours data is actually logged — predictability is available, not structural. |
| 11 | Quality Assurance | Strong | Prompt quality gate and review checklist run on every AI interaction; pre-generation checks catch duplication before code is written; test coverage gates apply to existing-code bolts. | Domain 8 of the diagnostics agent shows the framework audits a project's CI/test/security automation but doesn't provide or require it — QA automation itself remains the consumer project's responsibility to build. |
| 12 | Risk Management | Strong | Every intent carries an AI Risk classification; every bolt requires a blast-radius table, rollback assessment, and feature-flag decision before execution; forbidden zones block changes to protected code without named approval. | The criteria for what makes an intent "High" versus "Limited" risk aren't rubric-defined in the base framework, so classification rigor still depends on the team applying it. |
| 13 | Automation-Readiness | Moderate | Skills are written as literal, repeatable protocols (scriptable send/push steps for notifications and AI Hub Metrics); Domain 8 explicitly scores which SDLC stages are Full/Co-pilot/Not Automated. | The framework's own evidence collection — DoD checklists, sign-offs, retro completion — is still manually filled markdown; nothing stops a checkbox being ticked without the underlying condition being true until a diagnostic review catches it after the fact. |
| 14 | Feedback-Driven | Strong | Retro → Improvement → Applied is scored *Not Aligned* if it doesn't close; UAT sign-off and incident RCA both feed back into rules and skills. | There's no modeled ingestion of direct customer/user feedback (support tickets, NPS, usage analytics) — UAT sign-off and incidents are the closest proxies for user-side signal. |
| 15 | Scalability | Moderate | Parallel archaeology handles large codebases via segment-based analysis and synthesis; the dependency-map tracks cross-intent dependencies; the diagnostics agent's FDE domain assesses skills across single- or multi-member teams. | Turn-by-turn elaboration and mob sessions are designed around one engineer (or one mob) at a time — the framework doesn't describe how multiple concurrent bolts avoid colliding beyond the static forbidden-zones/entry-point registries. |
| 16 | Governance | Moderate | Forbidden Zones name a required approval role per protected path; the Entry Point Registry gates which modules a bolt may even start in; sign-off fields are mandatory, typed states (not free text). | Decision rights are role-shaped ("senior engineer") rather than tied to real org identities, and there's no defined escalation path when a classification or sign-off is disputed. |
| 17 | Auditability | Moderate | Every unit carries a Prompt Log; the diagnostics agent's Artifact Log records exactly which file, from where, was used as evidence for every finding. | These logs are markdown files maintained by the same agent whose work they document — there's no tamper-evidence or independent system of record; auditability is only as strong as process discipline. |
| 18 | Value Orientation | Strong | Every intent's "What," "Why," and "Success Looks Like" must be written from a user/business perspective; a technical metric standing in for a user outcome is an explicit diagnostic finding. | Value is modeled per-feature — the framework doesn't roll individual intents up to a portfolio-level objective or business OKR. |
| 19 | Resilience | Moderate | Rollback assessment is mandatory per bolt; the circuit breaker halts execution after three consecutive same-cause AI output failures; incident files require an explicit AI-DLC contributing-factor assessment and an RCA. | These are reactive, triggered mechanisms (incident, hotfix, circuit breaker) — the framework governs the response process, not proactive fault-tolerance testing, which remains an architecture concern outside its scope. |
| 20 | Continuous Improvement | Strong | Retro findings must produce applied improvements or a stated reason not to; Knowledge Promotion tests every applied improvement for whether it generalizes back into this base repo; Process Health surfaces decay signals over time. | The loop from one project's learning back into the shared framework requires a deliberate promotion step (see `CONTRIBUTING.md`) — there's no automatic aggregation of decay signals across projects, so the base repo only improves as fast as contributors send improvements back. |

**The overall pattern:** the framework is strongest where an attribute is delivered by a *specific, named artifact field* checked at a defined moment (intent risk classification, blast radius, testable ACs, retro-to-improvement closure). It is more moderate wherever delivery depends on a skill being *invoked* rather than always running, a role being *filled* by a real person rather than named generically, or a log being *trusted* rather than independently verified. That pattern is consistent with what this repo actually is — a set of conventions and artifact contracts for AI-assisted teams, not a piece of enforcing software — and it means the attributes rated Moderate above are the ones most worth strengthening through a consumer project's own tooling (CI checks on Sync Sets, always-on metrics pushing, automated DoD verification) rather than through more prose in the framework itself.
