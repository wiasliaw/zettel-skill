# devlog

staging 筆記工作日誌（`.zettel/logs/`）的操作協定。日誌檔完整格式與範例見同目錄 `devlog-format.md`。無腳本代勞，一切機械動作以下列 prose 規則自律執行。

## 觸發限定

- 記錄與復盤僅由 `/zk:fleeting`、`/zk:literature` 的流程呼叫；MUST NOT 依對話情境獨立觸發。
- `/zk:file` 不做開工解析、不寫入回合，唯一日誌動作是歸檔成功後刪除日誌（下方「刪除」節）。
- `/zk:atomize`、`/zk:revise`、`/zk:query`、`/zk:init`、`/zk:adapt` 不觸碰日誌。
- subagents MUST NOT 讀寫日誌——事件由 dispatch 方（外層指令對話）記錄。

## 路徑鏡射

日誌以 staging 筆記為單位，路徑 MUST 為 `.zettel/logs/<staging 相對路徑>`（例，預設佈局下：`inbox/foo.md` → `.zettel/logs/inbox/foo.md`；place 名依 config，鏡射規則不變）。「相關日誌存在」的判定訊號 MUST 恰為該鏡射路徑的檔案存在與否，MUST NOT 以語意相似另行認定。filed 筆記（config `permanent.place`、`reference.place` 下）MUST NOT 建日誌。

## 開工解析（指令取得目標筆記路徑後執行）

1. 檢查鏡射路徑檔案是否存在。
2. **不存在** → 建檔（`mkdir -p` 鏡射父目錄）：依 `devlog-format.md` 格式寫入 STATUS 與首回合，首回合 Ask 寫入使用者請求逐字。
3. **存在** → 讀全檔復盤（STATUS 與最近回合），向使用者摘述現況：
   - 存在**開放回合**（有 Ask 無 Reply，即上次中斷點）→ MUST 從其最後 WIP／RUN 的「下一步」接續，且接續前 MUST 先寫一則 RUN 說明恢復點（從哪個 WIP／RUN 恢復、本次打算做什麼）。
   - 無開放回合 → 開新回合寫入本次 Ask。

## 衍生筆記建檔

session 中衍生新 staging 筆記時（例：`/zk:literature` 研讀中判斷應另切一篇 literature），MUST 隨筆記建檔同時建立其日誌，且首回合第一條 MUST 記建立緣由：衍生自哪個筆記（路徑與回合編號）、為什麼建立。

## 工作中記錄

- **RUN 事件**：材料轉換時記（精讀成果交回、成稿寫回、審查報告交回、重要判斷）。
- **WIP 檢查點**：工作暫告段落或即將中斷時記，內容恰為已完成／進行中／下一步三項。

## 收尾

指令每輪收尾 MUST 寫 Reply（至少含 SUMMARY 節，內容即該輪指令的收尾回報）並重寫 STATUS——STATUS 是唯一可重寫的區塊。

## 刪除（僅 /zk:file）

`/zk:file` 成功完成歸檔收尾後 MUST 刪除目標筆記的日誌（`rm .zettel/logs/<鏡射路徑>`）——刪除前 MUST 確認精簡出處（`intent`、`review`）已寫入歸檔筆記 frontmatter；無日誌則跳過。刪除即封存，不設 archive。文件自歸檔起唯讀，devlog 的使命隨之結束。中途失敗未達刪除點時日誌保留原樣，重跑歸檔成功後補刪；補刪 MUST NOT 解析日誌回合。

## 機械紀律（執行關鍵）

- 每次寫入前 MUST 重讀該日誌全檔（取得編號最大值、防併發覆寫）。
- 編號 `A-###`、`RUN-###`、`WIP-###` MUST 各自以該檔全檔掃描現有最大值 +1 產生，MUST NOT 憑記憶猜測。
- 回合區 append-only：已寫入內容 MUST NOT 刪除或改寫（STATUS 除外）。
- 失敗 MUST 照記（原因、當時判斷、下一步）。
- 回答日誌相關問題 MUST 引用回合／事件編號；日誌中沒有的事 MUST NOT 推測。

## 寫入範圍

恰為 `.zettel/logs/` 下的日誌檔（含刪除）；MUST NOT 觸碰筆記、config、模板檔、conventions／proposals／intents 與 qmd 索引。
