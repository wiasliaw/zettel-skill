---
name: atomize
description: 僅由使用者明示發起的擷取流程：從 filed-doc（或依一段意圖跨多份來源）dispatch zk-atomizer 擬拆分計畫，人工批准後落檔原子筆記；來源筆記絕不改動。
argument-hint: <filed-doc 路徑 | 一段意圖>
disable-model-invocation: true
---

# /zk:atomize

角色：擷取（extract）——從歸檔內容立原子筆記卡；寫入範圍：新卡（config `permanent.place`）與索引；互動閘門：拆分計畫人工批准。本指令 MUST 僅由使用者主動發起——發覺歸人，成檔歸 agent；系統途中發現埋藏宣稱時恰以拆檔提案入佇列回應，MUST NOT 代使用者發起本指令。來源筆記 MUST NOT 改動——不留 stub、不回寫、不刪除。不觸碰日誌、不執行任何 git 指令、不修改 plugin 檔案。收尾是原子的：中途失敗即留原位重跑，不設續跑狀態。

## 流程

1. **開工前置**（依序，致命紀律）：
   a. 確認 vault 已初始化——可觀察訊號恰為 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立 config。
   b. 執行 `${CLAUDE_PLUGIN_ROOT}/skills/internals/sync.md`（開工冪等 reindex；qmd 未啟用時其步驟自行跳過）。
   c. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號或可入佇列的意外發現時，依同檔「proposal 寫入操作」寫入對應類型的 proposal 檔。
   d. **intent 掃描與離題捕獲**：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/intent.md` 執行（開工掃描協定、離題捕獲全程有效）；本指令的應用面：active intents 作為意圖輸入的解讀與拆分計畫取捨的依據。
   e. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。

2. **輸入偵測**（二擇一；place 活讀 config）：
   - **指定文件**：`reference.place` 下的 filed-doc 路徑，得點名只拆其中某條宣稱。staging 路徑拒絕並指路 `/zk:file`（先歸檔才擷取）；`permanent.place` 下的卡拒絕並指路 `/zk:revise`（卡的修改不走本指令）。
   - **一段意圖**：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md` 走檢索漏斗（主從順序與精確詞補充依該檔），併入使用者指名的筆記，取得候選來源；對實際採用的每份來源讀全文後決定素材範圍（讀全文鐵律——擷取素材所依賴的候選動筆前必讀全文）；來源得跨多份筆記。
   - `/zk:adapt` 採納拆檔提案轉入本流程時（使用者於該對話的採納即為明示發起），以提案所指文件與宣稱為輸入，走指定文件模式。

3. **dispatch `zk-atomizer` agent 擬拆分計畫**（`${CLAUDE_PLUGIN_ROOT}/agents/zk-atomizer.md`，其邊界自持）：帶上輸入模式與素材範圍（指定文件路徑與點名宣稱，或意圖文字＋採用來源的路徑清單）、active intents 內容、命中的 conventions 摘要。agent 產出恰為拆分計畫（每卡：主張式標題、topic、內文、`sources`、行文連結提議、附註），MUST NOT 寫入任何檔案。

4. **拆分計畫人工批准**：把計畫完整呈現於對話（含 agent 的附註、同宣稱既有卡的合併或對質建議），與使用者逐項批准、修改或駁回。**批准前 MUST NOT 落任何檔案**；計畫全數駁回時 MUST NOT 落檔，僅回報收尾。

5. **落卡**（僅批准項，逐卡）：
   - 活讀 config `permanent.templateFile` 所指模板取得欄位結構（schema SSOT，MUST NOT 另擬欄位），`mkdir -p <permanent.place>/<top>/` 後建卡（擬新增分區經批准即成立）。
   - basename 即主張式標題；依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「命名唯一性與撞名分流」查名。**主張式標題撞名 MUST 觸發合併或對質、MUST NOT 改名迴避**：內容一致 → 不另建卡，僅對既有卡增補 `sources`（同步 `updated`）；內容衝突 → 對質：呈使用者裁定，既有卡經 `/zk:revise` 修訂為最新認識後，同樣增補 `sources`。
   - frontmatter：`type: permanent`、tags 依 tagging convention 的 filed-atomic 方案（top 與歸檔資料夾一致）、`created`／`updated` 填當下本機時間、`version` 活讀 config 現值（絕不用模板 placeholder 字面值）、`sources` 依計畫（wikilink 清單指向來源筆記）、`intent` 填本次實際依循的 active intent（無則留空）、`review` 見步驟 6。模板內的 HTML comment 成稿移除。
   - **行文連結逐條驗證後才落**：讀連結目標全文、本卡行文確實論及其概念才建立（同批新卡尚未進索引，以本次計畫的確切路徑直接讀取，不依賴召回）；staging 與 filed-doc MUST NOT 被行文連結。
   - **來源筆記 MUST NOT 改動**：不留 stub、不回寫、不刪除；出處方向恆為卡 → 來源。

6. **審查**：
   - **跨來源綜合成卡**（素材跨多份來源的卡）MUST dispatch `zk-adversarial-review` agent（`${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md`，其邊界自持）：帶上卡路徑、`sources` 所指來源路徑、同批卡確切路徑與 active intents 脈絡；報告交回後修正逐條評估套用（`updated` 於套用修正的同一次編輯統一設定），該卡 `review` 欄填本次審查的結論摘記——忠實／存疑條數與存疑標的（值的規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「出處欄位」）。
   - **單文件擷取的卡**（內容已於來源歸檔時過審）不 dispatch 審查，僅步驟 5 的行文連結逐條驗證；`review` 欄不重審：抄錄來源文件 frontmatter 的 `review` 摘記並標明承自（例：`承自 [[來源]] 歸檔審查：<摘記>`；值的規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「出處欄位」）。

7. **增量 reindex**（`qmd.enabled: true` 時；`false` 則跳過——無索引可維護）：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「qmd 呼叫」前置（vault root＋索引路徑驗證）與「共用 args 執行方式」，依序直接呼叫 `qmd update` 與 `qmd embed`，兩者缺一不可；任一失敗即停止並回報，不建立函式或腳本。

8. **相關提案清理**：`.zettel/proposals/` 中與本次擷取宣稱對應的拆檔提案（含由 `/zk:adapt` 轉入的該份提案），於收尾一併刪除（刪除即封存）；無則跳過。

9. **完整收尾回報**：落卡清單（檔名、top、是否新增分區、`sources`）、合併或對質的處置結果、每卡建立的行文連結（目標與理由）與駁回的提議、審查報告完整格式（跨來源卡適用：總條數、各分類條數、非忠實項與依據、存疑原因、逐條連結判定）、reindex 狀態、清理的提案、記憶掃描註記。本指令被另一指令包裹呼叫時，此回報以其自身收尾格式原樣出現在該輪最終訊息，不得壓縮成外層摘要的一行。
