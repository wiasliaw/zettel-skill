# sync

`/zk:fleeting`／`/zk:literature`／`/zk:atomize` 開工前置的索引維護（開工冪等 reindex）。主索引維護點恰為兩類——本文件的開工 reindex，與 `/zk:file`、`/zk:atomize`、`/zk:revise` 的收尾增量 reindex；此外任何指令 MUST NOT 更新索引。本文件只做索引維護，不做任何筆記或記憶寫入。

## 步驟

1. 活讀 config 分流：

   ```bash
   jq -r '.qmd.enabled' .zettel.json
   ```

   - `false` → 跳過全部步驟（無索引可維護），回報「qmd 未啟用，跳過 reindex」。不視為錯誤、不引導安裝。
   - `true` → 續行。

2. **呼叫前置驗證**（致命約束）：MUST 於 vault root 執行（collection path 為相對路徑、以 CWD 解析；子目錄執行會靜默清空索引），先驗證：

   ```bash
   qmd status | grep '^Index:'
   ```

   Index 路徑 MUST 恰為 `<vault root>/.qmd/index.sqlite`；不是則停止並回報（qmd 找不到 project-local 索引會靜默退回全域快取，空索引上指令仍 exit 0——exit 0 不是索引可用的證據），絕不在錯誤索引上執行 update／embed。

3. **無條件 reindex**（冪等；索引已最新時為 no-op，不是錯誤）。依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「共用 args 執行方式」（只取該節，不執行召回流程），活讀 `.qmd.update`、逐參數引用後直接呼叫 `qmd update`；成功後再活讀 `.qmd.embed`，以同一規則直接呼叫 `qmd embed`。不建立函式或腳本。

   兩者缺一不可（`update` 收錄新增或搬移的檔案，`embed` 向量化它們）。任一指令失敗即停止並回報錯誤，MUST NOT 靜默當作成功。

## 收尾回報

一句白話併入外層指令的開工回報：索引狀態（首次建立、已更新或無變更；或「qmd 未啟用，跳過」）。
