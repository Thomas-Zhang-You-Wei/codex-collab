<!-- Paste this whole block at the top of your project's CLAUDE.md (bootstrap). This is the entry guide for Claude. -->

## Always Do First — Claude ⇄ Codex collaboration journal

This project is co-maintained by two agents, **Claude** and **Codex**. They don't share memory; their only shared memory is `collab_log/`. Before touching code and after finishing, always:

1. **Start:** read `collab_log/INDEX.md` — rules + the "🔴 In progress" block + recent summaries; then read today's day file `collab_log/<today>.md` (create from template if missing).
2. **Backfill yesterday's summary:** use `Grep pattern="⏳" path=collab_log/INDEX.md` to find any past day whose summary column is still ⏳; after reading that day's entries, write its `## Daily summary` (3–6 lines) and flip that INDEX row to ✅.
3. **Finish** (after a unit of work): prepend one entry (did / verify / files / handoff) to the **top** of the `# Entries` section in today's file; and **overwrite** the "🔴 In progress" block in INDEX to reflect the latest state.
4. Never delete history, never edit someone else's old entry. When a handoff is marked "needs verification", the receiver must actually run it before ticking it off.
