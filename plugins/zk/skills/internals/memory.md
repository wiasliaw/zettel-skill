# 記憶掃描與寫入（操作版）

conventions（長期記憶）與 proposals（提案佇列，三類：慣例／連結／拆檔）的掃描協定與寫入操作步驟。所有路徑相對 vault root（CWD）。

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

提案恰分三類，共用同一寫入協定，交由 `/zk:adapt` 後續裁定。任何一類都是「寫提案入佇列、不打斷當下工作、不代為執行」——proposals 是恆常性採納的唯一入口，無跳過佇列的直升路徑（唯一明定例外是 `/zk:init` 的出廠播種——出廠播種不是晉升）：

- **`convention`（慣例提案）**：工作中發現使用者慣例訊號（使用者的糾正、重複出現的偏好指示、值得留存的做事習慣）時寫入。恆常性慣例一律先寫 proposal，絕不逕自直接寫入 `.zettel/conventions/`。
- **`link`（連結提案）**：query 與日常工作中的意外發現——兩篇筆記可建行文連結時寫入。寫提案，絕不逕行落連結。
- **`atomize`（拆檔提案）**：檢索或寫作中發現行文需要指到還埋在 filed-doc 裡的宣稱時寫入。絕不代使用者發起擷取。`type: reference` 的筆記不產生拆檔提案（工具筆記是終點狀態；其內嵌的真宣稱被論證需要時，由使用者發起擷取照常處理）。

發現 plugin 自身的缺陷時不寫 proposal——runtime 不承載任何 plugin 變更流程；於對話中告知使用者即可。

寫入步驟：

1. **檔名格式**：`<主題 slug>.md`；來源資訊由 frontmatter `source` 欄承載，不入檔名。寫入前檢查檔名於 `.zettel/proposals/` 內唯一；已存在同名檔時改用更具體的主題 slug，絕不覆寫既有提案檔。
   - `source` 欄寫發現當下的情境一行描述（例如觸發的指令與正在處理的筆記）。例：檔名 `note-title-casing.md`，frontmatter `source: /zk:literature 成稿時使用者兩度糾正標題大小寫`。

2. **套 `${CLAUDE_PLUGIN_ROOT}/templates/proposal.md` template**：frontmatter 恰含 `type`（`convention`｜`link`｜`atomize`）、`source`、`created`（`YYYY-MM-DD`）三欄；body 依類型——
   - `convention`：依序四節 Observation／Proposal／Impact assessment／User review。User review 節記錄使用者對本提案的直接反饋：寫入提案當下已有的反饋原樣記入，其後使用者再給反饋時就地補記；無反饋時留空。該節是 `/zk:adapt` 裁定的直接依據。
   - `link`：兩端筆記（wikilink 與提及方向）、發現脈絡、提議的行文位置與措辭。
   - `atomize`：埋藏宣稱（以擬成卡的主張式陳述句表述）、所在文件與段落、發現脈絡（為何被需要）。

   convention 類完整範例（檔名 `note-title-casing.md`）：

   ```markdown
   ---
   type: convention
   source: /zk:literature 成稿時使用者兩度糾正標題大小寫
   created: 2026-09-08
   ---
   # 筆記標題的專有名詞大小寫應依來源慣例

   ## Observation

   成稿 draft 兩度被使用者糾正專有名詞大小寫（「OAuth」被寫成「Oauth」），使用者明言以來源慣例為準。

   ## Proposal

   新增一則 convention：筆記標題內的專有名詞大小寫依來源慣例，建檔時核對一次；不批次改動存量筆記。

   ## Impact assessment

   影響所有產出標題的寫入型指令（/zk:fleeting、/zk:literature 的建檔與成稿、/zk:atomize 的擬卡標題）。

   ## User review

   使用者 2026-09-08 於對話中回饋：大小寫依來源慣例即可，存量不回改。
   ```

3. proposal 檔一律進 `.zettel/proposals/`；後續裁定（駁回或依類型採納）屬 `/zk:adapt` 職權，本檔不重述。
