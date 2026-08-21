# Skill: Notifications

**Purpose:** Send Slack notifications to the team at the moments in the delivery loop that need a human — so nobody has to sit watching a session. Notifications operate on two independent layers:

1. **Harness layer (deterministic, Claude Code only).** Hooks in the project's `.claude/settings.json` fire on the AI tool's own events — `Notification` (the AI is waiting on the engineer) and `Stop` (a turn finished). These run automatically, regardless of what work is happening. They carry a generic message.
2. **Lifecycle layer (agent-driven, all tools).** The experience agent sends a rich, event-specific notification at framework moments — bolt complete, UAT sign-off required, incident logged, and so on. There is no harness event for "a bolt closed," so these are sent by the agent running the configured send command via its shell tool at the point the workflow reaches that moment.

Use the harness layer for "attention/done" pings and the lifecycle layer for "this delivery moment needs you." Both read the same environment variable — no credentials are ever written into a committed file.

**Trigger:** The harness layer fires automatically once installed. The lifecycle layer fires automatically from the master rule file routing lines (see *Lifecycle events* below). An engineer can also say "send a notification that …" to fire an ad-hoc one, or "turn notifications off for this session" to suppress both layers until the session ends.

**Dependency classification:** Needs config — the send command and hooks work as-is, but each environment must set the endpoint variable below before anything is sent.

---

## What each AI tool gets

The harness layer is a Claude Code feature. The lifecycle layer works anywhere the AI can run a terminal command, which is most but not all tools. Tell the team which row they are on before configuring anything — a Cursor or Copilot team should not expect the deterministic "the AI is waiting on you" ping.

| Tool | Harness layer (`Notification` / `Stop` hooks) | Lifecycle layer (framework events) | How to stop it prompting |
|---|---|---|---|
| **Claude Code** | Yes — hooks in `.claude/settings.json` fire automatically | Yes | `permissions.allow` entry in `.claude/settings.json` |
| **Cursor** | No — no equivalent hook mechanism | Yes, in agent mode (it can run terminal commands) | Add `scripts/notify.sh` to the agent terminal-command allowlist in Cursor Settings |
| **GitHub Copilot** | No — no equivalent hook mechanism | Only in **agent mode**; completions and plain inline chat cannot run commands, so nothing is sent | Allow the command in VS Code's Copilot terminal auto-approve settings |

Two things to be explicit about with non-Claude-Code teams:

- **No deterministic ping.** Without the harness layer, every notification depends on the agent reaching the routing line and choosing to act on it. It is best-effort by construction — reliable enough for milestones, not a substitute for an alert you must not miss.
- **The routing line has to be in *their* rule file.** The lifecycle layer is driven by the master rule file, so Section 6's routing line and Section 10 must exist in the mirror the tool actually loads — `.cursorrules` or `.github/copilot-instructions.md`, not just `CLAUDE.md`. See the setup guide's mirror-file step; a stale mirror silently disables notifications for that tool.

Environment variables are often *easier* on these tools, because an interactive integrated terminal loads the shell profile, so `SLACK_WEBHOOK_URL` from `~/.zshrc` is visible. Do not rely on it: some agent integrations run commands through a non-interactive shell (`zsh -c`), which does not source `~/.zshrc`. The GUI-launch problem described below applies to any tool started from a Dock or Start-menu icon — including Claude Code's own shell tool, not just its hooks. Verify with a test send rather than assuming.

---

## Configuration — the endpoint comes from an environment variable

Notifications are sent to whatever endpoint is present in the environment. **Nothing is ever hardcoded into a framework file or committed** — a Slack incoming-webhook URL is a bearer credential and must be treated as a secret.

| Variable | Channel | Example value |
|---|---|---|
| `SLACK_WEBHOOK_URL` | Slack incoming webhook | `https://hooks.slack.com/services/T…/B…/…` |

Rules:
- Each engineer, CI runner, or environment sets its own variable (in their shell profile, `.envrc`, or CI secrets store). Leave it unset and notifications become a silent no-op — safe by default.
- Never echo the full webhook URL back to the engineer or into any artifact. Refer to it as "the configured Slack webhook."
- If the team keeps the endpoint in a file instead of the environment, it must be gitignored (e.g. `.claude/settings.local.json`) and never committed. The environment variable is the default and recommended path.

---

## Setting the environment variable

Each engineer (and each CI runner) does this once. Nothing here is committed.

### Who does what

This is the one part of the framework the AI cannot do for the engineer, because it involves a credential. Unlike every other skill — where the engineer only answers questions — notifications needs two actions taken outside the repo. Be explicit about that instead of stalling or improvising:

| Task | Who | Why |
|---|---|---|
| Create the Slack app and incoming webhook (Step 1) | **Engineer** | A browser flow under their own Slack login. The AI has no browser and no Slack session. |
| Put `SLACK_WEBHOOK_URL` in a shell profile, `.envrc`, or `settings.local.json` (Step 2) | **Engineer** | The AI would have to be told the URL to write it, which puts the credential in the conversation. |
| Verify with a test send (Step 3) | **AI** | Reads the variable from its own environment; never sees the value. |
| CI / shared-runner secrets (Step 4) | **Engineer** | Another credential, another web UI. |
| Create `scripts/notify.sh`, allowlist it, write `.claude/settings.json`, write Section 10 and the Section 6 routing line | **AI** | Ordinary file work — these hold only the *name* of the variable. |
| Send notifications from then on | **AI** | The point of the skill. |

**Do no part of the Slack side yourself.** Do not create or configure a Slack app, do not call the Slack API, do not open or ask anyone to open a browser on your behalf, and do not ask whether the engineer has permission to install apps — that is theirs to deal with, not yours to gate. Your entire role in Steps 1 and 2 is to present the instructions, then wait.

**Never ask the engineer to paste the webhook URL into the conversation**, and never offer to edit their shell profile for them. If they paste it anyway, do not repeat it back, do not write it to any file, and tell them it is now in the session transcript and should be rotated in Slack (**Incoming Webhooks → remove the webhook, add a new one**).

Present Steps 1 and 2 as a short checklist the engineer can follow at their own pace, then stop and wait for them to say it is done. Pick up at Step 3. If they are not ready — no time, no app-install permission, waiting on an admin — record `Status: Disabled` in Section 10, tell them it can be enabled later by running these same steps, and move on. Never block onboarding on it.

### Step 1 — Obtain the Slack webhook URL

*The engineer does this — see* Who does what *above.*

1. Go to <https://api.slack.com/apps> and click **Create New App → From scratch** (or open an existing app). Pick the target workspace.
2. Open **Incoming Webhooks** and toggle it **On**.
3. Click **Add New Webhook to Workspace**, choose the channel to post to, and **Allow**.
   - Many workspaces restrict who may install apps. If this step needs approval, the request goes to a workspace admin and you wait for them — that is normal, not a misconfiguration. Tell the AI to leave notifications disabled for now and come back to it once approval lands.
4. Copy the generated URL — its shape is `https://hooks.slack.com/services/<WORKSPACE_ID>/<WEBHOOK_ID>/<TOKEN>`, three path segments after `/services/`. This whole URL is a secret; treat it like a password. The channel is fixed at webhook creation time — to notify a different channel, create a second webhook.

> Deliberately no realistic-looking example above, and do not add one. A dummy webhook URL whose path segments imitate the real format (a `T…` workspace id, a `B…` webhook id, a long alphanumeric token) matches GitHub's secret-scanning pattern for Slack webhooks, so push protection rejects the commit — in this repo and in every project that copies this file. Keep placeholders in `<ANGLE_BRACKET>` form.

### Step 2 — Set the variable so it persists

**macOS / Linux — zsh** (default on modern macOS): add to `~/.zshrc`
```bash
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/T…/B…/…"
```
Then reload: `source ~/.zshrc`

**macOS / Linux — bash:** same `export` line in `~/.bashrc` (or `~/.bash_profile`), then `source ~/.bashrc`.

**Per-project instead of global** — use [direnv](https://direnv.net): put the `export` line in a `.envrc` at the repo root, run `direnv allow`, and **add `.envrc` to `.gitignore`**. The variable loads only inside that project directory.

**Windows — PowerShell:** `setx SLACK_WEBHOOK_URL "https://hooks.slack.com/services/…"`, then open a new terminal so the value is picked up.

> Do **not** put this `export` line in any file that gets committed (not the master rule file, not `.claude/settings.json`, not a checked-in `.env`). Shell profiles and gitignored `.envrc` files stay on the machine.

**If the AI tool is launched from a GUI** (the Claude Code desktop app, or an IDE started from the Dock or Start menu), it does not load your shell profile, so `SLACK_WEBHOOK_URL` will be unset and every send silently no-ops. Two options: launch the tool from a terminal where the variable is set, or put it in a **gitignored** `.claude/settings.local.json`:
```json
{ "env": { "SLACK_WEBHOOK_URL": "https://hooks.slack.com/services/…" } }
```
`settings.local.json` holds the real URL — it is the only artifact in this design that does. **Add `.claude/settings.local.json` to `.gitignore` before writing the URL into it**, then verify with `git check-ignore .claude/settings.local.json` (it must echo the path back). Never put the URL in `settings.json`.

### Step 3 — Verify

Open a new terminal (so the profile is loaded) and confirm the variable is visible and the endpoint works:
```bash
# Confirm the variable is set (does not print the URL)
echo "${SLACK_WEBHOOK_URL:+slack set}"

# Send a test message
[ -n "$SLACK_WEBHOOK_URL" ] && curl -sf --max-time 5 -X POST -H 'Content-Type: application/json' \
  --data '{"text":"AI-DLC notifications test"}' "$SLACK_WEBHOOK_URL" && echo " slack ok"
```
A `slack ok` line and the message arriving in the Slack channel confirms setup.

### New teammates joining later

The variable is per machine, so notifications do not follow a person into the team — a new engineer gets nothing until they set it on their own machine. And because an unset variable is a deliberate silent no-op, **nothing warns them**: their sessions simply never notify, and they have no reason to suspect the feature exists.

So a joining engineer does not repeat Step 1. The webhook belongs to the channel, not to a person — one webhook is shared by the whole team:

1. Get the existing webhook URL from wherever the team keeps shared secrets (password manager, vault, CI secrets store). Ask a teammate; do not create a second webhook for the same channel, and do not send the URL over Slack or email.
2. Do Step 2 only — put it in their own shell profile or `.envrc`.
3. Do Step 3 to confirm a test message arrives.

Create a *new* webhook only when the team wants a different channel. If the team keeps no shared secret store, say so plainly during onboarding: the URL will end up copied between people ad hoc, and it should be rotated whenever someone leaves.

The `new-engineer-induction` skill prompts for this automatically, so a joining engineer is asked rather than left to discover it.

### Step 4 — CI and shared runners (optional)

For notifications from CI or a shared/cron runner, store the same value in the platform's secrets store and expose it as an environment variable to the job — never in the repo:
- **GitHub Actions:** add repo secret `SLACK_WEBHOOK_URL`, then map it under the step's `env:` block.
- **GitLab CI:** add a masked CI/CD variable of the same name.
- Any runner: set it as an environment variable in the runner/agent configuration.

---

## The send command

Every send — both layers — goes through one script, `scripts/notify.sh` (installed during onboarding; see *Onboarding setup*, step 3). The script builds the JSON payload itself, so **the agent never escapes anything for JSON**:

```bash
scripts/notify.sh 'Message text, exactly as it should appear in Slack'
```

Run it from the repository root. Do not write `./scripts/notify.sh` or `bash scripts/notify.sh` — the permission allowlist matches the bare `scripts/notify.sh` form, and the other spellings will prompt.

**The single rule the agent must follow:** the argument is single-quoted, so any apostrophe inside the message must be written `'\''` (close quote, escaped quote, reopen quote) — or simply use a typographic `’` instead. Nothing else needs escaping: `$`, backticks, double quotes, backslashes, and newlines all pass through untouched, because the shell does not re-expand a single-quoted argument and the script does the JSON encoding.

For a message that is awkward to quote, pass it on stdin with a quoted heredoc instead — no escaping at all:

```bash
scripts/notify.sh <<'MSG'
Message text with 'apostrophes', "quotes", $dollars — all literal.
MSG
```

**One rendering rule, not an escaping rule:** Slack parses `&`, `<`, and `>` in message text as markup. If a message would contain them literally (e.g. an intent named `orders <v2>`), replace `&` with `&amp;`, `<` with `&lt;`, and `>` with `&gt;` first. Do this *before* prepending any `<!here>` or emoji prefix, so the prefix itself is not escaped. Note also that `*` and `_` in a name will render as bold/italic — harmless, but say so if a team asks why a unit name looks odd.

High-priority events prefix the message with `:rotating_light: ` so they stand out in the channel; normal events have no prefix. If the team wants a channel-wide alert on high-priority events, add `<!here> ` after the prefix — ask before enabling it, it is noisy.

---

## Message format

Every lifecycle message follows one shape so alerts are scannable:

```
[<ProjectName>] <Event> — <one-line detail>. <Action needed>
```

Examples:
- `[Acme] UAT sign-off required — Intent "Checkout v2" has all units Done. Run UAT to close it.`
- `[Acme] Bolt complete — "Payments hardening" (4/4 units Done). Retro is due.`
- `:rotating_light: [Acme] Incident logged — "Orders API 500s in prod" (Sev-1). Hotfix bolt started.`
- `:rotating_light: [Acme] Circuit breaker — unit "apply-coupon" output rejected 3× on the same failure. Execution paused.`
- `:rotating_light: [Acme] Dependency audit due — scheduled for today. Run it before other work.`

`<ProjectName>` is read from Section 1 of the master rule file.

**Keep sensitive detail out of the message.** A webhook posts into a Slack channel that may have a wider audience than the delivery team — name the intent, unit, or incident, but do not paste customer data, credentials, stack traces, or log excerpts. The detail belongs in the incident or unit file, which the message points to implicitly.

---

## Lifecycle events (the default notify set)

The agent sends a lifecycle notification when it reaches any of these moments. This set is the default; the project may add or remove events in the master rule file Notifications section.

| Event | When it fires | Priority |
|---|---|---|
| **Elaboration sign-off required** | The elaboration unit summary table is ready and the AI is waiting for engineer sign-off | high |
| **Bolt complete** | The last unit in a bolt is marked Done | normal |
| **UAT sign-off required** | All units under an intent are Done and UAT has not yet run | high |
| **Intent implemented** | An intent moves to Implemented and its Implementation Summary is written | normal |
| **Incident logged / hotfix started** | An incident file is created or a hotfix bolt begins | high |
| **Circuit breaker tripped** | The engagement circuit breaker fires (output rejected 3× on the same failure) | high |
| **Dependency audit due** | Session start on or after the `Next dependency audit` date in Section 9 | high |

Sending is best-effort and must never interrupt the workflow: send the notification, then continue the step. Do not wait for or report the curl result unless the engineer asked for an ad-hoc send.

**Expect some overlap with the harness layer, and keep the lifecycle events anyway.** Claude Code's `Notification` hook does *not* fire the moment the AI asks a prose question — it fires on a tool-permission request, and when the prompt has been idle for about a minute. So at an elaboration or UAT sign-off the lifecycle event is the notification that actually arrives on time; the hook may add a second, vaguer ping a minute later if the engineer does not respond. Two messages at those moments is the correct trade — do not remove the sign-off events from Section 10 to avoid it, or the only ping left is the delayed one.

---

## Onboarding setup (run once, during installation)

The onboarding agent performs these steps when the engineer opts into notifications:

**1. Ask whether the team wants notifications.**

> "Do you want Slack notifications for delivery moments that need a human — bolt complete, UAT sign-off, incidents, and so on? (You can skip this and add it later.)"

If the engineer declines, record "Notifications: disabled" in the master rule file Notifications section and stop here.

**2. Hand the engineer Steps 1 and 2 of *Setting the environment variable* and wait — these two are theirs, not yours (see *Who does what*). Never ask them to paste the webhook URL into the conversation or into a committed file.** Then: create the incoming webhook (Step 1), persist `SLACK_WEBHOOK_URL` in the shell profile or a gitignored `.envrc` for the correct platform (Step 2), and verify with a test send (Step 3). Each teammate does this on their own machine; CI runners use the secrets store (Step 4). If the variable is left unset, notifications are simply skipped.

**3. Install the send script.** Every send — both layers — goes through this one script, so JSON encoding lives in exactly one place and the command can be allowlisted once. Create it at **`scripts/notify.sh`, at the repository root** — not under `{FRAMEWORK_ROOT}`, and not under `.claude/`; it must be tool-neutral and reachable at a stable relative path.

```bash
#!/usr/bin/env bash
# Usage: scripts/notify.sh 'message text'   (or pipe the message on stdin)
# Takes RAW text — the caller does not escape anything for JSON.
[ -n "$SLACK_WEBHOOK_URL" ] || { echo "SLACK_WEBHOOK_URL not set — notification skipped" >&2; exit 0; }

if [ "$#" -gt 0 ]; then msg="$1"; else msg="$(cat)"; fi

if command -v jq >/dev/null 2>&1; then
  payload="$(printf '%s' "$msg" | jq -Rs '{text: .}')"
elif command -v python3 >/dev/null 2>&1; then
  payload="$(printf '%s' "$msg" | python3 -c 'import json,sys; print(json.dumps({"text": sys.stdin.read()}))')"
else
  echo "notify.sh needs jq or python3 to build the payload — notification skipped" >&2
  exit 0
fi

curl -sf --max-time 5 -X POST -H 'Content-Type: application/json' \
  --data "$payload" "$SLACK_WEBHOOK_URL" >/dev/null || true
```

Then `chmod +x scripts/notify.sh` and commit it — it contains no secret, only the variable reference. Notes on the design, which are worth preserving if the script is edited:

- **The script does the JSON encoding, not the caller.** Hand-escaping JSON in prose instructions is error-prone in exactly the way that fails silently: a malformed payload gets a 400 from Slack, `curl -f` fails, `|| true` swallows it, and nobody learns the notification was dropped. `jq -Rs` / `json.dumps` cannot get it wrong.
- **`--max-time 5`** keeps the promise that sending never blocks a step. Without it, a black-holed connection to `hooks.slack.com` stalls the turn for the full TCP timeout.
- **An unset variable exits 0 but says so on stderr**, so the ad-hoc "send a notification that …" path has something truthful to report instead of claiming success. Exit stays 0 so a Claude Code hook never surfaces an error.
- **POSIX shell required.** On Windows this needs Git Bash or WSL; plain PowerShell cannot run it.
- If the project already keeps scripts under another name (`bin/`, `tools/`), put it there instead and use that exact path everywhere — the allowlist rule and Section 10 must match it character for character.

**4. Stop the send prompting for permission.** The lifecycle layer runs the script through the AI's shell/terminal tool, so by default the tool asks the engineer to approve every notification — which defeats the point of being notified. Approve it once, per tool:

- **Claude Code** — add the allow rule to `.claude/settings.json` (see the combined file in step 5; do not write it as a separate file that overwrites the hooks block):
  ```json
  { "permissions": { "allow": ["Bash(scripts/notify.sh:*)"] } }
  ```
  The rule matches the bare `scripts/notify.sh …` form only. `./scripts/notify.sh` and `bash scripts/notify.sh` will still prompt.
- **Cursor** — add `scripts/notify.sh` to the agent's allowlist of terminal commands in Cursor Settings, so agent mode runs it without a confirmation.
- **GitHub Copilot** — allow the command in the VS Code terminal auto-approve settings for Copilot agent mode (`chat.tools.terminal.autoApprove`, or the equivalent in your version). Copilot must be in **agent mode**; completions and plain inline chat cannot run commands at all, so the lifecycle layer will never fire there.

If the team declines to allowlist anything, notifications still work — they just prompt before each send. Say so plainly rather than leaving it as a surprise.

**5. Install the harness layer (Claude Code only).** Create or merge into `.claude/settings.json` at the repo root. Both the hook and the allow rule from step 4 belong in this one file:

```json
{
  "permissions": {
    "allow": ["Bash(scripts/notify.sh:*)"]
  },
  "hooks": {
    "Notification": [
      {
        "hooks": [
          { "type": "command", "command": "scripts/notify.sh 'Claude Code needs your attention'" }
        ]
      }
    ]
  }
}
```

Notes:
- **If `.claude/settings.json` already exists, merge into it** — add the `Notification` array and the `allow` entry to what is there. Never write either block as a fresh file over the top of the other; step 4 and step 5 both touch this file.
- The hook calls the same script rather than its own inline `curl`, so there is one implementation to maintain and the hook inherits the timeout and payload encoding. It therefore depends on step 3 having run — do step 3 first.
- To also ping when a turn finishes, add the same `hooks` array under a `"Stop"` key — but `Stop` fires at the end of **every** turn, not just at the end of a piece of work. Offer it; do not install it by default.
- For Cursor and GitHub Copilot there is no equivalent hook mechanism — skip this step entirely and record "Harness hooks: not available (Cursor / Copilot)" in Section 10. See *What each AI tool gets* above.
- `.claude/settings.json` contains no secrets (only the variable reference), so it is safe to commit. Do **not** commit `.claude/settings.local.json` or any other file holding the actual URL.

**6. Wire the lifecycle layer into the master rule file** (Section 6 routing line and Section 10 — see the setup guide).

**7. Confirm.** Send one test notification whose text exercises the characters real messages contain — parentheses, an em-dash, an apostrophe, and a quoted name — so quoting and the allowlist are verified against the real shape, not a benign string:

```bash
scripts/notify.sh ':rotating_light: [<ProjectName>] Notifications configured — test of unit '\''apply-coupon'\'' (Sev-1) "quoted".'
```

Ask the engineer to confirm the message arrived in the Slack channel and that it was sent **without a permission prompt**. If it prompted, step 4 did not take effect.

---

## Turning it off

- **Per session:** the engineer says "turn notifications off for this session" — the agent suppresses both layers (skips lifecycle sends) until the session ends.
- **Per project:** set the Notifications section in the master rule file to "disabled," remove the `.claude/settings.json` hooks, and unset the environment variable.
