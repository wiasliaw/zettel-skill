# qmd-config

qmd 設定的操作規則，僅由 `/zk:init` 於訪談定為啟用 qmd 時呼叫。分兩段：**訪談段**只產出決策、不落任何檔案；**落地段**在 `/zk:init` 的 scaffold 計畫確認閘核准後才執行。qmd 的模型／後端選型歸使用者自理，本文件不代決、不寫入 `.qmd/index.yml` 的 `models` 鍵，除非使用者主動指定。

## 訪談段（不落檔）

1. **探測環境**：`qmd --version` 確認版本；`qmd doctor` 看環境概況（embed 模型、運算後端），結果呈現給使用者供 args 決策參考，不據以代答。

2. **索引範圍**（與使用者定）：collections MUST 含 config `permanent.place` 與 `reference.place` 兩個目錄，MAY 依使用者指定加入 vault 其他資料夾，MUST NOT 含 `fleeting.place` 與 `literature.place`（staging 不進索引）；每個 collection `pattern: "**/*.md"`，MUST NOT 設 `ignore`（staging 不在 collection path 內，本就不被索引，沒有東西需要排）、MUST NOT 設 collection `update-cmd`（reindex 只在 sync 開工與 `/zk:file`／`/zk:atomize`／`/zk:revise` 收尾兩類維護點手動執行；且 plugin 不執行 git）。

3. **各 operation 的 args 字串**（與使用者定）：`query`／`search`／`update`／`embed` 四鍵，落 config `qmd` 對應鍵；缺省空字串。向使用者說明語意——這些字串於呼叫時原樣拼接於 subcommand 之後（例 `qmd query "<查詢>" <args>`），依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「共用 args 執行方式」的文字規則保留 POSIX 引號／跳脫的參數邊界，不解讀 flags 語意、不執行 shell 展開；使用者無偏好時保持空字串。

4. **context**：`global_context` 一句話交代整個 vault 是什麼；各 collection 的 `context` 依 `/zk:init` 訪談的初始 topic 分區以「路徑前綴 → 描述」交代（分區留空則先只寫 global）。

5. 以上決策全部交回 `/zk:init` 納入 scaffold 計畫呈現；本段 MUST NOT 寫任何檔案。

## 落地段（scaffold 計畫確認後，由 /zk:init 呼叫）

1. **寫 `.qmd/index.yml`**（project-local 索引）。完整範例（出廠預設 place、訪談加入 `essays/`、初始分區含 `security` 時）：

   ```yaml
   global_context: "Personal Zettelkasten vault; notes in Traditional Chinese with English technical terms"
   collections:
     zettelkasten:
       path: zettelkasten
       pattern: "**/*.md"
       context:
         "/": "permanent atomic notes, one concept per file, partitioned by topic"
         "/security": "web and protocol security notes"
     reference:
       path: reference
       pattern: "**/*.md"
     essays:
       path: essays
       pattern: "**/*.md"
   ```

2. **驗證 project-local 索引**：

   ```bash
   qmd status | grep '^Index:'
   ```

   Index 路徑 MUST 恰為 `<vault root>/.qmd/index.sqlite`；不是則停止並修正後才續行（qmd 會靜默退回全域快取，空索引上指令仍 exit 0，exit 0 不是索引可用的證據）。

3. config `qmd` 鍵的值（`enabled: true` 與四條 args 字串）交由 `/zk:init` 寫入 `.zettel.json`（config 落檔屬 init 職權）。

4. **建立初始索引**（從 vault root）。依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「共用 args 執行方式」（只取該節，不執行召回流程），活讀 `.qmd.update`、逐參數引用後直接呼叫 `qmd update`；成功後再活讀 `.qmd.embed`，以同一規則直接呼叫 `qmd embed`。不建立函式或腳本。

   兩者依序、缺一不可；任一失敗即回報錯誤，不靜默續行。

5. **驗證**：`qmd doctor` 全綠才算落地完成；有紅項時呈現給使用者並說明（模型／後端類異常屬使用者自理範圍，plugin 不代修）。
