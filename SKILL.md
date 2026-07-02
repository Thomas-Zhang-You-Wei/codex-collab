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
2. **Reconcile the block with reality**: check each "In progress" claim against `git status --short` and recent `git log`; if a claim no longer holds (e.g. it says "X is uncommitted" but the tree is clean), fix the block right now. This is part of the start ritual, not optional cleanup — a stale block sends the next agent down a false trail.
3. Read today's day file; create it from the template if it doesn't exist.
4. Backfill yesterday's summary: use `Grep pattern="⏳" path=collab_log/INDEX.md` to locate the day whose summary column is still ⏳ (don't Read the whole file and scan by hand), read all its entries, write 3–6 lines in its `## Daily summary`, and update that row in INDEX.md to ✅.

## Step 3 — Finish ritual (after completing a unit of work)

1. Prepend an entry to the **top** of the `# Entries` section in today's file (newest first):

```
## [YYYY-MM-DD HH:MM TZ] <Claude | Codex | Codex (via MCP)> — <one-line title>
- did: <what you did / decided, including key trade-offs>
- verify: <how you verified (build/test/screenshot), or "not verified">
- files: <paths touched, or "review only — no edits">
- handoff: <what the other agent should know / verify / pick up>
```

2. **Overwrite the "🔴 In progress" block in INDEX**: drop finished threads, add new ones. History goes in the day file; current state goes in this block. **The ritual is incomplete until this block is updated** — an entry without the block update is the most common way the two agents desync.
3. If the project uses git, commit the log change together with the work; an uncommitted log is invisible to anyone pulling the repo elsewhere.

Rules: never delete history, never edit someone else's old entry.

## Step 4 — How to put Codex to work

- Use `mcp__codex__codex` for a new task, `mcp__codex__codex-reply` for follow-ups (keeps its conversation context).
- **Feed distilled context; don't make it read the whole log.** MCP calls are stateless, and Codex's ChatGPT account has a rolling-window rate limit — re-reading the entire `collab_log/` each time is slow and burns quota. Inline the one or two relevant entries + the "In progress" block in the prompt; only tell it to read `AGENTS.md` and `collab_log/` when it genuinely needs the full picture.
- The prompt **must** include: the project absolute path (cwd), the concrete task, and acceptance criteria.
- **For review / second opinion, make it actually diverge:** do **not** give Codex your reasoning or preferred answer first; ask it to assess independently and explicitly list risks and "what it would do differently", then you reconcile. Stating your conclusion first biases it toward agreement and wastes the second opinion.
- **Who writes the entry:** when Codex is called by you via MCP, **you (Claude) write its entry on its behalf**, author field `Codex (via MCP)`; only when Codex works standalone in its own CLI does it write its own. This avoids missing or duplicated entries.
- Typical delegations: review the diff you just wrote, propose an alternative to the same problem, review English copy, independently reproduce a bug.
- Codex's reply is only visible to you — **relay the conclusion to the user**, and write important conclusions into the log.

## Step 4.5 — Codex's sandbox limits: network / package installs (read before delegating)

The biggest trap in "Claude as brain, Codex as hands": **Codex's sandbox has no network access.**

- The MCP default `sandbox=workspace-write` **blocks outbound network**. Reading files and editing existing code work fine, but **any step that needs the network fails** — most commonly installing a new package (`ENOTFOUND registry.npmjs.org`), also fetching remote resources or calling external APIs.
- Worse: under the default `approval-policy=on-failure`, Codex **cannot escalate its own permissions** to retry (it gets *"cannot ask for escalated permissions if the approval policy is OnFailure"*). So it spins between "can't install, can't escalate" **until the user aborts manually**. It looks like Codex "refusing" or "hanging", but it's an environment deadlock — **not rate limiting, not the model giving up**.
- **Division rule: Claude (the brain) handles anything that needs the network first, then hands the pure code changes to Codex (the hands).** Pick one:
  1. **Claude installs first** (recommended): before delegating a task that needs a new dependency, run `npm install <pkg>` locally (Claude has no network restriction). With deps in place, Codex can finish the code changes + type-check in its offline sandbox.
  2. **Grant Codex network**: delegate with `sandbox=danger-full-access` + `approval-policy=never` so it can reach the registry itself. ⚠️ Full access = no guardrails; only for trusted local projects.
- **Ask one question before delegating**: does this task install packages / fetch remote resources / call APIs? If yes, handle that first — don't let Codex hit the wall.
- **Diagnosing a stuck Codex**: read `~/.codex/sessions/<yyyy>/<mm>/<dd>/rollout-*.jsonl` — ending at `role=user` / `turn_aborted` means stuck; for long-running goals check `thread_goals` in `~/.codex/goals_1.sqlite`.

## Red lines

- **Before delegating a task that installs packages / needs the network, either install as Claude first or use `danger-full-access`** — don't drop network steps into the default sandbox, or Codex spins until aborted (see Step 4.5).
- Never skip the start ritual ("read the log") or the finish ritual ("write the entry + overwrite In progress") — this is the only shared memory between the two agents.
- **The "In progress" block holds open threads only — never workspace state that git already records** (uncommitted files, WIP diffs). Such claims go stale the moment anyone commits; git is the authority. On a mismatch: trust git, fix the block (see the start ritual).
- INDEX.md holds only rules, the "In progress" block, and one-line index entries; all detail goes in day files; keep any single file from bloating.
- **When a handoff is marked "needs verification", the receiving agent must actually run it (build / test / reproduce) and record the result in a new entry's `verify`** — don't tick it off on the other's word that "it's fixed". Independent verification is where two agents beat one.
- When both agents touched the same thing, the entry must spell out who changed which part, to avoid overwriting each other.

## Maintaining this skill

Source of truth is the `codex-collab` GitHub repo (EN at the root, zh-Hant under `i18n/`). To change the skill: edit the repo first (both languages together), then copy the zh-Hant `SKILL.md` + `templates/` into `~/.claude/skills/codex-collab/`. Never patch the installed copy alone — an installed-only edit (the sandbox-deadlock section, 2026-06-28) once drifted from the repo unnoticed for ten days.
