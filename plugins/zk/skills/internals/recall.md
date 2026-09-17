# 召回操作

召回的操作規則。所有步驟於 vault root 執行；目錄一律活讀 config（`.zettel.json`）各 step 的 `place`，MUST NOT 假定預設目錄名。

## 分流

先活讀 `jq -r '.qmd.enabled' .zettel.json`（此為唯一 runtime 訊號）：

- **`true` → qmd 主漏斗**（下方「qmd 呼叫」節）。qmd 呼叫失敗 MUST 停止並回報錯誤，MUST NOT 靜默降級為 Grep——enabled 即表示已安裝且允許使用，失敗是要修的異常。
- **`false` → Grep 降級**：跳過 qmd，以 Grep／Glob 對 filed 目錄（config `permanent.place`、`reference.place`）暴力檢索：先以查詢的關鍵詞與其同義變形 Grep 內文、Glob 比對檔名，讀候選標題與命中段落初篩。不視為錯誤。

無論何者，MUST 另加**精確詞補充**：對查詢中的字面 token（識別碼、專有名詞、程式碼符號，如「EIP-4626」）跑精確詞 Grep 與檔名比對，補語意檢索對字面命中的弱點，命中併入同一候選池。

## qmd 呼叫（enabled: true 時每次呼叫的前置）

1. MUST 從 vault root 執行（collection path 為相對路徑、以 CWD 解析；子目錄執行 `qmd update` 會靜默清空索引）。
2. MUST 先驗證作用中索引是 project-local：

   ```bash
   qmd status | grep '^Index:'
   ```

   Index 路徑必須恰為 `<vault root>/.qmd/index.sqlite`；不是則立即停止並回報，絕不在該索引上執行任何 qmd 指令。qmd 找不到 project-local 索引時會靜默退回全域快取——空索引上的指令仍 exit 0，exit 0 不是索引可用的證據。

3. **args 拼接**：依下方「共用 args 執行方式」活讀該次 operation 的 args，由執行本文件規則的模型辨識引號與跳脫所表達的參數邊界，再產出逐參數引用的直接 CLI 呼叫。MUST NOT 解讀 flags 的業務語意或硬編碼個人化 flags／環境變數。例如 config `qmd.query` 為 `--context "long context" -n 5` 時，實際呼叫形如：

   ```bash
   qmd query '<查詢>' '--context' 'long context' '-n' '5'
   ```

4. **BM25 替代**：embed 模型不可用（`qmd query` 因 embed 模型報錯）時，以 `qmd search` 替代主漏斗；重新活讀 `.qmd.search`，依同一規則組成直接呼叫，MUST NOT 沿用 query 的 args。

## 共用 args 執行方式

recall、sync、`/zk:file`／`/zk:atomize`／`/zk:revise` 收尾 reindex 與 qmd-config 初始索引均依本節文字規則執行，不建立或內嵌解析程式、shell 函式或輔助腳本。呼叫方仍須先完成 vault root 與索引路徑驗證。

1. 每次呼叫前活讀 `.zettel.json` 中該 operation 的 args 字串（可用 `jq -r '.qmd.query // empty' .zettel.json` 等）。缺省為空字串；存在但不是字串時停止並回報。
2. 模型僅辨識 POSIX 引號／跳脫所表達的參數邊界：未引用的空白分隔參數；單引號內全部為字面值；雙引號保留其中空白；反斜線依所在引用情境處理跳脫；相鄰且未以空白分隔的片段屬同一參數。MUST 保留順序、值及明示的空參數，不推測或改寫 flags 語意。引號不成對、末尾跳脫不完整等格式錯誤，MUST 在呼叫 qmd 前停止並回報。
3. 產出直接 `qmd` 呼叫：subcommand 後先放必要參數（查詢全文恰為一個參數），再附加步驟 2 辨識的每個 args。**每個參數各自以 shell 單引號包住**；值內若有單引號，以結束引用、跳脫單引號、重新開始引用的方式表示，例如 `it's` 寫為 `'it'\''s'`。這適用於 Bash 與 zsh，不依賴變數的自動分詞。
4. 空字串或只有空白的 args 不附加參數；`""` 則附加一個 `''`。`$VAR`、`$(...)`、反引號、萬用字元及 shell 運算子均只作字面資料，MUST NOT 展開或執行。MUST NOT 直接把原始 args 當 shell 程式碼貼入、使用未加引號的 `$QUERY_ARGS`／`$(jq ...)` 展開，或使用 `eval`。

上方 `--context` 與 `-n` 僅示範含空白的參數，不是預設 flags。執行前核對產出的各參數與活讀值一致；qmd 失敗時依呼叫方規則停止並回報。

## 讀全文鐵律

任何命中都只是候選：作答實際引用、建立連結或**擷取素材**所依賴的候選，動筆前 MUST 讀過全文（深度閘門，通常 2 到 5 篇，其餘略讀至足以排除）。相似度分數與命中次數單獨 MUST NOT 構成引用、連結或入卡的理由——高分候選也必須先讀全文才可能被採用。

## 索引唯讀

本文件的全部操作皆為唯讀查詢，不執行 `qmd update`／`qmd embed`——主索引維護點恰為 sync 開工與 `/zk:file`／`/zk:atomize`／`/zk:revise` 收尾，與本文件無涉（「共用 args 執行方式」僅是它們共用的文字規則）。
