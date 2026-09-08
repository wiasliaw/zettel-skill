---
name: adapt
description: 與使用者逐條裁定 .zettel/proposals/ 的提案佇列：駁回（刪除即封存）或晉升為 convention。晉升的 convention 影響後續所有寫入型 zk 指令。
disable-model-invocation: true
---

# /zk:adapt

本流程是**對話流程**：逐條停下等待使用者裁定，MUST NOT 批次帶過、MUST NOT 代為決定。寫入範圍恰為 `.zettel/conventions/` 與 `.zettel/proposals/`；不觸碰筆記、config、日誌與索引；不修改 plugin 檔案——plugin 的變更不在 runtime 職權，只發生於 plugin repo 的開發流程。

## 流程

1. **前置**：確認 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`。依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 `.zettel/conventions/` 全部檔案（本流程需要知道既有主題檔，晉升時才能判定「建立或就地更新」）；全程有效：工作中發現使用者慣例訊號時，依同檔「proposal 寫入操作」寫入 proposal 檔。

2. **讀 proposals 並逐條討論**：讀 `.zettel/proposals/` 下每一份提案檔全文，逐檔向使用者呈現並討論，絕不略過任一份未經呈現的提案；佇列為空時直接回報「無待議提案」收尾。提案的 User review 節已有內容時一併呈現，作為該案裁定的直接依據。

3. **逐條裁定：二擇一**（與討論交錯進行，非全部討論完才裁定）：

   **套用前重讀**：每次建立／更新 convention 或刪除 proposal 前，MUST 重讀該 proposal 全文（含 User review），重新列出 conventions 並讀取對應主題檔全文；預定新建的主題也須重查是否已被建立。以最後一次呈現並獲裁定的內容為比對基準。若提案、User review 或相關 convention 有變動，MUST 先呈現差異並重新討論裁定，MUST NOT 沿用過期裁定寫入或刪除；提案已不存在時回報並跳過，不依舊副本晉升。無變動才以最小 diff 套用，保留所有無關內容，不回滾併發修改。

   - **駁回**：

     ```bash
     rm .zettel/proposals/<檔名>
     ```

     駁回即刪除檔案，刪除即封存，不另設 archive 目錄。

   - **晉升為 convention**：依 `${CLAUDE_PLUGIN_ROOT}/templates/convention.md` template，在 `.zettel/conventions/` 建立或就地更新對應主題檔——一主題一檔，MUST NOT 為同一主題另建副本（既有主題檔存在時就地更新它）。frontmatter 恰含 `when`（一句話觸發條件，綁可觀察訊號）。格式範例見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md`。寫入成功後，刪除提案前 MUST 再次執行上述重讀：proposal 與裁定基準一致，且 convention 與剛寫入結果一致，才刪除該提案。若任一方改變，保留提案、呈現差異並重新裁定；已寫入的 convention 不自動回滾。寫入失敗亦保留提案，不視為已消化。

4. **收尾回報**：逐案列出裁定結果（駁回／晉升至哪個主題檔）、目前 conventions 清單，並提醒晉升的 convention 自下次寫入型指令開工掃描起生效。
