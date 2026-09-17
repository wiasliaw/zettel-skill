---
name: revise
description: filed 層筆記（filed-doc 與原子筆記）內容修改的唯一路徑：對話定出修訂方向、修訂稿經使用者確認、對抗式審查（dispatch zk-adversarial-review）後寫回原檔並增量 reindex。使用者在對話中要求修改某篇 filed 筆記時即進入本流程，無需記指令。
argument-hint: <filed 筆記路徑>
---

# /zk:revise

filed 層筆記（config `permanent.place`、`reference.place` 下）內容修改的**唯一路徑**。觸發有二：使用者明示 `/zk:revise <路徑>`；或模型情境觸發——使用者在對話中要求修改某篇 filed 筆記時即進入本流程（此為寫入型指令唯一的模型觸發例外）。寫入範圍：目標筆記與索引；MUST NOT 搬移筆記、MUST NOT 逆轉狀態（filed 絕不退回 staging）、MUST NOT 繞過審查。不觸碰日誌、不執行任何 git 指令、不修改 plugin 檔案。

## 流程

1. **開工前置**（依序，致命紀律）：
   a. 確認 vault 已初始化——可觀察訊號恰為 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立 config。
   b. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號或可入佇列的意外發現時，依同檔「proposal 寫入操作」寫入對應類型的 proposal 檔。
   c. **intent 掃描與離題捕獲**：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/intent.md` 執行（開工掃描協定、離題捕獲全程有效）；本指令的應用面：active intents 作為審查 dispatch 傳遞的脈絡。
   d. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。

2. **輸入限定**：目標 MUST 為 filed 層筆記（`permanent.place` 或 `reference.place` 下，place 活讀 config）；staging 筆記拒絕並指路——fleeting 草稿走 `/zk:fleeting`、literature 筆記走 `/zk:literature`。

3. **定修訂方向**：讀入目標筆記現行全文為基準，與使用者在對話中定出修訂方向與範圍。

4. **修訂稿確認**：呈現修訂後全文（或足以核對的明確 diff），等待使用者確認；確認前不寫入，否決則不寫。

5. **dispatch `zk-adversarial-review` agent**（`${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md`，其邊界自持）審查修訂稿：帶上修訂稿全文與筆記種類的錨定資訊（literature 型 filed-doc：frontmatter `source`；fleeting 型 filed-doc：無主錨；原子筆記：frontmatter `sources` 所指來源路徑），並附本次 active intents 摘要（如有）。原子筆記另由 agent 逐條驗證 inline 行文連結（不通過者列建議移除）。agent 一律只回報不修改。

6. **修正逐條評估後寫回**：報告建議逐條評估、與使用者確認後，把定稿寫回原檔——同一次編輯設定 `updated` 與活讀的 `version`；`created` 不動、`type` 與 tags 不因修訂改寫、未知欄位一律保留。合併或對質裁定轉入的修訂（同一宣稱的既有卡改為最新認識）同此路徑，寫回後由呼叫脈絡增補 `sources`。

7. **增量 reindex**（`qmd.enabled: true` 時；`false` 則跳過——無索引可維護）：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「qmd 呼叫」前置（vault root＋索引路徑驗證）與「共用 args 執行方式」，依序直接呼叫 `qmd update` 與 `qmd embed`，兩者缺一不可；任一失敗即停止並回報，不建立函式或腳本。

8. **收尾回報**：修訂摘要（改了什麼、為什麼）、審查報告完整格式（總條數、各分類條數、非忠實項與依據、存疑原因、結構層發現，原子筆記另含逐條連結判定）、reindex 狀態、記憶掃描註記。本指令被另一指令包裹呼叫時，此回報以其自身收尾格式原樣出現在該輪最終訊息，不得壓縮成外層摘要的一行。
