---
name: adapt
description: 與使用者逐條裁定 .zettel/proposals/ 的提案佇列，依類型三分——convention 提案晉升為慣例或駁回；link 提案採納落 inline wikilink 或駁回；atomize 提案採納轉入 /zk:atomize 或駁回。駁回一律刪除即封存。
disable-model-invocation: true
---

# /zk:adapt

本流程是**對話流程**：逐條停下等待使用者裁定，MUST NOT 批次帶過、MUST NOT 代為決定。寫入範圍：`.zettel/conventions/` 與 `.zettel/proposals/`，以及採納 link 提案時的連結落點筆記（僅該篇的行文與 `updated`）；採納 atomize 提案時轉入 `/zk:atomize`，其寫入範圍由該指令自持。不觸碰 config、日誌與索引（落連結不 reindex——連結變動不需重建索引）；不修改 plugin 檔案——plugin 的變更不在 runtime 職權，只發生於 plugin repo 的開發流程。

## 流程

1. **前置**：確認 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`。依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 `.zettel/conventions/` 全部檔案（本流程需要知道既有主題檔，晉升時才能判定「建立或就地更新」）；全程有效：工作中發現使用者慣例訊號或可入佇列的意外發現時，依同檔「proposal 寫入操作」寫入對應類型的 proposal 檔。

2. **讀 proposals 並逐條討論**：讀 `.zettel/proposals/` 下每一份提案檔全文，逐檔向使用者呈現並討論，絕不略過任一份未經呈現的提案；佇列為空時直接回報「無待議提案」收尾。提案 frontmatter `type` 決定裁定路徑（見步驟 3）；缺 `type` 或值不在三類內時向使用者呈現原文並確認歸類後再裁定。convention 類的 User review 節已有內容時一併呈現，作為該案裁定的直接依據。

3. **逐條裁定**（與討論交錯進行，非全部討論完才裁定）：

   **套用前重讀**（適用全部類型）：每次寫入或刪除前，MUST 重讀該 proposal 全文，並重讀本次裁定將觸碰的檔案當前狀態（convention 類：重新列出 conventions 並讀對應主題檔全文，預定新建的主題也須重查是否已被建立；link 類：重讀兩端筆記全文）。以最後一次呈現並獲裁定的內容為比對基準；若有變動，MUST 先呈現差異並重新討論裁定，MUST NOT 沿用過期裁定寫入或刪除；提案已不存在時回報並跳過。無變動才以最小 diff 套用，保留所有無關內容，不回滾併發修改。

   **駁回**（適用全部類型）：

   ```bash
   rm .zettel/proposals/<檔名>
   ```

   駁回即刪除檔案，刪除即封存，不另設 archive 目錄。

   **採納，依類型三分**：

   - **`convention` → 晉升為 convention**：依 `${CLAUDE_PLUGIN_ROOT}/templates/convention.md` template，在 `.zettel/conventions/` 建立或就地更新對應主題檔——一主題一檔，MUST NOT 為同一主題另建副本（既有主題檔存在時就地更新它）。frontmatter 恰含 `when`（一句話觸發條件，綁可觀察訊號）。格式範例見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md`。寫入成功後，刪除提案前 MUST 再次執行套用前重讀：proposal 與裁定基準一致，且 convention 與剛寫入結果一致，才刪除該提案。若任一方改變，保留提案、呈現差異並重新裁定；已寫入的 convention 不自動回滾。寫入失敗亦保留提案，不視為已消化。

   - **`link` → 落 inline wikilink**：讀兩端筆記全文，逐項驗證後才落——連結目標 MUST 為原子筆記（config `permanent.place` 下；staging 與 filed-doc MUST NOT 被行文連結，目標不合格即不落，向使用者說明並改駁回或建議先擷取）；落點筆記的行文 MUST 確實論及目標概念（提案的提議位置與措辭供參考，以讀全文後的實況為準；行文並未論及時不硬塞，向使用者說明並駁回）。驗證通過即在落點行文落 inline `[[wikilink]]`，同一次編輯更新該筆記 `updated`（規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`）；不 reindex。寫入成功後刪除該提案。落連結屬**明定的非內容修改**、有自己的閘（`/zk:revise` 管的是本文主張內容的修改）：此為明定路徑，不轉 `/zk:revise`。

   - **`atomize` → 轉入 /zk:atomize**：使用者於本對話的採納即為明示發起，以該提案所指文件與宣稱為輸入執行 `/zk:atomize`（走其指定文件模式，流程與閘門由該指令自持）。提案檔 MUST NOT 於轉入前刪除——由 `/zk:atomize` 收尾的提案清理步驟處置。

4. **收尾回報**：逐案列出裁定結果（駁回／晉升至哪個主題檔／落了哪條連結／轉入 `/zk:atomize` 的執行結果）、目前 conventions 清單，並提醒晉升的 convention 自下次寫入型指令開工掃描起生效。轉入 `/zk:atomize` 的案子，其收尾回報依該指令的複合指令回報規則原樣呈現，不得壓縮成一行。
