---
name: codex-collab
description: Start Claude ⇄ Codex two-agent collaboration mode — set up a shared handoff journal (collab_log/) in any project, run start/finish rituals, and divide work via the Codex MCP. Use when the user says "start codex collab", "collaborate with codex", "start collaboration mode", "set up a collab log", "write the collaboration log", asks for a Codex second opinion / review, or the Chinese equivalents (啟動 codex 協作 / 跟 codex 合作 / 寫協作日誌).
---

# Codex Collab — Claude ⇄ Codex collaboration mode

Two AI agents co-maintain one project: **Claude** (Anthropic, Claude Code) and **Codex** (OpenAI, Codex CLI). They do **not** share memory; their only sync channel is the project-root `collab_log/` folder — a daily handoff journal, one file per day.

## Default division of labor

- **Claude (lead):** planning, implementation, running the server, screenshot verification, writing the log.
- **Codex (second opinion):** code review, alternative approaches, bug hunting, copy/layout review, independent verification / sign-off.
- The split is adjustable per project, but "whoever touched what gets written to the log" is non-negotiable.

## Step 0 — Check the Codex MCP

Confirm the `mcp__codex__codex` / `mcp__codex__codex-reply` tools exist (load their schema via ToolSearch if needed). If missing, ask the user to run once (user scope, applies to all projects):

```bash
claude mcp add --scope user codex -- codex mcp-server
```

Codex signs in with a ChatGPT account (no API key, no extra cost — works on the free tier too, just with a rolling-window rate limit).

## Step 1 — Bootstrap (first time collaboration starts in a project)

Only when the project root has **no** `collab_log/` yet:

1. Create `collab_log/INDEX.md`: copy [templates/INDEX.template.md](templates/INDEX.template.md) and replace `{{PROJECT_NAME}}` with the project name.
2. Create today's day file `collab_log/YYYY-MM-DD.md` from [templates/DAY.template.md](templates/DAY.template.md).
3. Add today's row to the index table in INDEX.md (summary "collaboration mode started", summary column ⏳).
4. **Route both agents to the journal** (use the ready-made snippets, don't improvise — keeps both sides consistent):
   - Paste the contents of [templates/CLAUDE-snippet.md](templates/CLAUDE-snippet.md) at the top of the project `CLAUDE.md` (create it if missing).
   - Paste the contents of [templates/AGENTS-snippet.md](templates/AGENTS-snippet.md) at the top of the project `AGENTS.md` (Codex's default read point; create it if missing).
5. Write the first entry in today's file (did: set up the collaboration mechanism).

## Step 2 — Start ritual (every session, before touching code)

1. Read `collab_log/INDEX.md` (rules + the "🔴 In progress" block + recent summaries). Reading that block tells you the current state — no need to scan entries to reconstruct it.
2. Read today's day file; create it from the template if it doesn't exist.
3. Backfill yesterday's summary: use `Grep pattern="⏳" path=collab_log/INDEX.md` to locate the day whose summary column is still ⏳ (don't Read the whole file and scan by hand), read all its entries, write 3–6 lines in its `## Daily summary`, and update that row in INDEX.md to ✅.

## Step 3 — Finish ritual (after completing a unit of work)

1. Prepend an entry to the **top** of the `# Entries` section in today's file (newest first):

```
## [YYYY-MM-DD HH:MM TZ] <Claude | Codex | Codex (via MCP)> — <one-line title>
- did: <what you did / decided, including key trade-offs>
- verify: <how you verified (build/test/screenshot), or "not verified">
- files: <paths touched, or "review only — no edits">
- handoff: <what the other agent should know / verify / pick up>
```

2. **Overwrite the "🔴 In progress" block in INDEX**: drop finished threads, add new ones. History goes in the day file; current state goes in this block.

Rules: never delete history, never edit someone else's old entry.

## Step 4 — How to put Codex to work

- Use `mcp__codex__codex` for a new task, `mcp__codex__codex-reply` for follow-ups (keeps its conversation context).
- **Feed distilled context; don't make it read the whole log.** MCP calls are stateless, and Codex's ChatGPT account has a rolling-window rate limit — re-reading the entire `collab_log/` each time is slow and burns quota. Inline the one or two relevant entries + the "In progress" block in the prompt; only tell it to read `AGENTS.md` and `collab_log/` when it genuinely needs the full picture.
- The prompt **must** include: the project absolute path (cwd), the concrete task, and acceptance criteria.
- **For review / second opinion, make it actually diverge:** do **not** give Codex your reasoning or preferred answer first; ask it to assess independently and explicitly list risks and "what it would do differently", then you reconcile. Stating your conclusion first biases it toward agreement and wastes the second opinion.
- **Who writes the entry:** when Codex is called by you via MCP, **you (Claude) write its entry on its behalf**, author field `Codex (via MCP)`; only when Codex works standalone in its own CLI does it write its own. This avoids missing or duplicated entries.
- Typical delegations: review the diff you just wrote, propose an alternative to the same problem, review English copy, independently reproduce a bug.
- Codex's reply is only visible to you — **relay the conclusion to the user**, and write important conclusions into the log.

## Red lines

- Never skip the start ritual ("read the log") or the finish ritual ("write the entry + overwrite In progress") — this is the only shared memory between the two agents.
- INDEX.md holds only rules, the "In progress" block, and one-line index entries; all detail goes in day files; keep any single file from bloating.
- **When a handoff is marked "needs verification", the receiving agent must actually run it (build / test / reproduce) and record the result in a new entry's `verify`** — don't tick it off on the other's word that "it's fixed". Independent verification is where two agents beat one.
- When both agents touched the same thing, the entry must spell out who changed which part, to avoid overwriting each other.
