# Skill: Notifications

**Purpose:** Send Slack notifications to the engineer at the moments in the delivery loop that need a human — so nobody has to sit watching a session. This is a **local, per-engineer setup**: each person creates their own Slack channel and their own webhook, on their own machine, and is notified about their own sessions. It is not a team broadcast channel, and no webhook URL is ever shared or committed. Notifications operate on two independent layers:

1. **Harness layer (deterministic, Claude Code only).** Hooks in the project's `.claude/settings.json` fire on the AI tool's own events — `Notification` (the AI is waiting on the engineer) and `Stop` (a turn finished). These run automatically, regardless of what work is happening. They carry a generic message.
2. **Lifecycle layer (agent-driven, all tools).** The experience agent sends a rich, event-specific notification at framework moments — bolt complete, UAT sign-off required, incident logged, and so on. There is no harness event for "a bolt closed," so these are sent by the agent running the configured send command via its shell tool at the point the workflow reaches that moment.

Use the harness layer for "attention/done" pings and the lifecycle layer for "this delivery moment needs you." Both read the same environment variable — no credentials are ever written into a committed file.

**Trigger:** The harness layer fires automatically once installed. The lifecycle layer fires automatically from the master rule file routing lines (see *Lifecycle events* below). An engineer can also say "send a notification that …" to fire an ad-hoc one, or "turn notifications off for this session" to suppress both layers until the session ends.

**Dependency classification:** Needs config — the send command and hooks work as-is, but each environment must set the endpoint variable below before anything is sent.

---

## Invariants

Anyone editing this file — human or AI — must keep all six of these true. They are the things earlier revisions got wrong, and each one fails silently when broken:

1. **One webhook per engineer, pointing at their own channel.** No shared URL, no shared secret store, no project-wide endpoint. The single exception is a CI/cron runner, which gets its own webhook on a team channel (Step 4).
2. **The AI never touches Slack.** It does not create apps, call the Slack API, ask for the URL, or gate on whether the engineer may install apps. It presents instructions and waits.
3. **`Status: Disabled` is an exact token.** `new-engineer-induction.md` string-matches it. Never write "Notifications: disabled" or any other spelling.
4. **`.gitignore` before the URL exists.** `.envrc` and `.claude/settings.local.json` are gitignored by the AI up front — this is the only remaining path by which a webhook URL could be committed.
5. **The script escapes, the caller does not.** `scripts/notify.sh` takes raw text and builds the JSON itself. Never reintroduce hand-escaping rules for the agent to follow.
6. **Every send exits 0.** An unset variable, an empty message, a missing `jq`/`python3`, a failed or timed-out request — all are silent no-ops. A notification must never break the step it reports on.

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
| `SLACK_WEBHOOK_URL` | Slack incoming webhook | `https://hooks.slack.com/services/<WORKSPACE_ID>/<WEBHOOK_ID>/<TOKEN>` |

Rules:
- **This is per engineer, per machine.** Each person points their own webhook at their own Slack channel and sets the variable locally (shell profile or `.envrc`). There is no project-wide endpoint, no shared URL, and nothing to distribute — a teammate who never sets the variable simply gets no notifications. A CI or cron runner is the one exception, and it gets its own webhook rather than borrowing anyone's (Step 4).
- Leave the variable unset and notifications become a silent no-op — safe by default.
- Never echo the full webhook URL back to the engineer or into any artifact. Refer to it as "the configured Slack webhook."
- If an engineer keeps the endpoint in a file instead of the environment, it must be gitignored (e.g. `.claude/settings.local.json`) and never committed. The environment variable is the default and recommended path.

---

## Setting the environment variable

Each engineer (and each CI runner) does this once. Nothing here is committed.

### Who does what

This is the one part of the framework the AI cannot do for the engineer, because it involves a credential. Unlike every other skill — where the engineer only answers questions — notifications needs two actions taken outside the repo. Be explicit about that instead of stalling or improvising:

| Task | Who | Why |
|---|---|---|
| Create their own channel, Slack app, and incoming webhook (Step 1) | **Engineer** | A browser flow under their own Slack login, in their own workspace. The AI has no browser and no Slack session. |
| Put `SLACK_WEBHOOK_URL` in a shell profile, `.envrc`, or `settings.local.json` (Step 2) | **Engineer** | The AI would have to be told the URL to write it, which puts the credential in the conversation. |
| Verify with a test send (Step 3) | **AI** | Reads the variable from its own environment; never sees the value. |
| CI / shared-runner secrets (Step 4) | **Engineer** | Another credential, another web UI. |
| Add `.envrc` and `.claude/settings.local.json` to `.gitignore` | **AI** | The one file edit that prevents a leak. Do it up front, before the engineer has a URL to put anywhere — not after. |
| Create `scripts/notify.sh`, allowlist it, write `.claude/settings.json`, write Section 10 and the Section 6 routing line | **AI** | Ordinary file work — these hold only the *name* of the variable. |
| Send notifications from then on | **AI** | The point of the skill. |

**Do no part of the Slack side yourself.** Do not create or configure a Slack app, do not call the Slack API, do not open or ask anyone to open a browser on your behalf, and do not ask whether the engineer has permission to install apps — that is theirs to deal with, not yours to gate. Your entire role in Steps 1 and 2 is to present the instructions, then wait.

**Never ask the engineer to paste the webhook URL into the conversation**, and never offer to edit their shell profile for them. If they paste it anyway, do not repeat it back, do not write it to any file, and tell them it is now in the session transcript and should be rotated in Slack (**Incoming Webhooks → remove the webhook, add a new one**).

Present Steps 1 and 2 as a short checklist the engineer can follow at their own pace, then stop and wait for them to say it is done. Pick up at Step 3. If they are not ready — no time, no app-install permission, waiting on an admin — record `Status: Disabled` in Section 10, tell them it can be enabled later by running these same steps, and move on. Never block onboarding on it.

### Step 1 — Create your channel and webhook

*The engineer does this — see* Who does what *above.* This is a personal setup: your own channel, your own webhook, notifying you about your own sessions. Teammates do not share either.

1. In Slack, create the channel you want to be notified in. A private channel with only you in it is the normal choice; name it for yourself (`#ai-dlc-alice`) so it is obvious what it is. Do not point this at a team channel unless the whole team has agreed to the noise.
2. Go to <https://api.slack.com/apps> and click **Create New App → From scratch** (or open an existing app). Pick the target workspace.
3. Open **Incoming Webhooks** and toggle it **On**.
4. Click **Add New Webhook to Workspace**, choose the channel you just created, and **Allow**.
   - Many workspaces restrict who may install apps. If this step needs approval, the request goes to a workspace admin and you wait for them — that is normal, not a misconfiguration. Tell the AI to leave notifications disabled for now and come back to it once approval lands.
5. Copy the generated URL — its shape is `https://hooks.slack.com/services/<WORKSPACE_ID>/<WEBHOOK_ID>/<TOKEN>`, three path segments after `/services/`. This whole URL is a secret; treat it like a password. The channel is fixed at webhook creation time — to notify a different channel, create a second webhook.

> Deliberately no realistic-looking example above, and do not add one. A dummy webhook URL whose path segments imitate the real format (a `T…` workspace id, a `B…` webhook id, a long alphanumeric token) matches GitHub's secret-scanning pattern for Slack webhooks, so push protection rejects the commit — in this repo and in every project that copies this file. Keep placeholders in `<ANGLE_BRACKET>` form.

### Step 2 — Set the variable so it persists

**macOS / Linux — zsh** (default on modern macOS): add to `~/.zshrc`
```bash
export SLACK_WEBHOOK_URL="<YOUR_WEBHOOK_URL>"
```
Then reload: `source ~/.zshrc`

**macOS / Linux — bash:** same `export` line in `~/.bashrc` (or `~/.bash_profile`), then `source ~/.bashrc`.

**Per-project instead of global** — use [direnv](https://direnv.net): put the `export` line in a `.envrc` at the repo root, run `direnv allow`, and **add `.envrc` to `.gitignore`**. The variable loads only inside that project directory.

**Windows — PowerShell:** `setx SLACK_WEBHOOK_URL "<YOUR_WEBHOOK_URL>"`, then open a new terminal so the value is picked up.

> Do **not** put this `export` line in any file that gets committed (not the master rule file, not `.claude/settings.json`, not a checked-in `.env`). Shell profiles and gitignored `.envrc` files stay on the machine.

**If the AI tool is launched from a GUI** (a desktop app, or an IDE started from the Dock or Start menu), it does not load your shell profile, so `SLACK_WEBHOOK_URL` will be unset and every send silently no-ops.

- **Any tool:** launch it from a terminal where the variable is already set. This is the simplest fix and the only one that needs no extra file.
- **Any tool, OS-level:** set the variable where GUI apps can see it — `launchctl setenv SLACK_WEBHOOK_URL "…"` on macOS (per login session; add it to a login item to persist), or the user environment variables dialog / `setx` on Windows.
- **Claude Code only:** put it in a **gitignored** `.claude/settings.local.json`:
  ```json
  { "env": { "SLACK_WEBHOOK_URL": "<YOUR_WEBHOOK_URL>" } }
  ```
  `settings.local.json` holds the real URL — it is the only artifact in this design that does. **Add `.claude/settings.local.json` to `.gitignore` before writing the URL into it**, then verify with `git check-ignore .claude/settings.local.json` (it must echo the path back). Never put the URL in `settings.json`. Do not offer this option on Cursor or Copilot projects: `env` in that file is a Claude Code setting, so those tools never read it — the engineer would be writing a live credential to disk for no benefit.

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

### Step 4 — CI and shared runners (optional, and the one shared exception)

A CI job or cron runner is not a person, so it is the deliberate exception to everything above: it needs a **separate webhook of its own, pointing at a team channel**, owned by the team.

**Never put an engineer's personal webhook in a CI secret.** A repo-level secret is readable by every workflow and every collaborator with the right access, the alerts would land in a channel only one person can see, and revoking it would break that person's local setup too. Create a second webhook for a shared channel (e.g. `#ai-dlc-ci`) and use that.

Then store it in the platform's secrets store and expose it as an environment variable to the job — never in the repo:
- **GitHub Actions:** add repo secret `SLACK_WEBHOOK_URL`, then map it under the step's `env:` block.
- **GitLab CI:** add a masked CI/CD variable of the same name.
- Any runner: set it as an environment variable in the runner/agent configuration.

Because that channel does have an audience, the sensitive-detail rule under *Message format* matters more there than it does in a personal channel.


### New teammates joining later

This is a **personal, per-machine setup, not a team channel.** Each engineer has their own Slack channel and their own webhook pointing at it, so the notifications they get are about the work in *their* session, on *their* machine. Nobody shares a webhook URL, and nothing about it is committed or centrally configured.

That has one consequence worth stating up front: a new engineer gets no notifications until they set this up themselves, and because an unset variable is a deliberate silent no-op, **nothing warns them**. Their sessions simply never notify, and they have no reason to suspect the feature exists.

So a joining engineer runs the **full sequence from Step 1**, exactly as the first engineer did:

1. Create their own channel to be notified in — a private channel, or a channel with just them in it. Naming it for themselves (`#ai-dlc-alice`) keeps it obvious.
2. Create their own Slack app and incoming webhook pointing at that channel (Step 1 above).
3. Set `SLACK_WEBHOOK_URL` on their own machine (Step 2), and confirm with a test send (Step 3).

**Never hand a webhook URL to a teammate**, and never ask one for theirs. It is a personal credential like an SSH key: if two people share it, one person's session noise lands in the other's channel, and revoking it cuts off both. There is no shared secret store to keep, because there is no shared secret.

The `new-engineer-induction` skill prompts for this automatically, so a joining engineer is asked rather than left to discover it.

---

## The send command

Every send — both layers — goes through one script, `scripts/notify.sh` (installed during onboarding; see *Onboarding setup*, step 4). The script builds the JSON payload itself, so **the agent never escapes anything for JSON**:

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

**One rendering rule, not an escaping rule:** Slack parses `&`, `<`, and `>` in message text as markup. If a message would contain them literally (e.g. an intent named `orders <v2>`), replace `&` with `&amp;`, `<` with `&lt;`, and `>` with `&gt;` first. Do this *before* prepending the emoji prefix, so the prefix itself is not escaped. Note also that `*` and `_` in a name will render as bold/italic — harmless, but say so if a team asks why a unit name looks odd.

High-priority events prefix the message with `:rotating_light: ` so they stand out in the channel; normal events have no prefix. Do not add `<!here>` or `<!channel>`: the destination is the engineer's own channel, so there is nobody else to alert — it just badges them a second time in their own channel. (The exception is a webhook deliberately pointed at a shared channel, such as the CI webhook in Step 4.)

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

**Keep sensitive detail out of the message.** The destination is the engineer's own channel, so the audience is normally just them — but the message still leaves the machine, crosses Slack's servers, and stays in that channel's history and their phone's notification shade. Name the intent, unit, or incident; do not paste customer data, credentials, stack traces, or log excerpts. The detail belongs in the incident or unit file, which the message points to implicitly.

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

**1. Ask whether the engineer wants notifications.**

> "Do you want Slack notifications for delivery moments that need a human — bolt complete, UAT sign-off, incidents, and so on? It's a personal setup: your own channel, your own webhook, on this machine. Each teammate does their own, and nothing gets committed. (You can skip this and add it later.)"

Section 10 records whether *this project* fires the lifecycle events at all; each engineer's own environment variable decides whether they personally receive them. So enabling Section 10 does not switch anything on for anyone else, and a teammate who never sets the variable is unaffected.

If the engineer declines, write Section 10 with **`Status: Disabled`** — that exact token, since `new-engineer-induction.md` matches on it — and stop here.

**2. Close the leak paths first, before the engineer has a URL in hand.** Add `.envrc` and `.claude/settings.local.json` to the project's `.gitignore` (create the file if there is none), and confirm with `git check-ignore .envrc .claude/settings.local.json`. This is yours to do, and doing it now means there is no window in which a webhook URL could land in a tracked file.

**3. Hand the engineer Steps 1 and 2 of *Setting the environment variable* and wait — these two are theirs, not yours (see *Who does what*). Never ask them to paste the webhook URL into the conversation or into a committed file.** Then: they create their own channel and incoming webhook pointing at it (Step 1), persist `SLACK_WEBHOOK_URL` in their shell profile or a gitignored `.envrc` for the correct platform (Step 2), and you verify with a test send (Step 3). Every teammate repeats this on their own machine with their own channel — see *New teammates joining later*. If the variable is left unset, notifications are simply skipped.

**4. Install the send script.** Every send — both layers — goes through this one script, so JSON encoding lives in exactly one place and the command can be allowlisted once. Create it at **`scripts/notify.sh`, at the repository root** — not under `{FRAMEWORK_ROOT}`, and not under `.claude/`; it must be tool-neutral and reachable at a stable relative path.

```bash
#!/bin/sh
# Usage: scripts/notify.sh 'message text'   (or pipe the message on stdin)
# Takes RAW text — the caller does not escape anything for JSON.
[ -n "$SLACK_WEBHOOK_URL" ] || { echo "SLACK_WEBHOOK_URL not set — notification skipped" >&2; exit 0; }

if [ "$#" -gt 0 ]; then
  msg="$*"                      # join all args, so a forgotten quote truncates nothing
elif [ -t 0 ]; then
  msg=""                        # interactive with no argument: nothing to send, do not block on cat
else
  msg="$(cat)"                  # message piped in
fi
[ -n "$msg" ] || { echo "notify.sh: empty message — notification skipped" >&2; exit 0; }

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
- **Plain POSIX shell**, hence `#!/bin/sh` — nothing here needs bash. On Windows it needs Git Bash or WSL; plain PowerShell cannot run it.
- **No message is a no-op, not an error.** Called with no argument and nothing piped in, it says so on stderr and exits 0 rather than hanging on `cat` waiting for input that never comes.
- If the project already keeps scripts under another name (`bin/`, `tools/`), put it there instead and use that exact path everywhere — the allowlist rule and Section 10 must match it character for character.

**5. Stop the send prompting for permission.** The lifecycle layer runs the script through the AI's shell/terminal tool, so by default the tool asks the engineer to approve every notification — which defeats the point of being notified. Approve it once, per tool:

- **Claude Code** — add the allow rule to `.claude/settings.json` (see the combined file in step 6; do not write it as a separate file that overwrites the hooks block):
  ```json
  { "permissions": { "allow": ["Bash(scripts/notify.sh:*)"] } }
  ```
  The rule matches the bare `scripts/notify.sh …` form only. `./scripts/notify.sh` and `bash scripts/notify.sh` will still prompt.
- **Cursor** — add `scripts/notify.sh` to the agent's allowlist of terminal commands in Cursor Settings, so agent mode runs it without a confirmation.
- **GitHub Copilot** — allow the command in the VS Code terminal auto-approve settings for Copilot agent mode (`chat.tools.terminal.autoApprove`, or the equivalent in your version). Copilot must be in **agent mode**; completions and plain inline chat cannot run commands at all, so the lifecycle layer will never fire there.

If the team declines to allowlist anything, notifications still work — they just prompt before each send. Say so plainly rather than leaving it as a surprise.

**6. Install the harness layer (Claude Code only).** Create or merge into `.claude/settings.json` at the repo root. Both the hook and the allow rule from step 5 belong in this one file:

```json
{
  "permissions": {
    "allow": ["Bash(scripts/notify.sh:*)"]
  },
  "hooks": {
    "Notification": [
      {
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/notify.sh 'Claude Code needs your attention'" }
        ]
      }
    ]
  }
}
```

Notes:
- **If `.claude/settings.json` already exists, merge into it** — add the `Notification` array and the `allow` entry to what is there. Never write either block as a fresh file over the top of the other; step 5 and step 6 both touch this file.
- **The hook spells the path differently from the call sites, on purpose.** Hook commands do not run with a guaranteed working directory, so a bare `scripts/notify.sh` dies with "No such file or directory" — silently, since the hook swallows failures — for anyone who starts a session in a subdirectory. `$CLAUDE_PROJECT_DIR` always resolves to the repo root. The call sites keep the bare relative form because that is what the allowlist matches, and hooks are not governed by `permissions.allow` at all.
- The hook calls the same script rather than its own inline `curl`, so there is one implementation to maintain and the hook inherits the timeout and payload encoding. It therefore depends on step 4 having run — do step 4 first.
- To also ping when a turn finishes, add the same `hooks` array under a `"Stop"` key — but `Stop` fires at the end of **every** turn, not just at the end of a piece of work. Offer it; do not install it by default.
- For Cursor and GitHub Copilot there is no equivalent hook mechanism — skip this step entirely and record "Harness hooks: not available (Cursor / Copilot)" in Section 10. See *What each AI tool gets* above.
- `.claude/settings.json` contains no secrets (only the variable reference), so it is safe to commit. Do **not** commit `.claude/settings.local.json` or any other file holding the actual URL.

**7. Wire the lifecycle layer into the master rule file** (Section 6 routing line and Section 10 — see the setup guide).

**8. Confirm.** Send one test notification (substituting the real project name for `<ProjectName>`) whose text exercises the characters real messages contain — parentheses, an em-dash, an apostrophe, and a quoted name — so quoting and the allowlist are verified against the real shape, not a benign string:

```bash
scripts/notify.sh ':rotating_light: [<ProjectName>] Notifications configured — test of unit '\''apply-coupon'\'' (Sev-1) "quoted".'
```

Ask the engineer to confirm the message arrived in the Slack channel and that it was sent **without a permission prompt**. If it prompted, step 5 did not take effect.

---

## Turning it off

- **Per session:** the engineer says "turn notifications off for this session" — the agent suppresses both layers (skips lifecycle sends) until the session ends.
- **Just for you:** unset `SLACK_WEBHOOK_URL` in your shell profile. Nobody else is affected — the endpoint is per engineer.
- **For the whole project:** set Section 10 to **`Status: Disabled`** (that exact token) so the lifecycle events stop firing for everyone, and remove the `.claude/settings.json` hooks.
