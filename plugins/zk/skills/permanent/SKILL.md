---
name: permanent
description: 全自動把一篇 staging 筆記拆分成原子 permanent 筆記（dispatch zk-atomizer）、源筆記歸檔 reference、刪 devlog、增量 reindex、對抗式驗證（dispatch zk-adversarial-review）、完整回報。零確認、無互動閘門。
argument-hint: <staging 筆記路徑>
disable-model-invocation: true
---

# /zk:permanent

步驟總序：拆分 → 內連結 → 歸檔 reference → 刪 devlog → 增量 reindex → 驗證 → 回報。本指令不執行任何 git 指令、不修改 plugin 檔案。

## 流程

1. **開工**（致命紀律）：
   a. 活讀 `.zettel.json` 取得全部 step `place` 與 `qmd.enabled`；config 不存在則停止並引導執行 `/zk:init`，MUST NOT 代為建立。
   b. 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 掃描 conventions／proposals；全程有效：工作中發現使用者慣例訊號時，依同檔「proposal 寫入操作」寫入 proposal 檔。
   c. 寫入紀律（全程有效）：任何寫入前重讀檔案當前狀態（防 Obsidian 併發編輯）、最小 diff、MUST NOT 回滾使用者的併發修改。
   d. 本指令不做 devlog 開工解析、不寫入回合；唯一日誌動作是步驟 4b 的刪除。

2. **宣告與輸入限定**：全自動執行到底，全程零確認、無互動閘門（本次執行開始後不再中斷詢問）。先依下方「部分完成的續跑」檢查是否為既有歸檔操作；符合者只恢復收尾，不進步驟 3。進步驟 3 前 MUST 另以 `Glob` 檢查 `<permanent.place>/*/<源筆記 basename>(*`（basename 不含 `.md`，括號按字面比對）：有命中即代表存在先前中斷的部分批次，此時若無可核對的續跑資料（本對話的拆分批次資料與步驟 3 回傳，或使用者提供），停止並回報命中路徑，MUST NOT 重新拆分。新拆分的輸入 MUST 限定為存在的 staging 筆記——`fleeting.place` 或 `literature.place` 下的檔案；其他路徑拒絕執行並說明。reference 路徑只能用來辨識已開始操作的續跑，不能當作新拆分輸入。

3. **dispatch `zk-atomizer` agent**（`${CLAUDE_PLUGIN_ROOT}/agents/zk-atomizer.md`）拆分與內連結：帶上源筆記路徑，保留拆分時的 `source-basename`（不含 `.md`，之後源筆記改名亦不變）。dispatch 前 MUST 先在對話輸出「拆分批次資料」：vault root、源筆記 staging 相對路徑、source-basename、源筆記 SHA-256、日誌鏡射路徑與其 SHA-256（不存在則明記無）。雜湊須從實際檔案計算（如 `shasum -a 256`），MUST NOT 憑空填寫；這是進度輸出，不等待確認——它讓 agent 落檔後、「歸檔續跑資料」輸出前的中斷窗口也留下可核對的批次資訊。agent 讀源筆記全文並自主拆分成一概念一檔的原子 permanent 筆記、選定 top tag、建 inline 連結、落檔至 `permanent.place/<top>/`，全程零確認、絕不中途停下提問；其完整裁量邊界已自我完備，本流程不重述，只負責 dispatch 與交回結果。一次執行涵蓋多篇源筆記（分多輪 dispatch）時，每輪 dispatch prompt MUST 另列出同批先前輪次拆分時的 source-basename、全部原子筆記確切路徑與其落檔分區，並註明尚未進索引、須以路徑與 `Glob` 發現；reference 最終檔名另列，不取代原前綴。

4. **歸檔收尾**（全部原子筆記落檔後依序完成，中途無使用者檢查點）。**空結果防護**：步驟 3 回傳零篇原子筆記（全部概念被判定素材不足而略過）時，MUST NOT 進入本步驟與步驟 5——保留源筆記與其日誌於原位、不搬移、不刪日誌、不 reindex，直接跳到步驟 6 回報各概念被略過的原因（可建議回到 `/zk:fleeting`／`/zk:literature` 補足素材）：

   a. **源筆記處置**：源筆記移入 `reference.place/<top>/`，MUST NOT 刪除。`<top>` 取本批原子筆記 top tag 多數決；平手或難以判定時取拆分輸出順序中最先出現的 top，不中斷、不詢問。移動前依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/note-format.md` 檢查源筆記 basename 全域唯一（排除原 staging 路徑自身與 `.zettel/`；其他撞名自行改名消歧）並 `mkdir -p <reference.place>/<top>/`。把源筆記 `tags` 依 tagging convention 的 filed 方案正規化（出廠方案為 `[<kind>, <top>]`，`kind` 為源筆記 frontmatter `type` 之值，MUST NOT 改標為別種 kind），同一次編輯設定 `updated` 與活讀的 `version`。除 `tags`、`updated`、`version` 外，不改動任何 frontmatter 欄位與本文；未知欄位一律保留。**正規化後、實際搬移前**，MUST 先在對話輸出下方定義的「歸檔續跑資料」，再以不覆寫方式搬移；若目標臨時被占用，重新消歧並輸出更新後的續跑資料，才重試搬移。確認原 staging 路徑已消失且目標檔案符合續跑資料後，才進 4b。

   b. **刪除源筆記日誌**：重讀並核對 `.zettel/logs/<源筆記原 staging 鏡射路徑>` 與續跑資料中的刪除前 SHA-256；一致才刪除該確切檔案，無日誌則跳過。日誌改變或原路徑出現新 staging 筆記時停止並回報，不刪除新工作。刪除即封存，不設 archive；失敗時保留日誌，輸出失敗點與續跑資料。源筆記已搬移者依下方續跑路徑補刪，MUST NOT 再拆分。

   c. **增量 reindex**（`qmd.enabled: true` 時；`false` 則跳過——無索引可維護）：從 vault root 先驗證 `qmd status` 的 Index 路徑恰為 `<vault root>/.qmd/index.sqlite`（不是則停止並回報），再依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/recall.md`「共用 args 執行方式」（只取該節，不執行召回流程），活讀 `.qmd.update`、逐參數引用後直接呼叫 `qmd update`；成功後再活讀 `.qmd.embed`，以同一規則直接呼叫 `qmd embed`。任一失敗即停止並回報，不建立函式或腳本。

      兩者缺一不可。這是主索引兩個維護點之一（另一點是 sync 開工 reindex）。

5. **dispatch `zk-adversarial-review` agent**（`${CLAUDE_PLUGIN_ROOT}/agents/zk-adversarial-review.md`）驗證：帶上源筆記路徑（已在 `reference.place/<top>/` 的新位置）與本批全部原子筆記路徑（permanent 筆記 frontmatter 無 `source` 欄，審查 agent 拿不到就無從錨定）。agent 逐篇驗忠實度（分段錨定，主錨為源筆記）、逐條驗證 inline 連結（目標全文讀過、行文確實提及才通過；不通過列「建議移除」）；一律只回報不修改。審查回傳後，本對話逐條評估建議、套用選定的修正（含移除連結），並於套用修正的同一次編輯統一設定該筆記的 `updated`（修正後的內容由下一個索引維護點自然收錄，本指令不再 reindex——維護點恰兩處）。

6. **完整收尾回報**：於對話並列呈現歸檔結果與審查報告，各自完整：
   - 歸檔結果：落檔的原子筆記清單（檔名、top tag、是否新增分區）、每篇建立的 inline 連結（目標與理由）、源筆記的最終去向、任何因素材不足而略過的概念與原因、日誌刪除與 reindex 狀態。
   - 審查報告：總條數、各分類（忠實、事實錯誤、存疑）條數、每個非忠實項的主張文字與判定依據、每項存疑為何無法定案、每條連結的通過與否及理由；並逐項區分「agent 建議」與「本對話實際採用」（採用與否及理由）。

   本指令被另一指令包裹呼叫時，此回報以此完整格式原樣出現在該輪最終訊息，不得壓縮成外層摘要的一行。

## 部分完成的續跑

「歸檔續跑資料」是搬移前已輸出至對話的結構化資料，不是 devlog，不新增 vault 狀態檔、不讀取日誌回合。MUST 含：vault root、原 staging 相對路徑、最終 reference 相對路徑、拆分時 source-basename、正規化後源筆記的 SHA-256、原日誌鏡射路徑與其 SHA-256（不存在則明記無）、全部原子筆記路徑，以及步驟 3 的完整回傳摘要（供續跑審查與完整回報）。雜湊須從實際檔案計算（如 `shasum -a 256`），MUST NOT 憑空填寫。這是進度輸出，不等待確認；搬移前再次核對源筆記未變，改變則停止並保留檔案。

使用者重跑原 staging 路徑或最終 reference 路徑時，先比對本對話已有、或使用者提供的續跑資料；跨對話需提供此資料，MUST NOT 僅憑同名、日誌存在或內容相似猜測歸檔對應。核對 vault、路徑所屬 config place、日誌鏡射規則、全部原子筆記仍存在，且現存源筆記 SHA-256 符合資料，再依檔案狀態續行：

| 原 staging | 記錄的 reference | 動作 |
|---|---|---|
| 存在且雜湊相符 | 不存在 | 拆分已完成；跳過步驟 3，從 4a 的搬移續行，不重做正規化 |
| 不存在 | 存在且雜湊相符 | 搬移已完成；跳過步驟 3、4a，從 4b 補刪日誌，續做 4c、5、6 |
| 同時存在／同時不存在／任一核對不符 | — | 停止並回報衝突與確切路徑，不覆寫、不刪日誌、不重新拆分 |

本對話只有「拆分批次資料」與步驟 3 回傳、尚未輸出「歸檔續跑資料」時（中斷落在原子筆記全數落檔後、4a 輸出續跑資料前）：核對 vault root、原 staging 相對路徑、source-basename 相符，步驟 3 回傳所列原子筆記全數存在，且現存源筆記 SHA-256 符合拆分批次資料所記雜湊（代表正規化尚未進行），則跳過步驟 3、從 4a 頭部續行（正規化、輸出歸檔續跑資料，再搬移）。現存源筆記雜湊與拆分批次資料不符時（可能中斷於正規化中途），停止並回報現存狀態與確切路徑，MUST NOT 逕行正規化或搬移、MUST NOT 重新拆分。

日誌已刪除時 4b 為 no-op；reindex 不確定是否完成時可冪等重跑。不存在可核對的續跑資料時，缺失的 staging 或 reference 輸入均停止並說明需要哪份資料，不承諾能自動推斷恢復；步驟 2 偵測到 `<源筆記 basename>(` 前綴的既有原子筆記而無可核對資料時亦同——停止並列出命中路徑，不重新拆分、不清除任何檔案。此路徑處理原子筆記全數落檔後的正規化／搬移／刪日誌中斷，不把部分拆分誤判成可重做的完整批次。
