<!-- 把以下整段貼到專案 CLAUDE.md 的最上方（bootstrap 用）。這是給 Claude 看的入口指引。 -->

## Always Do First — Claude ⇄ Codex 協作日誌

本專案由 **Claude** 與 **Codex** 兩個 agent 協作，兩者不共享記憶，唯一的共同記憶是 `collab_log/`。動程式前與完工後務必：

1. **開工**：先讀 `collab_log/INDEX.md` —— 規則 +「## 🔴 現在進行中」區塊 + 最近幾天摘要；再讀今天的日檔 `collab_log/<今天日期>.md`（不存在就用範本新建）。
2. **回填昨天總結**：用 `Grep pattern="⏳" path=collab_log/INDEX.md` 找出總結欄還是 ⏳ 的舊日子；讀完該日所有 entry 後補寫其 `## 當日總結`（3–6 行），並把 INDEX 索引表該列改成 ✅。
3. **收工**（完成一個工作單元後）：在今天日檔 `# Entries` 區**最上方** append 一條 entry（did / verify / files / handoff）；同時**覆寫** INDEX 的「## 🔴 現在進行中」區塊，反映最新狀態。
4. 不刪歷史、不改別人的舊 entry。當 handoff 標「需驗證」時，接手方要實際跑過再勾。
