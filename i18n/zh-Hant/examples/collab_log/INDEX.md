# COLLAB_LOG — Claude ⇄ Codex 協作日誌（索引）— Acme Web

兩個 AI agent 在本專案上的交接日誌：**Claude**（Anthropic, Claude Code）與 **Codex**（OpenAI, Codex CLI）。
兩者不共享記憶，這個資料夾就是彼此同步的唯一管道。

每天一個檔案 `YYYY-MM-DD.md`，避免單一檔案無限膨脹。本檔（INDEX.md）只放**規則**、**「現在進行中」區塊**與**每日一句話索引**。

---

## 🔴 現在進行中（open threads）

> 這是本日誌**唯一可覆寫**的區塊：每次收工時直接改寫成最新狀態，**不保留歷史**。開工時讀這一塊就知道現況。

- ⏳ 結帳 API 的 rate-limit middleware 已實作，**待 Codex 獨立驗證**並發 PR（見今天日檔最上方 entry 的 handoff）。
- 🟢 登入頁改版已上線，無 open thread。

---

## 開工流程（每個 session，動程式前）
1. **讀本檔 `INDEX.md`** —— 規則 +「🔴 現在進行中」區塊 + 最近幾天的摘要。
2. **讀「今天」的日期檔** `collab_log/<今天日期>.md`；不存在就用範本新建。
3. **補寫昨天的總結（若還沒寫）**：用 `Grep pattern="⏳" path=collab_log/INDEX.md` 定位待補的日子，補完總結並把該列改 ✅。

## 收工流程（完成一個工作單元後）
1. 在**今天**的日期檔，`# Entries` 區、最新在上，append 一條 entry。
2. **覆寫上面的「🔴 現在進行中」區塊**。
3. 不刪歷史、不改別人的舊 entry。當 handoff 標「需驗證」時，接手方要實際跑過再勾。

---

## 每日索引（最新在上）

| 日期 | 一句話摘要 | 總結 |
|------|-----------|------|
| 2026-06-16 | 結帳 API 加上 rate-limit middleware；Codex review 抓到一個邊界 bug | ⏳ |
| 2026-06-15 | 登入頁改版、Codex 審英文文案 | ✅ |
