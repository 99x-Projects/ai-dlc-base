# Skill: Notifications

**Purpose:** Send Slack notifications to the engineer at the moments in the delivery loop that need a human — so nobody has to sit watching a session. This is a **local, per-engineer setup**: each person creates their own Slack channel and their own webhook, on their own machine, and is notified about their own sessions. It is not a team broadcast channel, and no webhook URL is ever shared or committed. Notifications operate on two independent layers:

1. **Harness layer (deterministic).** Hooks in the tool's own config fire on the tool's lifecycle events — a turn ending (work done, question asked, awaiting the next prompt) and needing attention (permission request, idle prompt). All three supported tools have these; the event names and config file differ. They run automatically regardless of what work is happening, so they cannot be forgotten — but they carry a generic message, because a hook does not know what a bolt is.
2. **Lifecycle layer (agent-driven, all tools).** The experience agent sends a rich, event-specific notification at framework moments — bolt complete, UAT sign-off required, incident logged, and so on. There is no harness event for "a bolt closed," so these are sent by the agent running the configured send command via its shell tool at the point the workflow reaches that moment.

Use the harness layer for "attention/done" pings and the lifecycle layer for "this delivery moment needs you." Both send through the same script and resolve the same endpoint — no credentials are ever written into a committed file.

**Trigger:** The harness layer fires automatically once installed. The lifecycle layer fires automatically from the master rule file routing lines (see *Lifecycle events* below). An engineer can also say "send a notification that …" to fire an ad-hoc one, or "turn notifications off for this session" to suppress both layers until the session ends.

**Dependency classification:** Needs config — the send command and hooks work as-is, but each engineer must supply their own endpoint (Step 2) before anything is sent.

---

## Invariants

Anyone editing this file — human or AI — must keep all six of these true. They are the things earlier revisions got wrong, and each one fails silently when broken:

1. **One webhook per engineer, pointing at their own channel.** No shared URL, no shared secret store, no project-wide endpoint. The single exception is a CI/cron runner, which gets its own webhook on a team channel (Step 4).
2. **The AI never touches Slack.** It does not create apps, call the Slack API, ask for the URL, or gate on whether the engineer may install apps. It presents instructions and waits.
3. **`Status: Disabled` is an exact token.** `new-engineer-induction.md` string-matches it. Never write "Notifications: disabled" or any other spelling.
4. **`.gitignore` before the URL exists.** `scripts/notify.env`, `.envrc` and `.claude/settings.local.json` are gitignored by the AI up front — these are the only files that may ever hold the URL, and the only remaining path by which it could be committed.
5. **The script escapes, the caller does not.** `scripts/notify.sh` takes raw text and builds the JSON itself. Never reintroduce hand-escaping rules for the agent to follow.
6. **Every send exits 0.** An unset variable, an empty message, a missing `jq`/`python3`, a failed or timed-out request — all are silent no-ops. A notification must never break the step it reports on.

---

## What each AI tool gets

**All three supported tools have lifecycle hooks.** Claude Code has had them longest; Cursor added them in 1.7 and GitHub Copilot's are in preview, so treat the two newer ones as more likely to move — *verified September 2026, re-check before relying on an exact event name.*

The event names and config files differ; the role does not. All three call the same `scripts/notify.sh`, which is why the send script is deliberately tool-neutral and not under `.claude/`.

| | **Claude Code** | **Cursor** | **GitHub Copilot** |
|---|---|---|---|
| Hook config | `.claude/settings.json` | `.cursor/hooks.json` (or `~/.cursor/hooks.json`) | `.github/hooks/*.json` (or `~/.copilot/hooks/`) |
| Turn ended — work done, question asked, awaiting your next prompt | `Stop` | `stop` | `agentStop` |
| Needs permission / attention | `Notification` | `beforeShellExecution` | `permissionRequest`, `notification` |
| Session over | — | `sessionEnd` | `sessionEnd` |
| Subagent finished | `SubagentStop` | `subagentStop` | `subagentStop` |
| Lifecycle layer (framework events) | Yes | Yes, in agent mode | Agent mode only — completions and inline chat cannot run commands |
| Stop the send prompting | `permissions.allow` in `.claude/settings.json` | agent terminal-command allowlist in Cursor Settings | VS Code Copilot terminal auto-approve |

Per-tool gotchas:

- **Cursor CLI is not the Cursor IDE.** Community reports say the CLI omits `stop`, `afterAgentResponse` and `beforeSubmitPrompt` while the IDE emits them. A headless `cursor-agent` run may notify nothing. Verify with a test send rather than assuming.
- **Copilot takes separate commands per platform** — a hook entry has both `bash` and `powershell` keys. That makes it the only one of the three with cross-platform dispatch built into the format.
- **Copilot's cloud agent reads only `.github/hooks/*.json`** from the cloned repo, not user-level config.
- **Copilot needs agent mode** for the lifecycle layer at all; completions and plain inline chat cannot run a command.

### Why there are still two layers

Not because some tools lack hooks — they all have them now. The layers answer different questions:

- **The harness layer is deterministic but semantically blind.** The tool fires it, so it cannot be forgotten; but a hook has no idea what a *bolt* is. It knows a turn ended, not that a bolt closed and a retro is due.
- **The lifecycle layer is semantic but best-effort.** It knows exactly which delivery moment was reached, because the agent reached it — which is also why it depends on the agent noticing the routing line.

Use both. Neither substitutes for the other.

One thing that still needs saying to Cursor and Copilot teams: **the routing line has to be in *their* rule file.** The lifecycle layer is driven by the master rule file, so the Section 6 routing line and Section 10 must exist in the mirror the tool actually loads — `.cursorrules` or `.github/copilot-instructions.md`, not just `CLAUDE.md`. A stale mirror silently disables the lifecycle layer for that tool.

Environment variables are often *easier* on these tools, because an interactive integrated terminal loads the shell profile. Do not rely on it: some agent integrations run commands through a non-interactive shell, which reads no startup file on bash and a different one on every platform. That is why Step 2 Option A (`scripts/notify.env`) is the recommended setup for every tool.

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
- The endpoint is normally kept in a gitignored `scripts/notify.env`, which the send script reads directly — this is the only setup that behaves identically on macOS, Windows and Linux. An environment variable takes precedence over the file when both are present, which is how CI supplies it.

---

## Setting the environment variable

Each engineer (and each CI runner) does this once. Nothing here is committed.

### Who does what

This is the one part of the framework the AI cannot do for the engineer, because it involves a credential. Unlike every other skill — where the engineer only answers questions — notifications needs two actions taken outside the repo. Be explicit about that instead of stalling or improvising:

| Task | Who | Why |
|---|---|---|
| Create their own channel, Slack app, and incoming webhook (Step 1) | **Engineer** | A browser flow under their own Slack login, in their own workspace. The AI has no browser and no Slack session. |
| Write the endpoint into `scripts/notify.env` (or set the variable — Step 2) | **Engineer** | The AI would have to be told the URL to write it, which puts the credential in the conversation. |
| Verify with a test send (Step 3) | **AI** | Reads the variable from its own environment; never sees the value. |
| CI / shared-runner secrets (Step 4) | **Engineer** | Another credential, another web UI. |
| Add `scripts/notify.env`, `.envrc` and `.claude/settings.local.json` to `.gitignore` | **AI** | The one file edit that prevents a leak. Do it up front, before the engineer has a URL to put anywhere — not after. |
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

### Step 2 — Tell the script where to send

Two ways. The first works identically on macOS, Windows and Linux and needs no shell knowledge — prefer it, and offer the second only to engineers who ask for it.

#### Option A (recommended) — `scripts/notify.env`

Create a file next to the send script containing one line:

```sh
SLACK_WEBHOOK_URL="<YOUR_WEBHOOK_URL>"
```

That is the whole setup. The script reads this file when the variable is not already in the environment. It is gitignored (the AI adds it in onboarding step 2), so it is never committed.

**Why this is the default:** an AI tool runs commands in a *non-interactive* shell, and that shell reads a different startup file on every platform — or, on bash, none at all:

| Platform / shell | What a non-interactive shell reads |
|---|---|
| macOS / Linux, **zsh** | `~/.zshenv` only (**not** `~/.zshrc` — that is interactive-only) |
| Linux / Git Bash, **bash** | **nothing.** Non-interactive bash reads no startup file. `BASH_ENV` would have to already be in the environment to be read. |
| Windows PowerShell / cmd | the user environment from the registry (`setx`) |

So "put an export in your shell profile" is wrong on two of the three rows, and the failure is silent — the engineer's terminal works while the AI's shell sees nothing. The env file sidesteps all of it.

#### Option B — an environment variable, if the engineer prefers it

Correct per platform. Get this wrong and it appears to work in a terminal while every AI-driven send silently no-ops.

- **macOS / Linux, zsh:** add `export SLACK_WEBHOOK_URL="<YOUR_WEBHOOK_URL>"` to **`~/.zshenv`** — not `~/.zshrc`. `.zshenv` is read on every zsh invocation; `.zshrc` only for interactive ones. Then `chmod 600 ~/.zshenv`.
- **Linux, bash or mixed shells:** put it in `~/.config/environment.d/notifications.conf` as `SLACK_WEBHOOK_URL=<YOUR_WEBHOOK_URL>` (systemd user environment, inherited by GUI apps and their children after the next login). `~/.profile` covers login shells only.
- **macOS, shell-agnostic:** `launchctl setenv SLACK_WEBHOOK_URL "<YOUR_WEBHOOK_URL>"` makes it visible to GUI-launched apps, but it is lost on reboot unless installed as a LaunchAgent.
- **Windows:** `setx SLACK_WEBHOOK_URL "<YOUR_WEBHOOK_URL>"`, then open a new terminal. This is user-level and shell-agnostic, so it covers PowerShell, cmd and Git Bash. Note the send script is POSIX `sh`, so **running** it needs Git Bash or WSL.
- **Per-project instead of global:** [direnv](https://direnv.net) with the export in a gitignored `.envrc`. Convenient in a terminal, but direnv hooks the *interactive* shell, so an AI tool's shell will usually not see it. Use Option A instead.

> Never put the URL in any file that gets committed — not the master rule file, not `.claude/settings.json`, not a checked-in `.env`. `scripts/notify.env`, `.envrc`, and `.claude/settings.local.json` are all gitignored; nothing else may hold it.

#### Claude Code only — `.claude/settings.local.json`

If the team is on Claude Code and would rather keep it in tool config than a script-adjacent file:

```json
{ "env": { "SLACK_WEBHOOK_URL": "<YOUR_WEBHOOK_URL>" } }
```

Gitignored, and it reaches every shell Claude Code spawns. Do not offer this on Cursor or Copilot projects: `env` there is a Claude Code setting, so those tools never read it and the engineer would be writing a live credential to disk for nothing.

### Step 3 — Verify

Run this from the repository root. It exercises whichever route Step 2 used, and it checks the **HTTP status**, not the exit code:

```sh
scripts/notify.sh 'AI-DLC notifications test'
```

If nothing arrives, the script says why on stderr — either no endpoint was found, or `jq`/`python3` is missing. It always exits 0, so **never treat exit 0 as proof of delivery**: a failed or timed-out request exits 0 too, by design, because a notification must never break the step it reports on.

To confirm delivery rather than infer it, ask Slack directly:

```sh
[ -f scripts/notify.env ] && . scripts/notify.env
curl -s -o /dev/null -w 'HTTP %{http_code}\n' --max-time 5 \
  -X POST -H 'Content-Type: application/json' \
  --data '{"text":"AI-DLC notifications test 2"}' "$SLACK_WEBHOOK_URL"
```

`HTTP 200` (body `ok`) means Slack accepted it. Anything else — `403`, `404`, a timeout — means the webhook is wrong or revoked, not that the script is broken.

**The check that actually matters:** run the send from a *non-interactive* shell, because that is what the AI tool uses. If Step 2 Option A was used this cannot fail, which is the point of preferring it. If the engineer chose Option B, verify it explicitly:

```sh
zsh -c 'scripts/notify.sh "non-interactive test"'    # zsh users
bash -c 'scripts/notify.sh "non-interactive test"'   # bash users
```

An `SLACK_WEBHOOK_URL not set` here while the plain terminal works is the classic symptom: the export is in an interactive-only startup file (`~/.zshrc`, or `~/.bashrc` on bash). Move it to `~/.zshenv`, or switch to Option A.

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

**Expect overlap with the harness layer, and keep both.** With the turn-ended hook installed, asking for sign-off *is* the end of a turn, so the harness fires there too — you will get two messages at a sign-off moment. That is the correct trade, not a defect to tune away:

- The **harness** message is instant and generic: "finished its turn — waiting for you." It cannot be missed, and it cannot tell you what for.
- The **lifecycle** message says *which* delivery moment and *what is needed*: "UAT sign-off required — intent 'Checkout v2' has all units Done."

Delete the lifecycle sign-off events and you keep only the ping that does not say why. Delete the turn-ended hook and you depend on the agent remembering. Neither is worth the saving of one Slack line.

The attention hook (`Notification` / `beforeShellExecution` / `permissionRequest`) is different again — it fires on a permission request or an idle prompt, not when a question is asked, so it rarely doubles up with anything.

---

## Onboarding setup (run once, during installation)

The onboarding agent performs these steps when the engineer opts into notifications:

**1. Ask whether the engineer wants notifications.**

> "Do you want Slack notifications for delivery moments that need a human — bolt complete, UAT sign-off, incidents, and so on? It's a personal setup: your own channel, your own webhook, on this machine. Each teammate does their own, and nothing gets committed. (You can skip this and add it later.)"

Section 10 records whether *this project* fires the lifecycle events at all; each engineer's own environment variable decides whether they personally receive them. So enabling Section 10 does not switch anything on for anyone else, and a teammate who never sets the variable is unaffected.

If the engineer declines, write Section 10 with **`Status: Disabled`** — that exact token, since `new-engineer-induction.md` matches on it — and stop here.

**2. Close the leak paths first, before the engineer has a URL in hand.** Add `scripts/notify.env`, `.envrc` and `.claude/settings.local.json` to the project's `.gitignore` (create the file if there is none), and confirm with `git check-ignore scripts/notify.env .envrc .claude/settings.local.json`. This is yours to do, and doing it now means there is no window in which a webhook URL could land in a tracked file.

**3. Hand the engineer Steps 1 and 2 of *Setting the environment variable* and wait — these two are theirs, not yours (see *Who does what*). Never ask them to paste the webhook URL into the conversation or into a committed file.** Then: they create their own channel and incoming webhook pointing at it (Step 1), persist `SLACK_WEBHOOK_URL` in their shell profile or a gitignored `.envrc` for the correct platform (Step 2), and you verify with a test send (Step 3). Every teammate repeats this on their own machine with their own channel — see *New teammates joining later*. If the variable is left unset, notifications are simply skipped.

**4. Install the send script.** Every send — both layers — goes through this one script, so JSON encoding lives in exactly one place and the command can be allowlisted once. Create it at **`scripts/notify.sh`, at the repository root** — not under `{FRAMEWORK_ROOT}`, and not under `.claude/`; it must be tool-neutral and reachable at a stable relative path.

```bash
#!/bin/sh
# Usage: scripts/notify.sh 'message text'   (or pipe the message on stdin)
# Takes RAW text — the caller does not escape anything for JSON.
# Endpoint resolution, in order:
#   1. SLACK_WEBHOOK_URL already in the environment (CI, or an engineer who set it there)
#   2. scripts/notify.env next to this script — gitignored, one line, works on every OS/shell
# The file exists because a non-interactive shell (which is what an AI tool spawns) reads no
# startup file on bash, and a different one on every platform. Reading it here removes the
# whole problem: no shell configuration anywhere, same behaviour on macOS, Windows and Linux.
if [ -z "$SLACK_WEBHOOK_URL" ]; then
  env_file="$(dirname "$0")/notify.env"
  [ -f "$env_file" ] && . "$env_file"
fi
[ -n "$SLACK_WEBHOOK_URL" ] || { echo "SLACK_WEBHOOK_URL not set (no environment variable, no scripts/notify.env) — notification skipped" >&2; exit 0; }

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

**6. Install the harness layer.** Every supported tool has hooks — install them, do not offer them. Two events by default: **turn ended** and **needs attention**. The turn-ended hook is the one engineers actually want: it fires when the agent finishes work, asks a question, or stops for the next prompt, which is the whole point of not watching a session.

Use the config file for the team's tool (see *What each AI tool gets*). Prefix every message with the project name so a person watching several repos can tell them apart.

**Claude Code** — `.claude/settings.json`, merged with the `allow` entry from step 5:
```json
{
  "permissions": { "allow": ["Bash(scripts/notify.sh:*)"] },
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/notify.sh ':white_check_mark: [<ProjectName>] Claude finished its turn — waiting for you.'" } ] }
    ],
    "Notification": [
      { "hooks": [ { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/notify.sh ':bell: [<ProjectName>] Claude needs your attention — permission request or idle prompt.'" } ] }
    ]
  }
}
```

**Cursor** — `.cursor/hooks.json`:
```json
{
  "version": 1,
  "hooks": {
    "stop": [ { "command": "scripts/notify.sh ':white_check_mark: [<ProjectName>] Cursor finished its turn — waiting for you.'" } ],
    "beforeShellExecution": [ { "command": "scripts/notify.sh ':bell: [<ProjectName>] Cursor is asking to run a command.'" } ]
  }
}
```

**GitHub Copilot** — `.github/hooks/notify.json`:
```json
{
  "version": 1,
  "hooks": {
    "agentStop": [ { "type": "command",
      "bash": "scripts/notify.sh ':white_check_mark: [<ProjectName>] Copilot finished its turn — waiting for you.'",
      "powershell": "sh scripts/notify.sh ':white_check_mark: [<ProjectName>] Copilot finished its turn — waiting for you.'" } ],
    "permissionRequest": [ { "type": "command",
      "bash": "scripts/notify.sh ':bell: [<ProjectName>] Copilot needs your approval.'",
      "powershell": "sh scripts/notify.sh ':bell: [<ProjectName>] Copilot needs your approval.'" } ]
  }
}
```

Notes:

- **Merge, never overwrite.** If the tool's config file already exists, add these keys to what is there. For Claude Code, step 5 and step 6 both touch `.claude/settings.json`; for Cursor and Copilot the file may already hold unrelated hooks.
- **Turn-ended fires on every turn.** That is the intent — it is the event that replaces watching the session — but say the number out loud during onboarding: a forty-turn session is forty messages. If the team finds it too much, drop the turn-ended hook and keep the attention hook, which fires only when the agent is actually blocked on them.
- **Claude Code needs `$CLAUDE_PROJECT_DIR`; the others take a repo-relative path.** Hook commands have no guaranteed working directory, and Claude Code is the one with an explicit variable for the project root. Cursor resolves a project hook's relative path from the project root; Copilot accepts a `cwd` key if you need to pin it. Get this wrong and the hook dies with "No such file or directory" — silently, because hook failures are swallowed.
- **Hooks are not governed by the send-command allowlist.** `permissions.allow` (or the equivalent) covers the agent's own shell tool, not the harness. That is why the hook path and the call-site path are spelled differently on purpose.
- Hooks call the same `scripts/notify.sh`, so they inherit the timeout, the payload encoding, and the silent-no-op behaviour. Step 4 must have run first.
- **If `.claude/` (or the tool's config directory) is gitignored in this project**, the hooks cannot be shared — they become per-engineer setup like the endpoint. Record that in Section 10 and add it to the induction steps, or a new teammate silently gets nothing.
- None of these files contain a secret — only the script path and message text — so they are safe to commit wherever the project's ignore rules allow it.

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
