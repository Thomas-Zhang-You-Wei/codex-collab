<!-- Paste this whole block at the top of your project's AGENTS.md (bootstrap). This is the entry guide for Codex. -->

## Always Do First — Claude ⇄ Codex collaboration journal

You (**Codex**) and **Claude** co-maintain this project. You don't share memory; your only shared memory is `collab_log/`. Before touching code and after finishing, always:

1. **Start:** read `collab_log/INDEX.md` — rules + the "🔴 In progress" block + recent summaries; reconcile each "In progress" claim against `git status --short` / recent `git log` and fix stale claims on the spot; then read today's day file `collab_log/<today>.md` (create from template if missing).
2. **Backfill yesterday's summary:** grep `collab_log/INDEX.md` for any past day whose summary column is still ⏳; after reading that day's entries, write its `## Daily summary` (3–6 lines) and flip that INDEX row to ✅.
3. **Finish** (after a unit of work): prepend one entry (did / verify / files / handoff) to the **top** of the `# Entries` section in today's file; and **overwrite** the "🔴 In progress" block in INDEX to reflect the latest state (open threads only — never workspace state git already records). An entry without the block update is an unfinished ritual.
4. Never delete history, never edit someone else's old entry. When a handoff is marked "needs verification", you must actually run it before ticking it off.
5. **Note:** if you're invoked by Claude via MCP (rather than working standalone in your own CLI), Claude writes your entry for you (tagged `Codex (via MCP)`); you just need to state your conclusion clearly.
