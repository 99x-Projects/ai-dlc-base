# Skill: Product Engineering Essentials Check

**Purpose:** Assess how much of the foundational product-engineering groundwork a repository actually has in place, across ten pillars spanning product vision, domain understanding, requirements, UX, architecture, engineering practices, DevOps, quality engineering, security/compliance, and the delivery feedback loop. Produces a checklist report — what exists, what's partial, and what's missing — so the team has a shared, evidence-based picture instead of an assumption.

**These are not mandatory prerequisites.** A team does not need all ten pillars filled in before writing code, and this skill never blocks work. It is a best-practice visibility check: run it, see the gaps, and decide deliberately which ones matter for this project at this stage. Framing gaps as failures is a misuse of this skill — framing them as informed choices is the intent.

**Trigger:** Optionally offered once at the end of onboarding (see the onboarding completion report), and engineer-invoked at any later time by saying "run product engineering essentials", "check our product engineering foundations", or "what's missing from our essentials checklist."

**Never run automatically, and never run without the engineer's go-ahead** — not at session start, not on a schedule. If offered at the end of onboarding and declined, do not offer it again in the same session.

---

## Step 1 — Define Scope

Ask the engineer:

> "I can run the full ten-pillar check, or focus on specific pillars — for example just Architecture & Technical Foundation, or DevOps & Environment Foundation. Which would you like? (Say 'all' for the full check.)"

Record the answer. If the engineer says "all" or does not specify, check all ten pillars.

Also confirm scope of the codebase itself if the repository is a monorepo or contains multiple deployable services:

> "Should this cover the whole repository, or a specific service/package?"

---

## Step 2 — Gather Evidence

Read the repository before evaluating anything — do not score a pillar from memory or assumption. For each item, evidence is either a file that exists and says what it should, a file that exists but is a stub/placeholder, or the absence of anything on point.

Read, if present:

- The master rule file (`CLAUDE.md`, `.cursorrules`, or `.github/copilot-instructions.md`) — every section, not just the title
- `{FRAMEWORK_ROOT}/rules/*.md` and `{FRAMEWORK_ROOT}/guidelines/*.md` if AI-DLC is installed
- `{FRAMEWORK_ROOT}/ops/inception/intents/*.md` (a sample is enough for pattern-level items; read all of them for coverage items like MVP/roadmap)
- Top-level repo docs: `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `ARCHITECTURE.md`, anything under `docs/`
- CI/CD config: `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml`
- Infra/config: Dockerfiles, `terraform/`, `infra/`, `.env.example`, Kubernetes manifests
- Test directories and their structure (unit/integration/e2e/perf/security separation, or absence of separation)
- Package/dependency manifests (for the dependency-management and tech-choices items)

If the project has not been onboarded to AI-DLC, skip anything under `{FRAMEWORK_ROOT}` and rely entirely on generic repo signals — this skill must work on any repository, not only AI-DLC ones.

---

## Step 3 — Evaluate Each Pillar

For every item within the pillars in scope, assign one status:

| Status | Meaning |
|---|---|
| ✅ In place | A real, non-stub artifact exists and substantively addresses the item |
| ⚠️ Partial | Something exists but is thin, stale, a placeholder, or only covers part of the item |
| ❌ Missing | No evidence found anywhere in scope |
| — N/A | The engineer has told you this item is deliberately out of scope for this project (record why) |

Never infer ✅ from a file's mere existence — a `README.md` that only has a project name is not "In place" for Product Vision. Read enough of each artifact to judge substance, not just presence.

### Pillar 1 — Product Vision & Problem Definition

| Item | Look for |
|---|---|
| What problem are we solving? | Problem statement in README, product-vision doc, or master rule file Section 1 |
| Who has the problem? | Named user/customer segment, not just "users" |
| Why does it matter? | Stated business or user impact/urgency |
| What outcome are we trying to create? | A success definition distinct from a feature list |
| What is not part of the product? | An explicit out-of-scope or non-goals statement |

### Pillar 2 — User & Domain Understanding

| Item | Look for |
|---|---|
| Personas / user types | Named personas with distinguishing needs, not one generic "user" |
| User journeys | Documented end-to-end flows for at least the primary use case |
| Business processes | The real-world process the software supports or replaces |
| Domain terminology | `guidelines/domain-glossary.md` or equivalent, populated (not a stub) |
| Business rules | Documented rules/constraints distinct from code comments |
| Pain points | Documented problems with the current/prior state |
| Existing systems/processes | What this replaces, integrates with, or must coexist with |

### Pillar 3 — Product Requirements & Scope

| Item | Look for |
|---|---|
| Product capabilities | A capability-level description, not just a unit/ticket backlog |
| Functional requirements | Requirements traceable to intents or a requirements doc |
| Non-functional requirements | Performance, availability, compliance, etc. stated somewhere binding |
| MVP definition | An explicit "smallest thing that ships" statement |
| Priorities | Backlog or roadmap items ranked, not just listed |
| Acceptance criteria | ACs present on intents/units and testable (Given/When/Then or equivalent) |
| Future roadmap | Anything beyond the current bolt/sprint horizon |

### Pillar 4 — UX / Product Design Foundation

| Item | Look for |
|---|---|
| Information architecture | Navigation/structure documented or evident in routing |
| User flows | Flow diagrams or written step sequences for key tasks |
| Wireframes | Any low/high-fidelity design artifacts, linked or embedded |
| Interaction design | States, transitions, error/empty/loading states documented |
| UI design | Visual design artifacts or a live design file link |
| Design system | Shared component library, tokens, or style guide |
| Accessibility principles | An accessibility standard referenced (e.g. WCAG level) and applied |
| Responsive behavior | Breakpoint/behavior rules for different viewport sizes |

### Pillar 5 — Architecture & Technical Foundation

| Item | Look for |
|---|---|
| System architecture | A diagram or written description of the system shape |
| Major components/services | Enumerated components/services and their responsibilities |
| API strategy | REST/GraphQL/RPC conventions, versioning approach |
| Data architecture | Data model, storage choices, ownership boundaries |
| Integration architecture | How external systems are integrated (sync/async, contracts) |
| Technology choices | Stack documented with rationale, not just inferred from `package.json` |
| Security architecture | `rules/security.md` or equivalent — architectural security posture, not just checklist rules |
| Scalability considerations | Stated load expectations and how the design accommodates them |
| Infrastructure architecture | Hosting model, environments, network/topology description |

### Pillar 6 — Development Standards & Engineering Practices

| Item | Look for |
|---|---|
| Coding standards | `rules/code-standards.md` or equivalent, language/framework-specific |
| Git branching strategy | Documented branching model (trunk-based, GitFlow, etc.) |
| Pull-request process | PR template, required reviewers, or documented process |
| Code-review standards | A review checklist or documented review expectations |
| Definition of Done | Explicit DoD distinct from "tests pass" |
| Testing strategy | What layers are tested and to what depth, documented |
| Documentation standards | Where docs live and what must be documented, stated somewhere |
| Dependency management | An upgrade/audit cadence or policy (see `dependency-audit.md`) |
| Error-handling standards | Conventions for how errors are surfaced/handled/logged |
| Logging standards | What gets logged, at what level, in what format |
| Observability standards | Tracing/metrics conventions beyond ad hoc logging |

### Pillar 7 — DevOps & Environment Foundation

| Item | Look for |
|---|---|
| Source control | Present by definition if this is a git repo — confirm branch protection if visible |
| CI/CD | A working pipeline config, not just a placeholder file |
| Development environment | Setup instructions that a new engineer could follow (`guidelines/dev-setup.md` or `README`) |
| Test environment | A distinct test/QA environment referenced in config or docs |
| Staging/UAT | A staging environment and how it's used |
| Production | Production environment and access model documented |
| Infrastructure as Code | Terraform/CloudFormation/Pulumi/Bicep or equivalent, not manual console setup |
| Secrets management | A secrets manager or vetted pattern — never secrets committed to the repo |
| Configuration management | Environment-specific config handled consistently (env vars, config service) |
| Deployment strategy | Blue/green, canary, rolling, or documented plain deploys |
| Rollback strategy | A documented or scripted way to revert a bad deploy |

### Pillar 8 — Quality Engineering Foundation

| Item | Look for |
|---|---|
| Test automation strategy | Coverage across unit → component → integration → e2e → performance → security, as applicable to this product |
| Quality gates | CI enforces tests/lint/coverage before merge, not just runs them |
| Defect management | Where bugs are tracked and triaged |
| Regression strategy | How regressions are caught before release (test suite, manual pass, etc.) |
| Performance baseline | A recorded baseline or budget, not just "seems fast" |
| Security testing | SAST/DAST/dependency scanning, or a documented manual process |

### Pillar 9 — Security, Compliance & Operational Readiness

*Weight this pillar higher for SaaS, enterprise, financial, healthcare, or customer-data products — say so in the report if the project's domain (from Pillar 1/2 evidence) indicates one of these.*

| Item | Look for |
|---|---|
| Authentication | Implemented and documented auth mechanism |
| Authorization | Role/permission model, not just "logged in = allowed" |
| Identity model | How users/accounts/tenants are modeled |
| Data protection | Handling of sensitive data at rest and in transit |
| Encryption | TLS in transit; encryption at rest where warranted |
| Audit logging | Who-did-what-when trail for sensitive actions |
| Privacy | A privacy policy or data-handling statement |
| Compliance requirements | Named regulatory/contractual requirements (GDPR, HIPAA, SOC 2, etc.) if applicable |
| Backup/recovery | Backup cadence and tested restore process |
| Disaster recovery | An actual DR plan, not just backups existing |
| Monitoring and alerting | Alerts wired to something a human sees (see `notifications.md` if AI-DLC is installed) |

### Pillar 10 — Product Delivery & Feedback Loop

| Item | Look for |
|---|---|
| Discover | A mechanism for surfacing what to build next (user feedback, research, backlog grooming) |
| Design | Design happens before build, not concurrently by accident |
| Build | A working build process (the AI-DLC Inception → Build loop, or an equivalent) |
| Release | A defined release process, not ad hoc pushes |
| Measure | Product usage/outcome metrics actually collected post-release |
| Loop closure | Evidence that Measure feeds back into Discover — e.g. retros, `process-health.md` reports, or roadmap revisions citing usage data |

---

## Step 4 — Produce the Checklist Report

Present the full report in the conversation. Group by pillar, show every item's status, and total per pillar. Do not average pillars into a single score — a single number would imply the pillars are equally weighted and equally urgent for every project, which they are not.

```
─────────────────────────────────────────────────────
Product Engineering Essentials Check
Project: [project name]
Scope:   [All ten pillars | specific pillars named]
Date:    YYYY-MM-DD
─────────────────────────────────────────────────────

PILLAR 1 — Product Vision & Problem Definition        [X in place / Y partial / Z missing / W n/a]
  ✅ What problem are we solving? — [evidence: file/section]
  ⚠️ Who has the problem? — [note: named generically, no segment]
  ❌ Why does it matter? — no evidence found
  ...

PILLAR 2 — User & Domain Understanding                [X/Y/Z/W]
  ...

[... one block per pillar in scope ...]

─────────────────────────────────────────────────────
SUMMARY

| Pillar | In place | Partial | Missing | N/A |
|---|---|---|---|---|
| 1. Product Vision & Problem Definition | X | X | X | X |
| 2. User & Domain Understanding | X | X | X | X |
| ... | | | | |

─────────────────────────────────────────────────────
NOTABLE GAPS

[List every ❌ in a pillar that also had a domain signal making it high-stakes —
e.g. missing Authorization on a product that handles customer data, missing
Rollback strategy with an active production environment. This is judgment, not
a mechanical rule — explain why each listed gap is notable.]

─────────────────────────────────────────────────────
SUGGESTED FOCUS (not a mandate)

[1–3 sentences. The gaps most worth closing first, given what this specific
project is and where it is in its lifecycle — a pre-launch MVP and a mature
SaaS product should get different suggestions even from similar checklists.]
─────────────────────────────────────────────────────
```

---

## Step 5 — Offer to Save and Follow Up

Ask:

> "Should I save this report, and would you like any of the gaps turned into backlog items so they're tracked rather than forgotten?"

- If yes to saving: write the report to `{FRAMEWORK_ROOT}/ops/operate/product-engineering-essentials-YYYY-MM-DD.md` if AI-DLC is installed, or to a path the engineer names otherwise.
- If yes to backlog items: for each gap the engineer selects, create a unit under `{FRAMEWORK_ROOT}/ops/build/units/` following the existing unit template, with the gap description as context — do not auto-create units for gaps the engineer didn't select. This mirrors how `dependency-audit.md` converts findings into remediation units, but nothing here is created without explicit selection, since these gaps are not defects.
- If the engineer declines both, that's a complete and valid outcome — confirm the check is done and move on.

---

## Step 6 — Handoff

Close with:

> "That's the essentials check. None of this blocks any work — it's a shared picture of what's in place and what isn't, so gaps are a deliberate choice rather than an accident. Run this again whenever the project has moved on enough to be worth re-checking."
