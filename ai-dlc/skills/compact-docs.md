# Skill: Compact Docs

**Purpose:** Archive operational documents older than the project's configured threshold to keep the active workspace manageable without losing institutional memory.

**Trigger:** Engineer-initiated only. The engineer invokes this skill by saying something like: "run compact-docs", "archive old operational documents", or "compact the ops folder."

**Never run automatically.** This skill must not be triggered by any other workflow step.

---

## Step 1 — Determine the Archive Threshold

Read the **Archive threshold** value from the Process Configuration section of the master rule file.

- If the value is set, confirm it with the engineer before proceeding:
  > "The archive threshold is set to [X] months. Documents last modified before [calculated date] will be eligible. Shall I continue?"
- If the value is not set, ask the engineer:
  > "No archive threshold is configured. How many months old must a document be before it qualifies for archiving?"
  Record the answer and ask if it should be saved to the master rule file for future use.

---

## Step 2 — Scan for Qualifying Documents

Scan the following folders. A document qualifies for archiving if its last-modified date is older than the threshold **and** its status permits archiving:

| Folder | Eligible statuses |
|---|---|
| `ai-dlc/ops/inception/elaborations/` | Any (elaboration sessions are records, not live documents) |
| `ai-dlc/ops/build/units/` | Done, Cancelled |
| `ai-dlc/ops/build/bolts/` | Done, Cancelled |
| `ai-dlc/ops/operate/retros/` | Any |
| `ai-dlc/ops/operate/incidents/` | Resolved |
| `ai-dlc/ops/operate/improvements/` | Applied, Rejected |

**Never archive:**
- Intent files (`ai-dlc/ops/inception/intents/`) — these hold the Implementation Summary and are permanent references
- Any document with status Open, In Progress, Planned, or Blocked
- Template files (`_template.md`)
- Index and README files (`backlog.md`, `README.md`, `digest.md`)

---

## Step 3 — Present the Archive Preview

Before moving any file, present a table of everything that qualifies:

```
Documents to be archived (older than [X] months):

Folder                              | File                        | Last modified | Status
ai-dlc/ops/build/bolts/             | 2024-11-10-auth-bolt.md     | 2024-11-30    | Done
ai-dlc/ops/operate/retros/          | 2024-11-30-bolt-1-retro.md  | 2024-11-30    | —
...

[N] documents will be archived. [M] documents will remain active.

Proceed? (yes / no / show me what stays)
```

Do not move any file until the engineer confirms.

---

## Step 4 — Archive Documents

For each qualifying folder that has documents to archive:

1. Create `archive/` subfolder inside the folder if it does not already exist.
2. Move each qualifying document into `archive/`.
3. Append a one-line entry per document to `archive/digest.md` in that folder, creating the file if it does not exist. Entry format:

```markdown
| YYYY-MM-DD | [filename](filename) | [one-sentence summary of the document's key outcome or finding] |
```

The digest must contain enough information for a future engineer to know whether to retrieve a document without having to open it.

---

## Step 5 — Update the Backlog

`backlog.md` uses Markdown reference-style links defined in a **Reference Link Registry** section at the bottom of the file. All unit and bolt links in the table body are written as `[Unit-name][unit-name-ref]` rather than inline URLs. This means updating an archived item requires changing exactly one line in the registry — the path update — without touching the table matrix at all.

For each archived unit or bolt:

1. Locate its entry in the Reference Link Registry at the bottom of `backlog.md`. It will look like:
   ```
   [unit-name-ref]: ai-dlc/ops/build/units/filename.md
   ```
2. Update the path to point into `archive/`:
   ```
   [unit-name-ref]: ai-dlc/ops/build/units/archive/filename.md
   ```
3. Append ` <!-- archived -->` as an inline comment on the same line so archived entries are scannable without following the link:
   ```
   [unit-name-ref]: ai-dlc/ops/build/units/archive/filename.md <!-- archived -->
   ```

Do not touch the table rows. Do not rewrite inline links. If a backlog entry does not use a reference-style link (i.e., it was added manually with an inline URL), flag it to the engineer rather than attempting to rewrite it — inconsistent link styles should be corrected by the engineer, not silently patched by this skill.

---

## Step 6 — Report

Present a completion summary:

```
Compact-docs complete.

Archived:
  - [N] elaboration sessions  →  ai-dlc/ops/inception/elaborations/archive/
  - [N] units                 →  ai-dlc/ops/build/units/archive/
  - [N] bolts                 →  ai-dlc/ops/build/bolts/archive/
  - [N] retros                →  ai-dlc/ops/operate/retros/archive/
  - [N] incidents             →  ai-dlc/ops/operate/incidents/archive/
  - [N] improvements          →  ai-dlc/ops/operate/improvements/archive/

Digest files updated: [list of digest.md paths]
Backlog links updated: [yes / no changes needed]

To retrieve an archived document, look in the relevant archive/ subfolder or read the digest.md file there.
```
