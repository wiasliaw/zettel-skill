---
name: file
description: 歸檔一篇 staging 筆記：對抗式審查閘（dispatch zk-adversarial-review）通過後整份移入 filed-doc（reference.place/<top>/）、tags 正規化、寫入出處（intent／review）、刪除 devlog、增量 reindex。不拆分、不刪除筆記。
argument-hint: <staging 筆記路徑>
disable-model-invocation: true
---

# /zk:file

角色：歸檔——staging 筆記於品質關卡通過後**整份**移入 filed-doc，文件自此為唯讀來源；寫入範圍：目標筆記搬移與 frontmatter、devlog 刪除、索引；互動閘門：審查修正逐條評估＋top 分區確認。本指令 MUST NOT 拆分筆記（擷取屬 `/zk:atomize`）、MUST NOT 刪除筆記、不執行任何 git 指令、不修改 plugin 檔案。收尾是原子的：中途失敗即筆記與日誌留原位，修正後重跑本指令，不設續跑狀態。

## 流程

1. **開工前置**（依序，致命紀律）：
   a. 確認 vault 已初始化——可觀察訊號恰為 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立 config。
   b. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號或可入佇列的意外發現時，依同檔「proposal 寫入操作」寫入對應類型的 proposal 檔。
   c. **intent 掃描與離題捕獲**：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/intent.md` 執行（開工掃描協定、離題捕獲全程有效）；本指令的應用面：active intents 作為審查 dispatch 傳遞的脈絡與步驟 6c 出處 `intent` 欄的取值依據。
   d. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。
   e. 本指令不做 devlog 開工解析、不寫入回合；唯一日誌動作是步驟 7 的刪除。

2. **輸入限定**：目標 MUST 為存在的 staging 筆記——config `fleeting.place` 或 `literature.place` 下的檔案（place 活讀 config）；其他路徑拒絕執行並說明：filed 層筆記的修改走 `/zk:revise`，擷取走 `/zk:atomize`。

3. **審查閘**：dispatch `zk-adversarial-review` agent（`${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md`，其邊界自持）：帶上目標筆記路徑與筆記種類的錨定資訊（literature 筆記：frontmatter `source`；fleeting 筆記：無主錨），並附本次 active intents 摘要（如有）。agent 一律只回報不修改。報告交回後，本對話逐條評估建議、與使用者確認後套用選定修正（`updated` 於套用修正的同一次編輯統一設定）。**審查未通過**（致命問題未解：事實錯誤項未修正且使用者未明示放行）時 MUST NOT 移檔——筆記與日誌留原位，回報待處理項後收尾。

4. **top 分區確認**：依筆記內容與既有分區（topic 樹推導見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「Topic 樹推導」）提議 top；無合適者提議新分區。經使用者確認才續行。

5. **整份移入 filed-doc**：
   - 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「命名唯一性與撞名分流」查名（排除原 staging 路徑本身與 `.zettel/`；撞名即改名消歧並重查——歸檔筆記維持素樸 basename，不加前綴），`mkdir -p <reference.place>/<top>/` 後以不覆寫方式搬移，確認原路徑已消失、目標已存在。
   - MUST NOT 拆分、MUST NOT 刪除筆記。

6. **frontmatter 收尾**（搬移後，依序確認、可併為最少次編輯）：
   a. **tags 正規化**：依 tagging convention 的 filed 方案（出廠方案為 `[<kind>, <top>]`，`<kind>` 為筆記 frontmatter `type` 現值，MUST NOT 改標為別種 kind）。
   b. **`type: reference` 確認**：筆記屬程序知識工具筆記（標題不是宣稱，只有有效與過時——如操作手冊、設定步驟）時，經使用者確認才把 `type` 改為 `reference`；這是 `type` 唯一合法的改寫，也是終點狀態（不產生拆檔提案，內嵌真宣稱照常可被擷取）。非工具筆記保留原 `type`。
   c. **出處寫入**：`intent` 填驗證當下實際依循的 active intent（多筆時取本次實際依循者；無則留空）；`review` 填本次對抗式審查的結論摘記——忠實／存疑條數與存疑標的（例：`2026-09-17 審查：11 條忠實、1 條存疑（§3 效能數據無出處）`；值的規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「出處欄位」）。
   d. 同一次編輯設定 `updated` 與活讀的 `version`。除上列欄位外不改動任何 frontmatter 欄位與本文；未知欄位一律保留。

7. **刪除目標筆記日誌**：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/devlog.md`「刪除」節——刪除前 MUST 確認精簡出處（`intent`、`review`）已寫入歸檔筆記 frontmatter；無日誌則跳過。刪除即封存，不設 archive。

8. **增量 reindex**（`qmd.enabled: true` 時；`false` 則跳過——無索引可維護）：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「qmd 呼叫」前置（vault root＋索引路徑驗證）與「共用 args 執行方式」，依序直接呼叫 `qmd update` 與 `qmd embed`，兩者缺一不可；任一失敗即停止並回報，不建立函式或腳本。

9. **完整收尾回報**：歸檔去向（最終路徑、top 分區、是否新增分區）、tags 與 `type` 處置、寫入的出處欄位、日誌刪除與 reindex 狀態、記憶掃描註記，並列審查報告（總條數、各分類條數、每個非忠實項的主張文字與判定依據、每項存疑為何無法定案、結構層發現），各自完整。本指令被另一指令包裹呼叫時，此回報以其自身收尾格式原樣出現在該輪最終訊息，不得壓縮成外層摘要的一行。
