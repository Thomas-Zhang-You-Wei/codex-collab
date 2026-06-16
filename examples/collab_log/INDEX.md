# COLLAB_LOG — Claude ⇄ Codex collaboration journal (index) — Acme Web

Handoff journal for two AI agents on this project: **Claude** (Anthropic, Claude Code) and **Codex** (OpenAI, Codex CLI). They don't share memory; this folder is their only sync channel.

One file per day `YYYY-MM-DD.md`. This file (INDEX.md) holds only **rules**, the **"In progress" block**, and a **one-line-per-day index**.

---

## 🔴 In progress (open threads)

> The **only overwritable** block: overwrite with the latest state at each finish ritual, **no history kept**. Read this at the start to know the current state.

- ⏳ Rate-limit middleware for the checkout API is implemented, **awaiting independent verification by Codex** and a PR (see the handoff in the top entry of today's day file).
- 🟢 Login page redesign shipped, no open thread.

---

## Start ritual (every session, before touching code)
1. **Read this `INDEX.md`** — rules + "🔴 In progress" block + recent summaries.
2. **Read today's day file** `collab_log/<today>.md`; create from the template if missing.
3. **Backfill yesterday's summary (if not done):** use `Grep pattern="⏳" path=collab_log/INDEX.md` to locate the day to backfill, write the summary, and flip that row to ✅.

## Finish ritual (after completing a unit of work)
1. In **today's** day file, `# Entries` section, newest on top, prepend one entry.
2. **Overwrite the "🔴 In progress" block above.**
3. Never delete history, never edit someone else's old entry. When a handoff is marked "needs verification", the receiver must actually run it before ticking it off.

---

## Daily index (newest on top)

| Date | One-line summary | Summary |
|------|------------------|---------|
| 2026-06-16 | Added rate-limit middleware to checkout API; Codex review caught a boundary bug | ⏳ |
| 2026-06-15 | Login page redesign; Codex reviewed English copy | ✅ |
