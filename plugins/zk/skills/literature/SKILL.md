---
name: literature
description: 研讀流程：對一個來源精讀（dispatch zk-research）、問答、成稿、成稿後逐篇對抗式審計（dispatch zk-adversarial-review）。一筆記一 source。
argument-hint: <筆記路徑 | source（URL 或書目出處）>
disable-model-invocation: true
---

# /zk:literature

寫入範圍：config `literature.place`；互動閘門：建檔確認閘＋成稿 draft-then-review。本指令不執行任何 git 指令、不修改 plugin 檔案、不歸檔筆記（歸檔屬 `/zk:permanent`）。

## 流程

1. **開工前置**（依序，致命紀律）：
   a. 確認 vault 已初始化——可觀察訊號恰為 vault root 存在 `.zettel.json`；不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立 config。
   b. 執行 `${CLAUDE_PLUGIN_ROOT}/skills/internals/sync.md`。
   c. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號時，依同檔「proposal 寫入操作」寫入 proposal 檔。
   d. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。

2. **輸入偵測**（依序判定；`literature.place` 活讀 config）：
   - `$ARGUMENTS` 是既有 `literature.place` 筆記路徑：進入「研讀流程」；本文非空時先讀入既有內容作為先前理解的基準。
   - 引數形似檔案路徑但該檔不存在：拒絕執行並說明路徑不存在，避免把打錯的路徑當成 source 建檔。
   - 其餘一律視為 source（URL 或書目出處）：走步驟 4 建檔確認閘。

   **一筆記一 source**：使用者對一篇既有非空筆記要求「加入另一個來源」時，不併入該筆記，改把新來源當非路徑輸入走建檔確認閘另建獨立筆記；兩篇日後拆出的原子筆記以 inline `[[wikilink]]` 互連。

3. **devlog 開工解析**：目標筆記路徑在手時（既有筆記，或步驟 5 建檔完成後），依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/devlog.md` 解析其日誌。

4. **建檔確認閘**：
   - 使用者的指示已明確表達建檔意圖（例如明說「幫這個 URL 建一篇 literature note」）：視同已確認。
   - 其餘：先在對話呈現建檔計畫（擬用 basename、標題、source 原文、已知的 Intent 文字）並等待確認；確認前不寫任何檔案，否決則不建檔。

5. **建檔規則**（確認後）：活讀 config `literature.templateFile` 所指模板於 `literature.place` 建立新筆記：
   - frontmatter 欄位以模板為準，不加入模板以外的欄位，也不模仿存量筆記的欄位或本文結構。
   - `source` 填入引數原文，MUST NOT 為空——它是精讀的唯一錨點。
   - `version` 活讀 config 現值；tags 依 tagging convention；basename 由來源主題推導，MUST 全 vault 筆記唯一（查名操作見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`「命名唯一性」；撞名即改名消歧並重查後才落檔）。
   - 本文只含 `## {{title}}` 標題行與模板固定的 Intent 節：使用者已表達閱讀聚焦時填入其文字，否則留空，待成稿時綜合填寫。
   - 同時依 devlog 協定建立日誌（首回合 Ask 寫入使用者請求逐字）。

   建檔完成後直接續走研讀流程。

## 研讀流程

對任一有效目標（剛建檔、既有空筆記、既有非空筆記）走同一條流程：

1. **前置**：驗證 frontmatter `source` 存在，缺則拒絕執行並說明原因。
2. **精讀**：dispatch `zk-research` agent（`${CLAUDE_PLUGIN_ROOT}/agents/zk-research.md`，其邊界自持）：帶上筆記路徑；使用者已陳述閱讀焦點（Intent 節文字或對話指示）時，dispatch prompt 帶上該焦點。成果以回傳交回對話作研讀底料，不寫入筆記。來源取不到全文時 agent 中止並回報原因，流程結束、筆記不動；不得降級改用二手資料。精讀成果交回後依 devlog 協定記一則 RUN。
3. **對話**：使用者提問，回答錨定 `source` 與精讀成果；來源之外的知識明確標示，不混入錨定回答。此階段不寫筆記本文、不 dispatch 審查 agent。
4. **成稿**：使用者給出收尾信號（「寫成筆記」或同義表述）時，把問答軌跡（或精讀底料）綜合成本文：
   - 空筆記為新寫，非空筆記為就地增修既有內容；同一次成稿依對話實際聚焦填寫或更新 Intent 節（無節者一併補節）；同一次編輯更新 `updated`（規則見 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md`）。未知 frontmatter 欄位一律保留。
   - 成稿路徑二擇一，依使用者表述判定：
     - **draft-then-review**（預設）：以 draft 呈現全文，使用者確認後才寫回，否決則不寫。
     - **直接寫入**：使用者以「直接寫入」「先落地」等同義表述明示時跳過 draft-then-review 逕行寫回；把關由步驟 5 的成稿後審計與使用者事後 review 承接。無此明示一律走預設路徑。
5. **審計（成稿後立即執行）**：成稿寫回後，同一次指令內立即 dispatch `zk-adversarial-review` agent（`${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md`，其邊界自持）；一次指令涵蓋多篇成稿時逐篇成稿即審，不集中到指令末尾。審計一律只回報不修改；修正由本對話逐條評估報告建議後套用（`updated` 於套用修正的同一次編輯統一設定）。報告交回後依 devlog 協定記一則 RUN。
   - 需要 slip-box 交叉核對素材時，由本對話先依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md` 查詢（唯讀，不更新索引），再把結果放入 dispatch prompt——審查 agent 無 Bash 工具。
   - 存疑項的補讀處置（optional）：MAY 提議把存疑所缺的知識另建一篇新的 literature 筆記（走建檔確認閘，一筆記一 source；衍生筆記依 devlog 協定同時建日誌並記建立緣由）。補讀是提議而非自動執行，MUST 經使用者裁定才建檔；未獲裁定時存疑項維持原樣。

研讀中使用者明示要求審計（「幫我審一遍」或同義表述）時，直接對筆記現況全文 dispatch 審查 agent，報告呈現於對話，不成稿、不寫回。審查可重複觸發，每次以當下全文為對象。

## 收尾

每輪收尾依 devlog 協定寫 Reply 並重寫 STATUS。**複合指令回報**：審查報告以其自身收尾格式完整呈現於最終訊息——總條數、各分類（忠實、事實錯誤、存疑，適用時加超出 Intent）條數、每個非忠實項的主張文字與判定依據、每項存疑為何無法定案；本指令被另一指令包裹呼叫時，此回報原樣出現在該輪最終訊息，不得壓縮成外層摘要的一行。
