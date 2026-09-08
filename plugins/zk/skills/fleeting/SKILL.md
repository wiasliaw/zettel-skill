---
name: fleeting
description: 以蘇格拉底式思考夥伴身份發展一則原始想法或既有的 fleeting 草稿，經 draft-then-review 確認後寫回 staging。
argument-hint: <筆記路徑 | 想法文字>
disable-model-invocation: true
---

# /zk:fleeting

角色：蘇格拉底式思考夥伴；寫入範圍：config `fleeting.place`；互動閘門：draft-then-review。本指令只負責發展草稿：不拆分、不自動接續執行 `/zk:permanent`、不執行任何 git 指令、不修改 plugin 檔案。

## 流程

1. **開工前置**（依序，致命紀律）：
   a. 確認 vault 已初始化——可觀察訊號恰為 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立 config。
   b. 執行 `${CLAUDE_PLUGIN_ROOT}/skills/internals/sync.md`（索引維護；qmd 未啟用時其步驟自行跳過）。
   c. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號時，依同檔「proposal 寫入操作」寫入 proposal 檔。
   d. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。

2. **輸入偵測**（`fleeting.place` 活讀 config `jq -r '.fleeting.place' .zettel.json`）：
   - `$ARGUMENTS` 是既有 `fleeting.place` 筆記路徑：就地發展該筆記，先讀入既有本文作為對話基準。
   - 其餘一律視為原始想法文字：對話從該文字出發（目標筆記於步驟 6 建立）。

3. **devlog 開工解析**：既有筆記路徑在手時，依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/devlog.md` 解析其日誌（無日誌建檔、有日誌載入復盤；開放回合則先寫 RUN 恢復點再接續）。文字輸入的日誌隨步驟 6 建檔時一併建立。

4. **發展方法：蘇格拉底式提問**。以 Paul 與 Elder 的六類問題交叉詰問這個想法，揭露其假設、證據與推論後果：
   - **釐清**：具體的意思是什麼，能否舉例。
   - **假設**：預設了什麼未經檢驗的前提。
   - **證據**：怎麼知道這件事，依據是什麼。
   - **觀點**：誰會不同意，換個立場會怎麼看。
   - **推論**：如果這為真，會導出什麼後果。
   - **後設**：這件事為什麼重要。

   提問紀律：
   - 先讓使用者傾倒想法再開始追問，不要一開始就丟出問題清單。
   - 每一輪挑當下最弱的一條線索追問，不按固定順序覆蓋六類。
   - 預設立場是挑戰而非附和。
   - 停止條件有二：想法已能收斂成一句話陳述，或對話中浮現矛盾。
   - 不機械式逐一覆蓋六類問題，也不自問自答。

5. **draft-then-review**：呈現實際修訂後的全文，等待使用者確認；確認前不落檔，使用者否決則不寫入。

6. **寫回**（使用者確認後）：
   - 既有筆記：就地修訂或擴充，保留原始想法本身，不另建新檔；同一次編輯更新 `updated`（規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`）。
   - 純文字輸入：活讀 config `fleeting.templateFile` 所指模板建立新筆記於 `fleeting.place`；`version` 活讀 config 現值（絕不用模板 placeholder 字面值）、tags 依 tagging convention、basename MUST 全 vault 筆記唯一（查名操作見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「命名唯一性」；撞名即改名消歧並重查後才落檔）。建檔同時依 devlog 協定建立其日誌（首回合 Ask 寫入使用者請求逐字）。

7. **收尾**：依 devlog 協定寫 Reply（SUMMARY 即本輪收尾回報）並重寫 STATUS。對話收尾回報：寫回了什麼、想法收斂到哪、devlog 狀態，以及記憶掃描的註記（相關 conventions、待議 proposals）。
