# Migrating from the Old Structure

← [Back to README](../README.md)

---

## Background

Projects that adopted AI-DLC before the June 2026 restructuring used an `ai-dlc/` folder that served as both the bootstrap agent and the installed framework, with the diagnostic agent in a separate `ai-dlc-reviewer/` folder. The new methodology separates the bootstrap agents from the installed framework and introduces the `process-onboarding-agent/` / `process-diagnostic-agent/` naming.

**Old structure:**
```
<project>/
  ai-dlc/              ← bootstrap AND installed framework (mixed together)
  ai-dlc-reviewer/     ← diagnostic agent
```

**New structure:**
```
<project>/
  process-onboarding-agent/    ← bootstrap only (deleted after onboarding)
  process-diagnostic-agent/    ← diagnostic agent
  {docs}/
    intent-execution-framework/  ← the installed framework lives here
```

---

## What Gets Migrated

The migration agent handles all three agents in one session:

- **Onboarding agent** — `ai-dlc/` → `process-onboarding-agent/` (bootstrap only); all generated framework files moved to `intent-execution-framework/`
- **Experience agent** — master rule file path references updated from `ai-dlc/...` to the new `intent-execution-framework/` location
- **Diagnostic agent** — `ai-dlc-reviewer/` → `process-diagnostic-agent/` (with Trend Analysis capability added)

All operational data (intents, units, bolts, retros) is preserved throughout the migration.

---

## How to Use

1. Copy all three folders from `repository-agents/` in this repo into the root of the project being migrated:
   - `process-migration-agent/`
   - `process-onboarding-agent/`
   - `process-diagnostic-agent/`

2. Open your AI assistant inside the project repo.

3. Say:
   > "Read `process-migration-agent/migrate.md` and follow the instructions inside it."

4. The agent scans the old structure, presents a pre-flight summary (file counts, what will move, what will be replaced), confirms the migration plan with you, moves all files, updates the master rule file path references, and delivers a migration report.

---

## Migration Agent Files

| File | Purpose |
|---|---|
| `repository-agents/process-migration-agent/migrate.md` | Bootstrap trigger. Copy into the target repo and read it to the AI to start the migration. |
| `repository-agents/process-migration-agent/migration-guide.md` | Full migration protocol — pre-flight scan, file classification, relocation steps, master rule file path updates, and cleanup. |
