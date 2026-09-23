# Skill: Release Readiness Checklist

**Purpose:** Walk the engineer through the UAT Release Checklist or the Production Release Checklist item by item, one at a time, to get deliberate individual attention on every parameter before a release goes out — not a single glance-and-approve. Record the engineer's stated status for every item and produce a report at the end.

**This is a self-attestation record, not an evidence audit.** Unlike `product-engineering-essentials.md`, this skill never inspects the repository, CI logs, config, or any other artifact to verify a claim. The engineer states the status of each item; the AI's job is to ask, listen, record accurately, and move to the next item — never to second-guess, argue with, or independently verify what the engineer reports. It also never blocks a release: it surfaces the self-reported state so the people who actually approve the release (technical owner, QA, security, product, operations — as named in the checklist's own approval section) can decide with full information.

**Not the same as `uat.md`.** `uat.md` runs a stakeholder demo session to validate feature *behavior* against acceptance criteria. This skill checks release *mechanics* — build integrity, test completion, security scans, database changes, deployment and rollback readiness, environment availability, and sign-offs — for a specific UAT or production release. The two are complementary and may both run before the same release; neither depends on the other.

**Trigger:** engineer invokes directly — "run the UAT release checklist", "run the production release checklist", "release readiness check", "pre-release checklist", "check release readiness for [version]". Never run automatically or on a schedule — it starts only when the engineer asks for it.

---

## Step 1 — Choose the Checklist and Identify the Release

Ask:

> "Which checklist should I run — the **UAT Release Checklist** or the **Production Release Checklist**? And what release is this for — a version/build number, and a target date if you have one?"

Record:
- Checklist type (UAT or Production)
- Release version/build identifier
- Target release date
- Engineer running the checklist (and reviewer/approver name if different)
- If AI-DLC is installed and this release maps to a specific intent or bolt, the intent/bolt name (optional — ask once, do not chase it if the engineer doesn't have one handy). When supplied, carry it into the Step 3 report header (`Intent/Bolt:` line) — it's captured here so it stays traceable, not just for record-keeping in this step.

---

## Step 2 — Walk Through Every Item, One at a Time

Explain the format before starting:

> "I'll go through the checklist section by section, one item at a time. For each item, tell me: **Confirmed**, **Not confirmed**, or **N/A**. If it's Not confirmed, I'll ask what's blocking it. If N/A, give me a one-line reason so the record is useful to anyone reading it later."

Then work through the checklist in **Reference Checklists** below, in order, section by section.

**Rules for this step:**

- Present one item at a time and wait for an explicit response. Never infer or assume a status for an item the engineer hasn't addressed yet.
- If the engineer answers several items in one message (e.g., "1 through 4 confirmed, 5 not — no rollback script yet"), accept it, record each addressed item, and continue with whatever in that section is still unaddressed. Do not require strict one-message-per-item if the engineer is moving faster on their own.
- **Not confirmed:** ask "What's blocking this, or what needs to happen before it can be confirmed?" Record the answer as the blocker note for that item.
- **N/A:** ask for a one-line reason it doesn't apply to this release. Record it.
- Do not skip a section without the engineer explicitly opting out of it (e.g., "skip Database & Configuration, no data changes this release") — record that as a section-level N/A with the stated reason, applied to every item in the section.
- Do not editorialize on an answer, suggest the engineer is wrong, or supply your own assessment of whether an item is "really" done. If something the engineer says sounds concerning (e.g., "confirmed" for rollback validated but they mention in passing it's never been tested), you may ask a clarifying follow-up question, but the recorded status is always what the engineer states after that follow-up — never your own inference.

---

## Reference Checklists

**Maintenance note:** several items appear word-for-word in both checklists below (e.g. "Unit tests passed", "Rollback procedure documented"). If you edit an item's wording or threshold, check whether the same item exists in the other checklist and update it there too — the two lists are intentionally separate (production has more sections) but shared items should stay in sync.

### UAT Release Checklist

**1. Build & Code**
- Correct release version/build identified
- All code changes merged and code review completed
- CI/CD build completed successfully
- Release branch/tag created
- Correct deployment artifact confirmed

**2. Testing**
- Unit tests passed
- Integration/API tests passed
- Regression tests passed
- End-to-end tests passed where applicable
- No open critical/blocking defects
- Known issues documented

**3. Security**
- SAST/security scans completed
- Dependency/vulnerability scan completed
- No unresolved critical/high vulnerabilities
- Security exceptions formally approved where applicable

**4. Database & Configuration**
- Database changes reviewed
- Database migration scripts tested
- Configuration changes reviewed
- Required environment variables/secrets configured
- External integrations available and validated

**5. Deployment**
- UAT deployment procedure validated
- Deployment dependencies confirmed
- Rollback procedure documented
- Rollback procedure validated where applicable

**6. UAT Readiness**
- UAT test data is available
- UAT environment is available and stable
- UAT test cases/scenarios are prepared
- UAT users/business testers are informed
- Release notes provided
- Known limitations communicated

**7. Release Approval**
- Technical owner approval obtained
- QA approval obtained
- Release/change ticket updated
- UAT deployment approved

### Production Release Checklist

**1. UAT & Release Scope**
- UAT completed successfully
- Business/UAT sign-off obtained
- Exact production scope confirmed
- Release version/build is frozen
- Production artifact/version verified
- Release notes finalized

**2. Code & Testing**
- All production code changes approved
- CI/CD build completed successfully
- Unit tests passed
- Integration/API tests passed
- Regression tests passed
- End-to-end tests passed where applicable
- No unresolved critical/blocking defects
- Known issues documented and accepted where applicable

**3. Security & Compliance**
- Security scans completed
- Dependency/vulnerability scans completed
- No unresolved critical/high vulnerabilities
- Security exceptions formally approved where applicable
- Required compliance/privacy checks completed

**4. Database & Data**
- Production database migration scripts reviewed
- Database migration tested in a production-like environment
- Migration execution order confirmed
- Data migration/transformation validated
- Database backup completed/verified where required
- Database rollback/recovery approach confirmed

**5. Infrastructure & Configuration**
- Production infrastructure is ready
- Capacity/performance requirements verified
- Production configuration reviewed
- Secrets/credentials validated
- External dependencies/integrations validated
- Feature flags/configuration have the correct production state

**6. Deployment & Rollback**
- Production deployment procedure documented
- Deployment steps reviewed
- Deployment dependencies identified
- Rollback procedure documented
- Rollback/recovery approach validated
- Deployment window approved
- Required change-management approval obtained

**7. Monitoring & Operations**
- Application monitoring configured
- Infrastructure monitoring configured
- Logging verified
- Alerts configured and tested
- Key business metrics identified
- Post-deployment health checks defined
- Support/operations team notified
- On-call/escalation contacts confirmed
- Incident management process available

**8. Communication**
- Stakeholders informed of release
- Expected downtime/impact communicated
- Release notes published
- Support team provided with known issues/workarounds
- Customer/user communication completed where required

**9. Final Production Approval**
- QA approval obtained
- Technical/Engineering approval obtained
- Product/Business approval obtained
- Security approval obtained where required
- Operations/Infrastructure approval obtained where required
- Final release authorization obtained

**10. Post-Release Validation**
- Deployment completed successfully
- Application health checks passed
- Critical user journeys/smoke tests passed
- Database/application logs checked
- Monitoring and alerts checked
- No critical production errors observed
- Business validation completed where required
- Release/change ticket closed or updated

---

## Step 3 — Produce the Report

Present the full report in the conversation:

```
─────────────────────────────────────────────────────
[UAT | Production] Release Checklist
Release:       [version/build]
Target date:   [target release date from Step 1]
Checked on:    YYYY-MM-DD (date this checklist was run)
Engineer:      [name]
Approver:      [name, if different]
Intent/Bolt:   [name, if supplied in Step 1 — omit line if not]
─────────────────────────────────────────────────────

1. [SECTION NAME]                          [X confirmed / Y not confirmed / Z n/a]
  ✅ [item]
  ❌ [item] — Blocker: [reason given]
  — [item] — N/A: [reason given]
  ...

[... one block per section, in checklist order ...]

─────────────────────────────────────────────────────
SUMMARY

| Section | Confirmed | Not confirmed | N/A |
|---|---|---|---|
| 1. [Section name] | X | X | X |
| ... | | | |

Total: [X] / [Y] items confirmed

─────────────────────────────────────────────────────
ITEMS REQUIRING ATTENTION

[Every Not-confirmed item, grouped by section, with its blocker note.
Or: "None — every item was confirmed or explicitly marked N/A."]

─────────────────────────────────────────────────────
This is a self-reported record, not a verified audit — it reflects what the
engineer stated, not independent confirmation. It does not authorize or block
this release; that decision belongs to whoever holds sign-off per the
checklist's own approval section. Use it as the explicit, attributable record
of what was checked before this release went out.
─────────────────────────────────────────────────────
```

---

## Step 4 — Offer to Save and Follow Up

Ask:

> "Should I save this report, and would you like any 'Not confirmed' items turned into tracked units so they don't get lost before the release?"

- If yes to saving: write the report to `{FRAMEWORK_ROOT}/ops/operate/release-checklist-[uat|prod]-YYYY-MM-DD-[version-slug].md` if AI-DLC is installed, or to a path the engineer names otherwise. `[version-slug]` is the release version/build identifier from Step 1, slugified (lowercase, non-alphanumerics replaced with `-`) — this keeps same-day reports for different releases (e.g. a morning UAT run and an afternoon hotfix) from overwriting each other.
- If yes to backlog items: for each Not-confirmed item the engineer selects, create a unit with the item and its blocker note as context — do not auto-create units for items the engineer didn't select.
  - If AI-DLC is installed: create it under `{FRAMEWORK_ROOT}/ops/build/units/` following the existing unit template.
  - If AI-DLC is not installed: there is no unit template or backlog to target. Instead, append each selected item to the saved report (or to a path the engineer names) as a plain "Follow-up items" list with the item text and blocker note — do not invent a unit file structure.
- If the engineer declines both, that's a complete and valid outcome — confirm the checklist run is done and move on.

---

## Step 5 — Handoff

Close with:

> "That's the [UAT | Production] release checklist. It's a recorded self-attestation, not a gate — the release decision still belongs to whoever signs off per the checklist's approval section. Run this again for the next release rather than reusing this report; a checklist confirmed for one build doesn't carry over to another."
