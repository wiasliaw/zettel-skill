# 記憶掃描與寫入（操作版）

conventions（長期記憶）與 proposals（提案佇列）的掃描協定與寫入操作步驟。所有路徑相對 vault root（CWD）。

## conventions 開工掃描協定

任何寫入型指令開工時，依序：

1. 掃描 `.zettel/conventions/` 下全部檔案的 frontmatter（只讀 frontmatter，不讀本文）。目錄不存在或為空時跳過本節，於回報註記「無 conventions」。
2. 依 `when` 欄位判定與本次工作的相關性：
   - **命中**（`when` 描述的可觀察訊號符合本次工作）→ 讀該檔全文並遵循。
   - **無法判定**（讀 frontmatter 仍看不出是否相關）→ 視同命中，讀全文（存疑從讀）。
   - **明確不相關** → 止於 frontmatter，不讀全文。
   - **缺 frontmatter** → 視同命中，讀全文並遵循，且在該次執行的回報中註記「`<檔名>` 缺 frontmatter」。

convention 檔案格式完整範例（template 見 `${CLAUDE_PLUGIN_ROOT}/templates/convention.md`）：

```markdown
---
when: 建立或重命名 literature 筆記時
---
# literature-naming

literature 筆記檔名一律用 dash，不用 underscore（先例：eip-165 改名裁定）。
```

frontmatter 恰含 `when` 一個欄位（一句話觸發條件，綁可觀察訊號）。

## proposals 順帶掃描協定

同一次開工掃描順帶執行：掃描 `.zettel/proposals/` 下全部檔名（依檔名即可判斷主題，無需逐份讀全文），若存在與本次工作主題相關的提案，在回報中提示一行「有待議提案：`<檔名>`」。此提示絕不具約束力——絕不因存在相關提案而改變本次工作的執行方式、或暫停執行等待該提案裁定。

## proposal 寫入操作

工作中發現使用者慣例訊號（使用者的糾正、重複出現的偏好指示、值得留存的做事習慣）時，隨手寫入一個 proposal 檔至 `.zettel/proposals/`，交由 `/zk:adapt` 後續裁定。任何恆常性慣例一律先寫入 proposal，絕不逕自直接寫入 `.zettel/conventions/`——proposal 是晉升的唯一入口，無跳過佇列的直升路徑（唯一明定例外是 `/zk:init` 的出廠播種——出廠播種不是晉升）。

發現 plugin 自身的缺陷時不寫 proposal——runtime 不承載任何 plugin 變更流程；於對話中告知使用者即可。

寫入步驟：

1. **檔名格式**：`<主題 slug>.md`；來源資訊由 frontmatter `source` 欄承載，不入檔名。寫入前檢查檔名於 `.zettel/proposals/` 內唯一；已存在同名檔時改用更具體的主題 slug，絕不覆寫既有提案檔。
   - `source` 欄寫發現當下的情境一行描述（例如觸發的指令與正在處理的筆記）。例：檔名 `note-title-casing.md`，frontmatter `source: /zk:literature 成稿時使用者兩度糾正標題大小寫`。

2. **套 `${CLAUDE_PLUGIN_ROOT}/templates/proposal.md` template**：frontmatter 恰含 `source` 與 `created`（`YYYY-MM-DD`）兩欄；body 依序四節 Observation／Proposal／Impact assessment／User review。User review 節記錄使用者對本提案的直接反饋：寫入提案當下已有的反饋原樣記入，其後使用者再給反饋時就地補記；無反饋時留空。該節是 `/zk:adapt` 裁定的直接依據。完整範例（檔名 `note-title-casing.md`）：

   ```markdown
   ---
   source: /zk:literature 成稿時使用者兩度糾正標題大小寫
   created: 2026-09-08
   ---
   # 筆記標題的專有名詞大小寫應依來源慣例

   ## Observation

   成稿 draft 兩度被使用者糾正專有名詞大小寫（「OAuth」被寫成「Oauth」），使用者明言以來源慣例為準。

   ## Proposal

   新增一則 convention：筆記標題內的專有名詞大小寫依來源慣例，建檔時核對一次；不批次改動存量筆記。

   ## Impact assessment

   影響所有建檔與成稿的寫入型指令（/zk:fleeting、/zk:literature、/zk:permanent 的標題產出）。

   ## User review

   使用者 2026-09-08 於對話中回饋：大小寫依來源慣例即可，存量不回改。
   ```

3. proposal 檔一律進 `.zettel/proposals/`；後續裁定（駁回或晉升）屬 `/zk:adapt` 職權，本檔不重述。
