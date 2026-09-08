# 筆記格式操作規則

筆記 frontmatter、命名與連結紀律的操作規則。所有路徑活讀 config `.zettel.json` 各 step 的 `place`，MUST NOT 假定預設目錄名。

## Schema SSOT

frontmatter 欄位結構的唯一來源是 config `fleeting`／`literature`／`permanent` 各 step 的 `templateFile` 所指模板檔（由 `/zk:init` 播種，使用者可自訂）。建立新筆記前 MUST 活讀 config 取得該 step 的 `templateFile` 並讀其欄位結構，MUST NOT 在本檔或對話中另擬一份 YAML：

```bash
jq -r '.fleeting.templateFile' .zettel.json   # 依 step 換鍵名
```

模板內的 HTML comment 是給人看的說明，成稿 MUST 移除。

## 兩層筆記

- **staging**（`fleeting.place`、`literature.place`）：扁平存放、不進 qmd 索引、不可被 `[[wikilink]]` 連結。
- **filed**（`permanent.place/<top>/`、`reference.place/<top>/`）：依 topic 分區存放、進索引（`qmd.enabled: true` 時）、可被連結。

歸檔（staging → filed）是兩層之間唯一的轉換方向，MUST NOT 逆向搬遷。

## tags

tags 的判準 MUST 讀 vault 的 tagging convention（`.zettel/conventions/` 內，由 `/zk:init` 播種；經由 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 的開工掃描自然涵蓋），MUST NOT 硬編碼字面值。無論 convention 內容為何：staging 筆記的 tags 可唯一判定其 step；filed 筆記的 tags 可判定 step 與 topic；MUST NOT 把一種 kind 改寫為另一種（最小不變量）。

## version 與 updated

- plugin 產出或操作的筆記 frontmatter MUST 含 `version`，值一律即時活讀，絕不複製模板 placeholder 字面值：

  ```bash
  jq -r '.version' .zettel.json
  ```

- 對既有筆記做出實際改動（本文或 plugin 管轄欄位）時，同一次編輯把 `updated` 改為當下本機時間（`"YYYY-MM-DDTHH:mm"`）；`created` 只在建檔時寫入一次，之後任何流程不再變動。未做出實際改動的流程（審查零修正、research 中止、唯讀處理）MUST NOT 動 `updated`。

## 未知欄位開放擴充

MUST NOT 因筆記帶有核心 schema 以外的未知 frontmatter 欄位而拒絕處理，且 MUST NOT 剝除或改寫任何未知欄位（例如供 Obsidian Base 使用的自訂屬性）。

## 命名唯一性

filed 筆記的 basename 以 `[[wikilink]]` 解析，MUST 全 vault 唯一；staging 筆記建檔占用既有筆記的 basename 同樣破壞此唯一性。建立任一筆記（staging 建檔亦然）或搬移任一 filed 檔案前，從 vault root 執行：

```bash
find . -path './.zettel' -prune -o -type f -name '<候選檔名>.md' -print
```

`.zettel/` 是記憶與日誌，不是筆記，故不參與撞名。若是搬移既有源筆記，再從結果排除**原 staging 路徑本身**（以完整路徑比對，不排除其他同 basename 的檔案）；搬移後該路徑會消失。其他命中才代表候選名已被占用，改名後重查才落檔，絕不覆寫既有檔案。候選名若含 `find -name` 萬用字元，須跳脫為字面比對。

寫入或搬移進 filed 分區前先 `mkdir -p <place>/<top>/`（`mv` 不會自動建立父目錄），搬移前重查目標未被占用，使用不覆寫的搬移方式並確認源路徑已消失、目標已存在，MUST NOT 只以 `mv` 的 exit code 判定成功。staging 筆記與移入 `reference.place` 的源筆記維持素樸 basename，不加前綴；`/zk:permanent` 拆分批次的命名規則（`{source-basename}({n})-{title-slug}.md`）由 `${CLAUDE_PLUGIN_ROOT}/agents/zk-atomizer.md` 步驟 8 自持。

## Topic 樹推導

top 的主題集合即時從目錄推導，MUST NOT 寫入 config：

```bash
find "<permanent.place>" "<reference.place>" -mindepth 1 -maxdepth 1 -type d -exec basename {} \; 2>/dev/null | sort -u
```

含空白的分區名視為單一 top；任一側目錄不存在或尚無分區時，不影響另一側列出。

一個 top「存在」的條件是其分區目錄下至少有一篇筆記（空分區不視為存在）。

## Note body shape（permanent）

permanent 筆記本文為 `## title`（H2，title as API 的主張式陳述句）加開頭一句粗體主張，再依內容切 `## sections`（皆為 H2，命名依內容而定，短篇可零個 section）。

## 連結紀律

操作要點：

- 一律 `[[wikilink]]`，單向、行內、無 `## Related` 區；散文提及處加別名（`[[note|display]]`）讓語句通順；唯一例外是 frontmatter 欄位值（不在散文中渲染，維持裸 wikilink）。
- 召回命中僅是候選：建連結前 MUST 讀候選全文，且本篇行文確實論及其概念才建立；同源與相似分數單獨 MUST NOT 構成理由。
- 概念尚無對應筆記時保持純文字，待筆記存在後再連結。
- staging 筆記 MUST NOT 被連結。
- 反向關係交由 Obsidian backlink pane 呈現，不手動補建反向連結。
- **自帶出處段落**：內容超出筆記主要出處（literature 的 `source`、permanent 的源筆記）之外時，僅在就地引用自己的支撐時才允許寫入——跨筆記綜合句 MUST 內聯 `[[wikilink]]` 到具體已歸檔且支撐該句的筆記；外部素材 MUST 帶 markdown 外部連結。這是寫入時的要求，不是事後刪改的依據（審查側的錨定規則由 `${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md` 自持）。
