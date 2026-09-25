# PR Review Skill — `ai-dlc-base` Structural Integrity

← [Back to README](README.md) · Enforces [RULES.md](RULES.md)

This is a self-contained prompt protocol, in the same style as this repo's other skills: a reviewer reads this file to their AI assistant (Claude Code, Cursor, or Copilot) at review time. It is maintainer-facing only — it is not copied into consumer projects by the onboarding agent.

**How to invoke:**
> "Read `pr-review.md` and review PR #[N] against ai-dlc-base's structural rules."

or, with a GitHub PR link instead of a number:
> "Read `pr-review.md` and review https://github.com/[org]/[repo]/pull/[N] against ai-dlc-base's structural rules."

or, for an uncommitted local diff:
> "Read `pr-review.md` and review the current diff against ai-dlc-base's structural rules."

A PR number and a GitHub PR link are the same target for every rule below — "PR target" means either form. Only a local/uncommitted diff (no PR to comment on) is the different case.

This skill does not judge content quality on its own authority — a human still approves the PR. Its job is the thing humans reliably miss under review pressure: **did this change keep every cross-referenced file in sync — not just mentioning the same name, but still saying something true about it — and does the diff stay in scope.** Grep proves presence, not correctness; Step 5 exists because a passing grep and a broken repo are not mutually exclusive.

**Default output — post, don't fix:** When the target is a PR (given as a number or as a GitHub PR link), the default outcome of running this skill is a comment posted on that PR (Step 10) — not an edit to any file. This skill never modifies the repo, creates commits, or opens branches to address its own findings, Blocking or Advisory, unless the reviewer explicitly asks for the fix to be applied in this same invocation or a follow-up one (e.g. "now fix the blocking finding" or "apply that"). Reviewing and fixing are two separate asks; don't collapse them because a finding looks quick to fix.

---

## Step 1 — Get the diff

- For a PR number or a GitHub PR link: fetch it (`gh pr diff [N or URL]` if the `gh` CLI is available, or fetch the PR's file list and diff another way). Either form is a "PR target" for every rule below.
- For a local diff: `git diff main...HEAD` (or the appropriate base branch).

If you cannot obtain a diff, stop and tell the reviewer — do not review from memory or assumption.

---

## Step 2 — Build the Changed Files Inventory

List every changed file with its change type. Keep this table — it drives every later step.

| File | Change type | Placement Map category (RULES.md §1) |
|---|---|---|
| [path] | Added / Modified / Renamed / Deleted | [category, or "Unclassified"] |

If any file is **Unclassified**, flag it now: this repo has no established slot for it, so the PR is introducing a new category. That's not automatically wrong, but it must be explained in the PR description — check for that explanation before continuing.

---

## Step 3 — Scope discipline check

Look at the categories touched in the inventory.

- **Pass:** all changed files belong to one Sync Set (RULES.md §2), or are pure prose fixes to a single file with no cross-references.
- **Fail (Blocking):** the diff mixes an unrelated documentation fix with a skill/rule/template change, or touches more than one distinct capability. Name the split you'd ask for.

---

## Step 4 — Sync Set verification

For each changed file that is a **Skill file**, **Rule file**, **Ops template**, or **Agent entry point/folder** (per RULES.md §1), identify its Sync Set (A/B/C/D) and verify every sibling location. Use these concrete checks — run them, don't reason about them abstractly:

**Sync Set A (skill added/renamed/removed)** — for a skill named `<name>`:
```
grep -rn "<name>" readme-content/repository-contents.md
grep -rn "<name>" repository-agents/process-skills-agent/skills-guide.md
grep -rn "<name>" readme-content/skills-for-bespoke-processes.md
grep -rn "<name>" readme-content/agents.md
grep -rn "<name>" repository-agents/process-onboarding-agent/setup-guide.md
```
Every command must return a hit for an addition/rename (old name must return **zero** hits after a rename), and zero hits for a removal. If the skill count in `readme-content/agents.md` or `readme-content/skills-for-bespoke-processes.md` didn't change on an add/remove, that's a Blocking finding.

**Sync Set B (new agent)**:
```
grep -n "process-<name>-agent" README.md
grep -n "process-<name>-agent" readme-content/agents.md
```
Both must hit for a new agent folder.

**Sync Set C (template field changed)** — for a field named `<field>`:
```
grep -rn "<field>" repository-agents/process-diagnostic-agent/review-guide.md
grep -rn "<field>" repository-agents/process-onboarding-agent/setup-guide.md
```
If the field is renamed or removed from the template but still appears verbatim (old name) in either file, that's Blocking.

**Sync Set D (rule file changed)**:
```
grep -rn "<rule-name>" readme-content/repository-contents.md
grep -rn "<rule-name>" repository-agents/process-onboarding-agent/setup-guide.md
```

**Sync Set E (base repo's own master rule file changed)**:
```
diff CLAUDE.md .github/copilot-instructions.md
```
Read any diff output rather than treating it as automatically Blocking. `CLAUDE.md`, `.github/copilot-instructions.md`, and `.cursor/rules/ai-dlc-base-governance.mdc`'s body (everything after its `---` frontmatter block, modulo that frontmatter and its `../../`-prefixed relative links) must stay **behaviorally equivalent** — same sections, same triggers, same routing targets — not byte-identical. A tool-specific wording change that leaves the routing behavior unchanged is a Pass. A diff that adds, removes, or changes a section, a routing target, or a trigger in only one file is Blocking, even if the other two still read fine on their own — because the three tools would then behave differently on the same repo.

Record each check's result in a table:

| Sync Set | Member checked | Updated in this diff? | Verdict |
|---|---|---|---|
| A | skills-guide.md | Yes/No | Pass/Blocking |

A grep hit here means the *name* still appears somewhere in the sibling. It does not mean the sibling still describes the file correctly — that's a separate, mandatory check in Step 5. Do not mark a Sync Set member "Pass" from the grep result alone.

---

## Step 5 — Full-content comprehension pass

Grep tells you a string is present or absent. It cannot tell you whether a catalogue entry still describes what the skill *does*, whether a protocol still reads coherently end-to-end, or whether a rewritten paragraph quietly changed what a rule means while leaving every keyword intact. This step catches what Step 4 structurally cannot — read every file below in full; do not infer content from a diff hunk or a grep line.

**1. Read every file in the Changed Files Inventory in full**, not just the diff hunk. A localized edit can look correct in isolation while contradicting a claim made earlier or later in the *same* file that the diff didn't touch — a step count, a cross-reference to "the previous section," a list the edit forgot to extend.

**2. For every Sync Set sibling Step 4 found a hit on, read the full surrounding section — not just the matched line — and judge whether it is still *true*, not just still *present*:**
- A skill's catalogue entry (`skills-guide.md`'s "What you get" / dependency classification / invoke phrase, `repository-contents.md`'s one-line description) must still match what the skill file actually does after the edit. Naming the skill is not the same as describing it correctly.
- A rule or template reference in `review-guide.md` or `setup-guide.md` must still match the current field's name, shape, and semantics — not merely share a substring with it.
- Any count or classification (e.g. the skill count in `agents.md` / `skills-for-bespoke-processes.md`, a skill's `◆`/`◈`/`◇` dependency marker) must be recomputed against the actual current state, never assumed unchanged because grep didn't flag it.

**3. Read each touched protocol/guide file end-to-end for narrative coherence:** do step numbers, section references, and "read X before Y" instructions still resolve correctly after the edit? A single-step edit can silently break a numbered sequence, an internal cross-reference, or a routing instruction elsewhere in the same file without tripping any grep.

**4. Where the *meaning* of a source file changed — not just its wording** (a skill now requires different input, a rule's exception case changed, a template field's purpose shifted) **— check every place that meaning is paraphrased elsewhere in the repo**, not just where the file is named or linked. A grep for the file's name or path will not catch a paraphrase like "the release checklist never gates the release" sitting in a completely unrelated file — if that behavior changed, the paraphrase is now wrong even though it names nothing you searched for. This is the one check in this skill that requires judgment, not a command — read broadly (`agents.md`, `repository-contents.md`, and any file whose topic overlaps the changed one) rather than trusting that "no grep hit" means "no claim to check."

Classify each finding: **Blocking** if a sibling now asserts something false about the changed file's actual behavior; **Advisory** if it's a wording/coherence nit that doesn't change what a reader would conclude.

---

## Step 6 — Orphan check

For any **renamed or deleted** file, search the whole repo for its old path and old invoke phrase:
```
grep -rn "<old-path-or-filename>" . --include="*.md"
```
Any hit outside the diff itself is a Blocking finding — an orphaned reference.

---

## Step 7 — Style and tone spot-check

Open the changed file(s) and compare against a sibling file in the same category (e.g. a new skill against an existing skill in `skills/`). Check:
- Written as literal instructions for an AI agent, not descriptive prose about what a human should do
- Matches the section structure of its siblings (e.g. skill files with a "When to invoke" / "How to invoke" / "What you get" shape, if that's the local convention)
- No project-specific stack, domain, or team assumptions leaked into `skills/`, `rules/`, or `ops/**/_template.md` content (RULES.md §4 generic test)

Findings here are **Advisory** unless the leak is a hard-stop violation (RULES.md §5), in which case they're Blocking.

---

## Step 8 — Testing evidence check

Per CONTRIBUTING.md's testing section, a change to a skill/rule/agent protocol should have been validated by copying the folder into a scratch project and running it. Check the PR description for evidence of this (or an explanation of why it wasn't needed, e.g. a pure typo fix). Missing evidence on a behavioral change is **Advisory** — note it, don't block on it alone, but call it out clearly.

---

## Step 9 — Produce the PR Review Report

```
### PR Review — [PR # or diff description]

**Verdict:** Ready to merge / Changes needed

---

#### Changed Files Inventory
[table from Step 2]

#### Scope Discipline
[Pass, or the specific split needed]

#### Sync Set Verification
[table from Step 4 — omit if no Sync Set applies]

#### Content Comprehension Findings
[Per Step 5: for each Sync Set sibling read in full, one line — "accurate" or the specific mismatch found. Omit if no Sync Set applies.]

#### Orphan Check
[Pass, or list of orphaned references found]

#### Findings

[BLOCKING / ADVISORY] — [Step / area]
Observed: [what's in the diff]
Rule violated: [exact RULES.md section]
Required change: [specific file(s) and what to add/fix]

---

#### Verdict Rationale
[1-2 sentences: why Ready to merge, or what's blocking it]
```

**Verdict logic:** any Blocking finding → "Changes needed". Zero Blocking findings, regardless of Advisory count → "Ready to merge" (note the Advisory items for the author's discretion, don't gate on them).

---

## Step 10 — Post the review (default action for a PR target)

If Step 1's target was a PR — whether given as a number or a GitHub PR link — and the `gh` CLI is available, post the Step 9 report as a comment on that PR by default — do not just print it and stop:
```
gh pr comment [N or URL] --body-file [path to the report]
```
Confirm to the reviewer that the comment was posted and link it. If `gh` is unavailable or the target was a local/uncommitted diff (nothing to comment on), print the report instead and say why no comment was posted.

This step posts a *comment*, not a fix. Do not edit files, stage changes, or open a branch/PR to address any finding as part of this step — that only happens if separately, explicitly requested (see **Default output — post, don't fix** above).
