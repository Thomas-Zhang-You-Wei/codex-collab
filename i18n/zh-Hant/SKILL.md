---
name: codex-collab
description: 啟動 Claude ⇄ Codex 雙 agent 協作模式：在任何專案建立共享交接日誌（collab_log/）、執行開工/收工儀式、透過 Codex MCP 分工合作。當使用者說「啟動 codex 協作」「跟 codex 合作」「啟動協作模式」「建立 collab log」「寫協作日誌」或要求 Codex 第二意見/審稿時使用。
---

# Codex Collab — Claude ⇄ Codex 協作模式

兩個 AI agent 共同維護一個專案：**Claude**（Anthropic, Claude Code）與 **Codex**（OpenAI, Codex CLI）。
兩者**不共享記憶**，唯一的同步管道是專案根目錄的 `collab_log/` 資料夾（每日一檔的交接日誌）。

## 預設分工

- **Claude（主導）**：規劃、實作、跑 server、截圖驗證、寫日誌。
- **Codex（第二意見）**：code review、替代方案、抓 bug、文案/排版審稿、雙簽核可。
- 分工可依專案調整，但「誰動了什麼都要寫進日誌」不可省。

## Step 0 — 檢查 Codex MCP

確認 `mcp__codex__codex` / `mcp__codex__codex-reply` 工具存在（必要時用 ToolSearch 載入 schema）。
若不存在，請使用者執行一次（user scope，全專案有效）：

```bash
claude mcp add --scope user codex -- codex mcp-server
```

Codex 用 ChatGPT 帳號登入（無 API key、無額外費用，rolling window 限流）。

## Step 1 — Bootstrap（專案第一次啟動協作時）

只在專案根目錄**沒有** `collab_log/` 時做：

1. 建立 `collab_log/INDEX.md`：複製本 skill 的 [templates/INDEX.template.md](templates/INDEX.template.md)，把 `{{PROJECT_NAME}}` 換成專案名。
2. 建立今天的日檔 `collab_log/YYYY-MM-DD.md`：用 [templates/DAY.template.md](templates/DAY.template.md)。
3. 在 INDEX.md 的索引表加上今天那一行（摘要先寫「啟動協作機制」，總結欄 ⏳）。
4. **讓兩個 agent 都會被導到日誌**（用現成範本，不要即興寫，確保兩邊措辭一致）：
   - 專案 `CLAUDE.md`（沒有就建）的最上方貼上 [templates/CLAUDE-snippet.md](templates/CLAUDE-snippet.md) 的內容。
   - 專案 `AGENTS.md`（Codex 的預設讀取點，沒有就建）的最上方貼上 [templates/AGENTS-snippet.md](templates/AGENTS-snippet.md) 的內容。
5. 寫完後在今天日檔記第一條 entry（did: 建立協作機制）。

## Step 2 — 開工流程（每個 session，動程式前）

1. 讀 `collab_log/INDEX.md`（規則 +「🔴 現在進行中」區塊 + 最近幾天摘要）。讀這個區塊就知道現況，不必掃 entry 重建。
2. **核對區塊與現實**：把「現在進行中」的每一條對照 `git status --short` 與近期 `git log`；只要有一條不再成立（例如寫著「X 未提交」但工作區是乾淨的），當場修正區塊。這是開工儀式的一部分，不是可做可不做的清理——過期的區塊會把下一個 agent 帶進錯誤的方向。
3. 讀今天的日檔；不存在就從範本新建。
4. 補寫前一天的總結：用 `Grep pattern="⏳" path=collab_log/INDEX.md` 定位總結欄還是 ⏳ 的日子（不要 Read 整檔人工掃），讀完該檔所有 entry，在其 `## 當日總結` 寫 3–6 行（整體做了什麼、還開著什麼），並更新 INDEX.md 該列為 ✅。

## Step 3 — 收工流程（完成一個工作單元後）

1. 在今天日檔的 `# Entries` 區**最上方**（最新在上）append 一條：

```
## [YYYY-MM-DD HH:MM TZ] <Claude | Codex | Codex (via MCP)> — <一句話標題>
- did: <做了什麼 / 決定了什麼，含關鍵取捨>
- verify: <怎麼驗證的（build/測試/截圖），或 "not verified">
- files: <動到的路徑，或 "review only — no edits">
- handoff: <對方該知道 / 該驗證 / 該接手的事>
```

2. **覆寫 INDEX 的「🔴 現在進行中」區塊**：做完的 thread 拿掉、新開的加上。歷史進日檔，現況進這塊。**沒更新這個區塊就不算收工**——只寫 entry 不改區塊，是兩個 agent 失去同步最常見的原因。
3. 專案有 git 的話，日誌變更跟工作一起 commit；沒 commit 的日誌，對在其他地方 pull 這個 repo 的人是看不見的。

規則：不刪歷史、不改別人的舊 entry。

## Step 4 — 怎麼叫 Codex 工作

- 新任務用 `mcp__codex__codex`，後續追問用 `mcp__codex__codex-reply`（保留它的對話 context）。
- **精餵 context，別叫它讀整本 log**：MCP 呼叫是無狀態的，且 Codex 用 ChatGPT 帳號有 rolling window 限流，每次重讀整個 `collab_log/` 又慢又燒額度。prompt 裡直接**內嵌**「這次任務相關的那一兩條 entry +『現在進行中』區塊」；只有真的需要全貌時才叫它讀 `AGENTS.md` 與 `collab_log/`。
- prompt **一定要**給：專案絕對路徑（cwd）、具體任務、驗收標準。
- **要 review / 第二意見時，讓它真的發散**：先**不要**給 Codex 你的理由或你偏好的答案，要它獨立評估、明確列出風險與「它會怎麼做不一樣」，再由你調和兩邊。先講你的結論會把它帶往附和，第二意見就白找了。
- **誰寫 entry**：Codex 經 MCP 被你呼叫時，由**你（Claude）代寫**它的 entry，作者欄標 `Codex (via MCP)`；只有 Codex 在自己 CLI 獨立工作時才由它自己寫。這樣不會漏寫或兩邊重複寫。
- 典型委派：review 剛寫好的 diff、對同一問題出替代方案、英文文案審稿、獨立重現 bug。
- Codex 的回覆只有你看得到——把結論**轉述給使用者**，重要結論也寫進日誌。

## Step 4.5 — Codex 的沙盒限制：連網 / 裝套件（委派前必看）

「Claude 當大腦、Codex 當手」會踩到的最大坑：**Codex 的沙盒不能連網**。

- MCP 預設 `sandbox=workspace-write` **封鎖對外網路**。純讀檔、改既有程式碼都沒問題，但**任何要連網的步驟都會失敗**——最常見是 `npm install` 新套件（`ENOTFOUND registry.npmjs.org`），也包括抓遠端資源、打外部 API。
- 更糟：預設 `approval-policy=on-failure` 下 Codex **不能自己升權**重跑（會被回 *"cannot ask for escalated permissions if the approval policy is OnFailure"*）。於是它在「裝不了又升不了權」之間**空轉到使用者手動中斷**。表面像 Codex「拒絕」或「當機」，其實是環境死結——**不是限流、也不是它罷工**。
- **分工原則：連網的事由 Claude（大腦）先處理好，再把純改碼交給 Codex（手）。** 兩種做法擇一：
  1. **Claude 先裝**（推薦）：委派需要新依賴的任務前，先在本機 `npm install <pkg>`（Claude 不受網路限制）。裝好後 Codex 在離線沙盒就能完成程式碼改動 + type-check。
  2. **給 Codex 連網權**：委派時 `mcp__codex__codex` 帶 `sandbox=danger-full-access` + `approval-policy=never`，它就能自己連 registry。⚠️ 全權限＝不設防，只在信任的本機專案用。
- **委派前自問一句**：這個任務有沒有「裝新套件 / 抓遠端資源 / 打 API」？有 → 先照上面處理，別讓 Codex 撞牆。
- **診斷卡住的 Codex**：讀 `~/.codex/sessions/<年>/<月>/<日>/rollout-*.jsonl`，結尾停在 `role=user` / `turn_aborted` ＝卡住；長時 goal 狀態看 `~/.codex/goals_1.sqlite` 的 `thread_goals`。

## 紅線

- **委派「要裝新套件 / 連網」的任務前，先由 Claude 裝好、或開 `danger-full-access`**；別把連網步驟丟進預設沙盒，否則 Codex 會空轉到被中斷（見 Step 4.5）。

- 永遠不要跳過開工的「讀日誌」與收工的「寫 entry + 覆寫現在進行中」——這是兩個 agent 唯一的共同記憶。
- **「現在進行中」只放 open threads——不放 git 已經記錄的工作區狀態**（未提交檔案、WIP diff）。這種描述在任何人 commit 的瞬間就過期；git 才是權威。發現不一致：以 git 為準、修正區塊（見開工儀式）。
- INDEX.md 只放規則、「現在進行中」區塊和一句話索引，細節全進日檔；單檔不膨脹。
- **handoff 標「需驗證」時，接手方要實際跑過（build / 測試 / 重現）再在新 entry 的 `verify` 記結果**，不能憑對方「我修好了」就打勾。獨立驗證才是雙 agent 比單 agent 強的地方。
- 同一件事兩個 agent 都動過時，entry 要寫清楚誰改了哪部分，避免互相覆蓋。

## 維護這個 skill

事實來源是 GitHub repo `codex-collab`（根目錄英文版、`i18n/` 中文版）。要改 skill：先改 repo（兩個語言一起改），再把 zh-Hant 的 `SKILL.md` + `templates/` 複製到 `~/.claude/skills/codex-collab/`。**絕不要只改安裝版**——2026-06-28 的沙盒死結段落就是只改了安裝版，跟 repo 漂移了十天才被意外發現。
