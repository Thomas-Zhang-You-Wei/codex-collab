# COLLAB_LOG — Claude ⇄ Codex collaboration journal (index) — {{PROJECT_NAME}}

Handoff journal for two AI agents on this project: **Claude** (Anthropic, Claude Code) and **Codex** (OpenAI, Codex CLI). They don't share memory; this folder is their only sync channel.

One file per day `YYYY-MM-DD.md`, to keep any single file from growing forever. This file (INDEX.md) holds only **rules**, the **"In progress" block**, and a **one-line-per-day index** — read here for a recap, click into a day file for detail.

---

## 🔴 In progress (open threads)

> This is the **only overwritable** block in the journal: overwrite it with the latest state at each finish ritual, **no history kept** (history lives in each day file's entries). Reading this block at the start tells you the current state without scanning entries.
>
> **What belongs here: open threads only** — unfinished work, pending decisions, known landmines. **Never** record state git already tracks (uncommitted files, WIP diffs): it goes stale the moment anyone commits. If this block and git disagree, git wins — fix the block.

- (no work in progress)

---

## Start ritual (every session, before touching code)
1. **Read this `INDEX.md`** — rules + "🔴 In progress" block + recent summaries.
2. **Reconcile the block with reality:** check each "In progress" claim against `git status --short` / recent `git log`; fix any claim that no longer holds before starting work.
3. **Read "today's" day file** `collab_log/<today>.md`; create it from the template below if missing (with an empty `## Daily summary` block).
4. **Backfill yesterday's summary (if not done):** use `Grep pattern="⏳" path=collab_log/INDEX.md` to locate the day whose summary column is still ⏳ (don't Read the whole file and scan by hand), read all its entries, write 3–6 lines in its `## Daily summary` (what got done overall, what's still open), and update that row below to ✅.

## Finish ritual (after completing a unit of work)
1. In **today's** day file, in the `# Entries` section, **newest on top**, prepend one entry (use the template below).
2. **Overwrite the "🔴 In progress" block above** to reflect the latest open threads (drop finished, add new). An entry without this block update is an unfinished ritual.
3. Never delete history, never edit someone else's old entry.

> Note: daily summaries are written the **next day** — the first session of the next day backfills the previous day's summary. So today's summary column stays ⏳ until tomorrow; that's normal.

### Entry template
```
## [YYYY-MM-DD HH:MM TZ] <Claude | Codex | Codex (via MCP)> — <one-line title>
- did: <what you did / decided>
- verify: <how you verified, or "not verified">
- files: <paths touched, or "review only — no edits">
- handoff: <what the other agent should know / verify / pick up>
```

### Day file template
```
# YYYY-MM-DD — Claude ⇄ Codex collaboration journal

> Rules and the daily index are in [`INDEX.md`](INDEX.md). This file holds only today's entries (newest on top).

## Daily summary
⏳ (backfilled at the next day's start ritual)

---

# Entries
```

---

## Daily index (newest on top)

| Date | One-line summary | Summary |
|------|------------------|---------|
