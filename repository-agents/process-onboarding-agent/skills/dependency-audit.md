# Skill: Dependency Audit

**Purpose:** Audit the project's third-party dependencies for major version drift, known vulnerability patterns, and end-of-life packages. Produces a ranked findings list and converts actionable findings into Remediation Bolts in the backlog. Runs on a scheduled cadence — at least once a month — to catch what accumulates passively between active development.

**Trigger:** Engineer-initiated, or prompted automatically by the AI at session start when the `Next dependency audit` date in the master rule file Process Configuration has been reached. Invoked by saying "run dependency audit" or "run security posture check."

**Scheduling:** The next scheduled audit date is stored in the master rule file Section 9 (Process Configuration). The AI checks this date at the start of every session. If today is on or after the scheduled date, the AI prompts the engineer before any other work begins:

> "A dependency and security audit is scheduled for today. Would you like to run it now, or set a new date to defer it?"

The engineer may run the audit immediately or defer it to a specific date. Either way, the scheduled date is updated before the session continues.

**Important limitation:** The AI analyses manifests using training knowledge and cannot browse live vulnerability databases. For accurate, current CVE data the engineer must run the appropriate ecosystem security tool and share the output — this is Step 3 of the skill. The AI's role is to interpret, classify, and convert findings into backlog items — not to replace purpose-built scanning tools.

---

## Step 1 — Confirm Scope

Ask the engineer:

> "Should I audit all dependency manifests in the project, or focus on a specific subset? (e.g. backend only, a specific service)"

Read the `Next dependency audit` and `Last dependency audit` dates from the master rule file Section 9. Confirm:

> "Last audit: [date or 'none recorded']. I'll now read the dependency manifests and prepare for the scan."

Do not ask further questions — proceed directly to Step 2.

---

## Step 2 — Read Dependency Manifests

Locate and read all dependency manifest files in the repository. Common locations by ecosystem:

| Ecosystem | Files to read |
|---|---|
| Node.js | `package.json`, `package-lock.json` or `yarn.lock` |
| Python | `requirements.txt`, `requirements-dev.txt`, `pyproject.toml`, `Pipfile.lock` |
| Ruby | `Gemfile`, `Gemfile.lock` |
| Go | `go.mod`, `go.sum` |
| Java / Kotlin | `pom.xml`, `build.gradle`, `build.gradle.kts` |
| .NET | `*.csproj`, `packages.lock.json` |
| Rust | `Cargo.toml`, `Cargo.lock` |

Read every manifest found within the agreed scope. Extract:
- Package name
- Currently pinned version
- Whether it is a production dependency or a development/test-only dependency

Produce an internal inventory list (do not present it to the engineer yet — it is input for Steps 3 and 4).

---

## Step 3 — Request the Tool-Assisted Scan

Ask the engineer to run their ecosystem's security scanning tool and share the output. Present the correct command for the detected ecosystem(s):

> "To get current CVE data, please run the security scanner for your stack and paste the output here. I'll classify the findings and create backlog items from them.
>
> Recommended commands for your stack:
> - **Node.js:** `npm audit` or `yarn audit`
> - **Python:** `pip-audit` or `safety check`
> - **Ruby:** `bundler-audit check --update`
> - **Go:** `govulncheck ./...`
> - **Java:** `mvn dependency-check:check` or `gradle dependencyCheckAnalyze`
> - **.NET:** `dotnet list package --vulnerable`
> - **Rust:** `cargo audit`
>
> If you use a third-party scanner (Snyk, Dependabot, OWASP Dependency-Check), share that output instead. If no scanner is available right now, type 'skip' to proceed with AI-only analysis."

Wait for the engineer to paste the scanner output or type 'skip'. Do not proceed to Step 4 until the engineer responds.

---

## Step 4 — AI Analysis

Using training knowledge, analyse the dependency inventory from Step 2 for the following — independently of what the security scanner found:

**Major version drift:** For each dependency, assess whether the pinned version is more than one major version behind the commonly known current stable release (as of training knowledge). Flag packages that are multiple major versions behind.

**End-of-life and deprecated packages:**
- Runtimes past their official EOL date (e.g. Node.js LTS versions, Python versions)
- Packages with known deprecation notices (e.g. `request` for Node.js, `mock` for Python pre-3.x)
- Packages superseded by a successor with a different name (note the recommended replacement)

**Known vulnerability patterns from training knowledge:**
- Flag any dependency-version combination known to have had a significant CVE or security incident, even if the scanner did not flag it. Examples: very old versions of `lodash`, `log4j`, `jackson-databind`, `struts`, `openssl`. Note that this knowledge has a cutoff date and is not a substitute for live scanning.

**Development dependency risk:**
- Flag any package that is listed as a dev dependency but is also resolvable from production code (a common misconfiguration that exposes dev tools in production builds).

---

## Step 5 — Classify and Rank All Findings

Combine findings from the tool-assisted scan (Step 3) and AI analysis (Step 4). Deduplicate — if the scanner and AI both flagged the same package, merge into one entry.

Classify each finding by severity:

| Severity | Criteria |
|---|---|
| **Critical** | Known CVE with a CVSS score ≥ 9.0, or active exploit known; or production dependency that is completely unmaintained and has no viable alternative |
| **High** | Known CVE with CVSS 7.0–8.9; or runtime/framework at EOL with no upgrade path confirmed |
| **Medium** | Major version drift ≥ 2 versions behind; deprecated package with a known replacement; dev dependency exposed in production build |
| **Low** | Major version drift of 1 version; minor version significantly behind latest; advisory with no confirmed exploit |

Produce the findings table:

```
Dependency Audit Findings — YYYY-MM-DD

| # | Package | Current version | Issue | Severity | Source | Recommended action |
|---|---|---|---|---|---|---|
| 1 | [name] | [version] | [description] | Critical / High / Medium / Low | Scanner / AI | [upgrade to X / replace with Y / remove / investigate] |
```

Sort by severity (Critical first). Within each severity level, sort production dependencies before dev dependencies.

---

## Step 6 — Present and Confirm Findings

Present the ranked findings table to the engineer. Then ask:

> "Which of these findings should I convert to Remediation Bolts in the backlog? You can say 'all Critical and High', list specific numbers, or say 'none' to record the findings without creating backlog items."

Wait for the engineer's response before creating any backlog items.

---

## Step 7 — Create Remediation Bolts

For each finding the engineer approved, create an intent and associated unit in the backlog using the Remediation Bolt classification from Phase M1.4.

For each item:

1. Create a unit file at `process-onboarding-agent/ops/build/units/YYYY-MM-DD-<unix_timestamp>-dep-[package-slug].md` with:
   - **Status:** Open
   - **Context:** the dependency finding in plain language
   - **Acceptance Criteria:**
     - `Given the production build runs, when [package] is loaded, then it is version [target version] or later.`
     - `Given the CI pipeline runs, when the security scan executes, then no [severity] or higher vulnerabilities are reported for [package].`
   - **Priority:** Critical finding → High; High finding → High; Medium/Low → Medium or Low

2. Add the unit to the Open section of `process-onboarding-agent/ops/build/backlog.md` using a reference-style link.

After creating all units, summarise:

```
Remediation units created: [N]
  Critical: [N] units
  High:     [N] units
  Medium:   [N] units
  Low:      [N] units

Added to backlog: [list unit names]
```

---

## Step 8 — Schedule the Next Audit and Update Process Configuration

Ask the engineer:

> "When would you like to schedule the next dependency audit? The recommended interval is 30 days. You can give a specific date or say 'in [N] weeks/months'."

Convert the answer to an absolute date (YYYY-MM-DD). If the engineer says "default" or does not specify, set the next date to 30 days from today.

Update the master rule file Section 9 (Process Configuration) — specifically the `Last dependency audit` and `Next dependency audit` rows. If these rows do not yet exist in the table, add them.

```markdown
| **Last dependency audit** | YYYY-MM-DD | Date this audit was completed |
| **Next dependency audit** | YYYY-MM-DD | AI prompts at session start on or after this date |
```

Confirm completion:

```
Dependency audit complete.

Findings:        [N] total ([N] Critical, [N] High, [N] Medium, [N] Low)
Backlog items:   [N] Remediation units added
Findings skipped: [N] (engineer chose not to create backlog items)

Next audit scheduled: YYYY-MM-DD
Process Configuration updated in master rule file.
```
