# Skill: AI Hub Metrics

**Purpose:** Push usage statistics and activity events from repository-agents' delivery work to [99x AI Hub](https://ai-hub.99x.io) — the platform that lets a team observe AI work across a codebase: tokens, cost, model, and who participated (human, agent, or both) for each step. Each push is one **activity event**, correlated to one unit of AI-assisted delivery work (a unit, a bolt, a UAT sign-off) and attributed to the actors who did it. This is a **project-level integration, not a per-engineer one** — one Team, one Workflow, and one set of credentials cover the whole project, unlike the notifications skill.

**Trigger:** Fires automatically from the master rule file Section 6 routing line at the lifecycle events listed in Section 11 (unit Done, bolt complete, UAT sign-off, intent Implemented). An engineer can also say "push an AI Hub event for …" to send one ad-hoc; "enable AI Hub metrics" / "disable AI Hub metrics" to flip the project-wide switch (see *Enabling and disabling*); "turn off AI Hub metrics for this session" to suppress sending just for this session; or "show AI Hub status" / "is AI Hub metrics on" to see the status summary (see *Status summary*).

**Dependency classification:** Needs config — the send command works as-is once installed, but someone must first model a Workflow in AI Hub and supply four pieces of information (Step 1 below) before anything is sent.

---

## Invariants

Anyone editing this file — human or AI — must keep all six of these true:

1. **Credentials never committed.** The API key/PAT and the IDs that name where events land are read from environment variables or a gitignored file, never written into a committed file or the master rule file.
2. **The AI never touches the AI Hub console.** It does not create the team, register actors, generate a key, or model the workflow. It presents the setup steps and waits — see *Who does what*.
3. **The script escapes, the caller does not.** `scripts/ai-hub-push.sh` builds the JSON payload itself. Never reintroduce hand-escaping rules for the agent to follow.
4. **Every send exits 0.** A missing credential, an unreachable host, a non-202 response — all are silent no-ops on stderr. A metrics push must never break the delivery step it reports on.
5. **One `correlationId` per usage instance.** Reuse one across unrelated units and AI Hub aggregates them as a single instance in its dashboards — pick an id that names the thing that actually happened (a unit id, a bolt id), not a constant.
6. **Never guess the API host or the workflow IDs.** AI Hub's published documentation does not state a fixed API domain, and node/activity ids are generated per workflow. Both are confirmed by the engineer from their own AI Hub instance during setup (Step 1) — do not default or invent them.

---

## Core concepts (from AI Hub's documentation)

- **Team** — the ownership boundary for actors, workflows, API keys, and usage. One team per project is normal.
- **Actors** — team-scoped participants, optionally linked to an AI Hub user, or representing non-user systems (an AI tool counts as an actor). They appear on events for attribution and must be registered on the team before they can be pushed.
- **Workflow / Nodes / Node Activities** — a workflow models the process on a canvas; each node is a step, and each node can define activities with an actor mode of `human`, `agent`, or `human+agent`. AI-DLC's unit-based delivery loop maps onto this directly: model a workflow with nodes for the stages the team wants visibility into (e.g. "Elaboration", "Build", "UAT"), and give each node an activity in `human+agent` mode, since every unit here is a human+AI collaboration.
- **Events** — each push creates activity events keyed by a `correlationId`. `actors` records who participated; `dimensions` optionally carries `tokens`, `costUsd`, and `model` for usage totals.

---

## Configuration — the endpoint, credential, and workflow IDs come from environment variables

Nothing about *where* events go or *how they authenticate* is ever hardcoded into a framework file. All five pieces below come from `scripts/ai-hub.env` (or the environment) at send time.

| Variable | Meaning | Example value |
|---|---|---|
| `AI_HUB_BASE_URL` | The AI Hub instance's API host | `https://ai-hub.99x.io` — **confirm this against your own AI Hub console**; the public documentation does not publish it as a fixed value, so do not assume it without the engineer confirming it |
| `AI_HUB_API_KEY` **or** `AI_HUB_PAT` | Team API key (`ah_tm_…`) or personal access token (`ah_pat_…`) | one of the two — team key is the normal choice for a shared pipeline |
| `AI_HUB_NODE_ID` | The workflow node's public id | `nd_…` |
| `AI_HUB_NODE_ACTIVITY_ID` | The node activity's public id (optional — see *Which endpoint*) | `na_…` |

Rules:
- Leave any of these unset and a push becomes a silent no-op — safe by default, same as notifications.
- Never echo the API key/PAT back to the engineer or into any artifact. Refer to it as "the configured AI Hub credential."
- The values are normally kept in a gitignored `scripts/ai-hub.env`, read directly by the send script — this is the one setup that behaves identically on macOS, Windows, and Linux. An environment variable takes precedence over the file when both are present, which is how CI supplies it.
- Unlike the notifications skill, this is **one shared setup for the whole project** — the Team, Workflow, and credential belong to the project, not to an individual engineer. A team API key is the normal choice so any teammate's session can push without minting a personal token.

### Which endpoint

The script picks the endpoint from what is configured:
- **`AI_HUB_NODE_ACTIVITY_ID` set (recommended):** activity-scoped endpoint, `POST {AI_HUB_BASE_URL}/metrics/nodes/{AI_HUB_NODE_ID}/node-activities/{AI_HUB_NODE_ACTIVITY_ID}/events`. Use this when the project routes every event to one node activity (e.g. a single "Delivery" activity covering the whole loop).
- **`AI_HUB_NODE_ACTIVITY_ID` unset:** node-scoped endpoint, `POST {AI_HUB_BASE_URL}/metrics/nodes/{AI_HUB_NODE_ID}/events`, and every call must pass `--activity <name>` so the event names which activity on that node it belongs to. Use this when the project wants events split across several activities on one node (e.g. "Build" vs "UAT") without maintaining a separate `AI_HUB_NODE_ACTIVITY_ID` per call site.

Either way, expected response is `HTTP 202 Accepted`.

---

## Who does what

Like notifications, this needs two actions outside the repo that the AI cannot perform:

| Task | Who | Why |
|---|---|---|
| Create the team, register actors (including one for the AI tool itself), create a team API key, model the workflow with nodes/activities (Step 1) | **Engineer** | A browser flow under their own AI Hub login. The AI has no browser and no AI Hub session. |
| Write the five values into `scripts/ai-hub.env` (or set the variables — Step 2) | **Engineer** | The AI would have to be told the key to write it, which puts the credential in the conversation. |
| Verify with a test push (Step 3) | **AI** | Runs the script, which resolves the config itself; the AI never sees the values. |
| Add `scripts/ai-hub.env`, `.envrc`, and `.claude/settings.local.json` to `.gitignore` | **AI** | Do it up front, before the engineer has a key to put anywhere. |
| Create `scripts/ai-hub-push.sh`, allowlist it, write Section 11 and the Section 6 routing line | **AI** | Ordinary file work — none of these hold the credential, only a reference to it. |
| Send events from then on | **AI** | The point of the skill. |

**Never ask the engineer to paste the API key or PAT into the conversation.** If they paste it anyway, do not repeat it back, do not write it to any file, and tell them it is now in the session transcript and should be rotated in AI Hub (**Settings → API keys → revoke, then create a new one**).

### Step 1 — Model the workflow and get a credential

*The engineer does this.*

1. In AI Hub, open **Teams**, create (or pick) the project's team, and invite teammates.
2. Under **Actors**, register one actor per participant type this project wants attributed — at minimum, one for the engineer(s) and one representing the AI tool (e.g. "Claude Code", named per the AI tool actually in use). Assign each to the team.
3. In **Teams → API keys**, create a team key (`ah_tm_…`) for the shared pipeline. A personal access token (`ah_pat_…`) from **Settings** works too, but a team key is the better default since this is a project-wide integration, not a per-engineer one.
4. From the team's **Workflows** list, create a workflow and open the editor. Add a node for each stage the team wants visibility into. On the node(s) that should receive AI-DLC events, add an activity with actor mode `human+agent`.
5. Copy the node's public id (`nd_…`) and, if using the activity-scoped endpoint, the activity's public id (`na_…`).
6. Confirm the AI Hub API host for this instance — the documentation does not publish one fixed value, so read it from the console or ask whoever administers the AI Hub instance.

**Security note (from AI Hub's own documentation):** "Never commit API keys to version control or share them publicly. Revoke keys you no longer need."

### Step 2 — Tell the script where to send

#### Option A (recommended) — `scripts/ai-hub.env`

Create a file next to the send script:

```sh
AI_HUB_BASE_URL="<YOUR_AI_HUB_BASE_URL>"
AI_HUB_API_KEY="<YOUR_TEAM_API_KEY>"
AI_HUB_NODE_ID="<YOUR_NODE_ID>"
AI_HUB_NODE_ACTIVITY_ID="<YOUR_NODE_ACTIVITY_ID>"   # omit this line to use the node-scoped endpoint instead
```

Gitignored (the AI adds it during setup step 2 below), read by the script when the variables are not already in the environment. Same reasoning as the notifications skill applies here — a non-interactive shell (what an AI tool spawns) reads no startup file on bash and a different one on every platform, so a script-adjacent env file is the one setup that behaves identically everywhere.

#### Option B — CI / shared runner

Store the same four values in the platform's secrets store and expose them as environment variables to the job — never in the repo. A team API key is the right credential here, not a PAT, since a PAT is tied to one person's account.

#### Claude Code only — `.claude/settings.local.json`

```json
{ "env": { "AI_HUB_BASE_URL": "<...>", "AI_HUB_API_KEY": "<...>", "AI_HUB_NODE_ID": "<...>", "AI_HUB_NODE_ACTIVITY_ID": "<...>" } }
```

Gitignored, reaches every shell Claude Code spawns. Do not offer this on Cursor or Copilot projects — `env` there is a Claude Code setting only.

### Step 3 — Verify

```sh
scripts/ai-hub-push.sh --correlation-id 'ai-hub-metrics-test' --actor 'setup-check' --activity 'setup-check'
```

The script always exits 0, so **never treat exit 0 as proof of delivery** — it prints what happened on stderr. Confirm the event actually landed by opening the workflow in AI Hub: **Workflows → open the workflow → filterable event list**. Usage appears shortly after a successful push.

---

## The send command

Every push goes through one script, `scripts/ai-hub-push.sh` (installed during setup — see *Onboarding setup*). The script builds the JSON payload itself:

```bash
scripts/ai-hub-push.sh --correlation-id '<id>' --actor '<name>' [--actor '<name2>' ...] \
  [--activity '<activity-name>'] [--tokens <n>] [--cost-usd <n>] [--model '<model-name>']
```

Run it from the repository root — the permission allowlist matches the bare `scripts/ai-hub-push.sh` form.

- `--correlation-id` is required: the id of the usage instance (a unit id, a bolt id, an intent id — see *Lifecycle events* for the mapping).
- `--actor` is required, repeatable: pass one per participant (the engineer, and the actor representing the AI tool). Use the same names registered in AI Hub's Actors screen so attribution resolves.
- `--activity` is required only when `AI_HUB_NODE_ACTIVITY_ID` is unset (node-scoped endpoint); omit it when the activity is already fixed by `AI_HUB_NODE_ACTIVITY_ID`.
- `--tokens`, `--cost-usd`, `--model` are optional usage dimensions. This framework does not track token counts or cost by default — pass them only when the project separately captures that data (e.g. from the AI tool's own session/usage reporting); omit them otherwise. A push with no dimensions is still a valid, useful event — it records that the step happened and who did it.

```bash
#!/bin/sh
# Usage: scripts/ai-hub-push.sh --correlation-id '<id>' --actor '<name>' [--actor '<name2>' ...] \
#          [--activity '<name>'] [--tokens <n>] [--cost-usd <n>] [--model '<name>']
# Pushes one activity event to 99x AI Hub. Best-effort: never blocks the step it reports on.
#
# Config resolution, in order:
#   1. AI_HUB_BASE_URL / AI_HUB_API_KEY (or AI_HUB_PAT) / AI_HUB_NODE_ID / AI_HUB_NODE_ACTIVITY_ID
#      already in the environment (CI, or an engineer who set them there)
#   2. scripts/ai-hub.env next to this script — gitignored, read the same way on every OS/shell

if [ -z "$AI_HUB_BASE_URL$AI_HUB_API_KEY$AI_HUB_PAT$AI_HUB_NODE_ID" ]; then
  env_file="$(dirname "$0")/ai-hub.env"
  [ -f "$env_file" ] && . "$env_file"
fi

api_key="${AI_HUB_API_KEY:-$AI_HUB_PAT}"

if [ -z "$AI_HUB_BASE_URL" ] || [ -z "$api_key" ] || [ -z "$AI_HUB_NODE_ID" ]; then
  echo "ai-hub-push.sh: AI Hub not configured (need AI_HUB_BASE_URL, AI_HUB_API_KEY or AI_HUB_PAT, AI_HUB_NODE_ID — env or scripts/ai-hub.env) — push skipped" >&2
  exit 0
fi

correlation_id=""
activity=""
tokens=""
cost_usd=""
model=""
actors=""

while [ "$#" -gt 0 ]; do
  case "$1" in
    --correlation-id) correlation_id="$2"; shift 2 ;;
    --actor) actors="$actors${actors:+|}$2"; shift 2 ;;
    --activity) activity="$2"; shift 2 ;;
    --tokens) tokens="$2"; shift 2 ;;
    --cost-usd) cost_usd="$2"; shift 2 ;;
    --model) model="$2"; shift 2 ;;
    *) echo "ai-hub-push.sh: unknown argument '$1' — push skipped" >&2; exit 0 ;;
  esac
done

[ -n "$correlation_id" ] && [ -n "$actors" ] || {
  echo "ai-hub-push.sh: --correlation-id and at least one --actor are required — push skipped" >&2
  exit 0
}

if [ -n "$AI_HUB_NODE_ACTIVITY_ID" ]; then
  url="$AI_HUB_BASE_URL/metrics/nodes/$AI_HUB_NODE_ID/node-activities/$AI_HUB_NODE_ACTIVITY_ID/events"
else
  if [ -z "$activity" ]; then
    echo "ai-hub-push.sh: AI_HUB_NODE_ACTIVITY_ID is unset, so --activity is required — push skipped" >&2
    exit 0
  fi
  url="$AI_HUB_BASE_URL/metrics/nodes/$AI_HUB_NODE_ID/events"
fi

if command -v jq >/dev/null 2>&1; then
  payload="$(
    printf '%s\n' "$actors" | tr '|' '\n' | jq -Rs --arg cid "$correlation_id" --arg act "$activity" \
      --arg tok "$tokens" --arg cost "$cost_usd" --arg model "$model" '
      (split("\n") | map(select(length > 0))) as $actors
      | {
          correlationId: $cid,
          actors: $actors
        }
      + (if $act != "" then {activity: $act} else {} end)
      + (
          ( {}
            + (if $tok  != "" then {tokens:  ($tok  | tonumber)} else {} end)
            + (if $cost != "" then {costUsd: ($cost | tonumber)} else {} end)
            + (if $model != "" then {model: $model} else {} end)
          ) as $dims
          | if ($dims | length) > 0 then {dimensions: $dims} else {} end
        )
      | [.]
    '
  )"
elif command -v python3 >/dev/null 2>&1; then
  payload="$(python3 - "$correlation_id" "$activity" "$tokens" "$cost_usd" "$model" "$actors" <<'PY'
import json, sys
cid, act, tok, cost, model, actors_raw = sys.argv[1:7]
event = {"correlationId": cid, "actors": [a for a in actors_raw.split("|") if a]}
if act:
    event["activity"] = act
dims = {}
if tok:
    dims["tokens"] = float(tok) if "." in tok else int(tok)
if cost:
    dims["costUsd"] = float(cost)
if model:
    dims["model"] = model
if dims:
    event["dimensions"] = dims
print(json.dumps([event]))
PY
  )"
else
  echo "ai-hub-push.sh needs jq or python3 to build the payload — push skipped" >&2
  exit 0
fi

curl -sf --max-time 5 -X POST \
  -H "Authorization: Bearer $api_key" \
  -H 'Content-Type: application/json' \
  --data "$payload" "$url" >/dev/null \
  || echo "ai-hub-push.sh: push to AI Hub failed or timed out — continuing" >&2
```

Then `chmod +x scripts/ai-hub-push.sh` and commit it — it contains no secret, only variable references. Design notes worth preserving if the script is edited:

- **The script does the JSON encoding, not the caller.** Same reasoning as `notify.sh` — a malformed payload fails on the server side, `curl -f` fails, and the caller learns nothing if it isn't reported. `jq`/`json.dumps` cannot get the escaping wrong.
- **`--max-time 5`** keeps the promise that pushing never blocks a delivery step.
- **Missing config or a failed send both exit 0**, but say so on stderr, so an ad-hoc "push an event" request has something truthful to report instead of claiming success.
- **Plain POSIX shell.** On Windows it needs Git Bash or WSL.
- **Actors are joined on `|`, not comma**, because an actor name (an email) can't contain `|` but a display name plausibly could contain a comma.

---

## Lifecycle events (the default push set)

The agent pushes one event at each of these AI-DLC moments. This set is the default; the project may add or remove events in the master rule file Section 11.

| Event | `correlationId` | `--actor` values | Suggested `--activity` |
|---|---|---|---|
| Unit marked Done | the unit's id | engineer, AI tool actor | `Build` |
| Bolt complete (all units Done) | the bolt's id | engineer, AI tool actor | `Build` |
| UAT sign-off recorded | the intent's id | engineer, AI tool actor | `UAT` |
| Intent moves to Implemented | the intent's id | engineer, AI tool actor | `Delivery` |

Omit `--activity` on any row where `AI_HUB_NODE_ACTIVITY_ID` is configured (activity-scoped endpoint) — the table's `--activity` column applies only to the node-scoped setup, where the project wants events split by activity on one node.

Sending is best-effort and must never interrupt the workflow: push the event, then continue the step. Do not wait for or report the push result unless the engineer asked for an ad-hoc send.

---

## Onboarding setup (run once, during installation)

Whoever is installing this skill — the onboarding agent as part of full framework setup, or the AI directly when the skill is adopted standalone via the skills catalogue — performs these steps when the engineer opts into AI Hub metrics:

**1. Ask whether the project wants AI Hub metrics.**

> "Do you want delivery events (units done, bolts complete, UAT sign-off, intents implemented) pushed to 99x AI Hub so the team can see AI-assisted work in one dashboard — tokens, cost, model, and who did it? This is a project-wide setup: one team, one workflow, one shared credential — not per engineer. (You can skip this and add it later.)"

If the engineer declines, write Section 11 with **`Status: Disabled`** and stop here.

**2. Close the leak paths first, before a credential exists.** Add `scripts/ai-hub.env`, `.envrc`, and `.claude/settings.local.json` to the project's `.gitignore` (create the file if there is none; these three may already be present from the notifications skill — do not duplicate). Confirm with `git check-ignore scripts/ai-hub.env`.

**3. Install the send script** at **`scripts/ai-hub-push.sh`** — repository root, not under `{FRAMEWORK_ROOT}`, so it is reachable at a stable relative path and shares no naming collision with `scripts/notify.sh`. Then `chmod +x scripts/ai-hub-push.sh` and commit it.

**4. Stop the send prompting for permission.** Add the allow rule alongside any existing ones (merge, never overwrite):

- **Claude Code** — `.claude/settings.json`:
  ```json
  { "permissions": { "allow": ["Bash(scripts/ai-hub-push.sh:*)"] } }
  ```
- **Cursor** — add `scripts/ai-hub-push.sh` to the agent's terminal command allowlist in Cursor Settings.
- **GitHub Copilot** — allow it in `chat.tools.terminal.autoApprove` (or equivalent) for Copilot agent mode.

**5. Hand the engineer Step 1 and Step 2 of *Who does what* above, and wait.** These are theirs, not yours: modeling the workflow and minting a credential is a browser flow under their AI Hub login. Never ask them to paste the API key into the conversation or into a committed file.

When they confirm, run *Step 3 — Verify* yourself to check the push reaches AI Hub — the send script exists by now, which is why this step comes after installing it.

**6. Populate the master rule file Section 11** (see the setup guide) with `Status: Enabled`, the node/activity id source (env var names, never the values themselves), and the event table.

**7. Confirm.** Send one test push (Step 3) and ask the engineer to confirm the event appears in the AI Hub workflow's event list, and that it was sent **without a permission prompt**. If it prompted, step 4 did not take effect.

**8. Show the status summary.** Once the test push is confirmed, present the *Status summary* card below to the engineer — this is their record of what was just turned on and how to control it. Do this whether the skill was installed by the full onboarding agent or adopted standalone via the skills catalogue; it is the one moment every setup path shares.

---

## Status summary

Show this card at the end of setup (Onboarding setup step 8), and any time the engineer asks "show AI Hub status", "is AI Hub metrics on", or similar — read Section 11 and the configured environment to fill it in:

> **AI Hub Metrics: Enabled**
> - Pushing events for: *[list the events currently in the Section 11 table, e.g. unit Done, bolt complete, UAT sign-off, intent implemented]*
> - Sending to: `<AI_HUB_BASE_URL>` — open it and go to **Workflows → [your workflow]** to watch events land
> - Credential: *[team API key / personal access token]*, read from the environment or `scripts/ai-hub.env` — never displayed here
> - Pause for just this session: *"turn off AI Hub metrics for this session"*
> - Turn off for the whole project: *"disable AI Hub metrics"*
> - Turn back on: *"enable AI Hub metrics"*

If Section 11 is `Status: Disabled` (or absent), show instead:

> **AI Hub Metrics: Disabled** — no events are being pushed to AI Hub. Say "enable AI Hub metrics" to turn it on.

Never include the API key/PAT value in this card, even partially — only that a credential is configured and which kind.

---

## Enabling and disabling

Three levels, lightest to heaviest — use the lightest one that satisfies what the engineer asked for:

1. **Silence for this session only** — *"turn off AI Hub metrics for this session."* Non-persistent: the agent just skips pushes until the session ends. Section 11 is untouched, so the next session pushes normally again without being asked.
2. **Project-wide switch** — *"enable AI Hub metrics"* / *"disable AI Hub metrics"* (also recognize "turn on/off AI Hub integration"). The agent edits Section 11's `Status` field in the master rule file directly, to `Enabled` or `Disabled`, and confirms the new state with the *Status summary* card. Because the master rule file is shared, this takes effect for every engineer's next session immediately — unlike the notifications skill's per-engineer toggle, there is nothing for teammates to individually opt into.
   - **Enabling when nothing is configured yet** (no node/activity id, no credential anywhere): do not just flip the status — that would silently no-op every push. Run the *Onboarding setup* steps above first, then flip Section 11 to `Enabled` as part of step 6.
   - **Enabling when configuration already exists** (Section 11 was previously `Enabled` and only got switched off, or `scripts/ai-hub.env`/the environment already has all required values): flipping `Status` to `Enabled` is enough — verify with one test push (*Step 3 — Verify*) before confirming with the status card.
   - **Disabling** never removes the script, the gitignore entries, or the credential — it only flips the switch. Re-enabling later needs no re-setup as long as the credential is still valid.
3. **Revoke the credential** — delete the API key/PAT in AI Hub (**Team → API keys**, or **Settings → Personal access tokens**). Every push then fails closed (silent no-op) regardless of what Section 11 says, until a new credential is configured. Use this — not the Section 11 switch — when the integration must stop working immediately and irreversibly from AI Hub's side: offboarding, or rotating a key that may have leaked.
