# devlog 檔案格式

日誌檔的完整格式定義與範例（操作協定見同目錄 `devlog.md`）。

## 結構

每檔恰為兩部分，依序：

1. **STATUS 投影**（頂部，≤60 行）：欄位恰為 `Note`、`Last round`、`Proven`、`Open`、`Next` 五個且順序固定。每輪收尾重寫（全檔唯一可重寫的區塊）。
   - `Note`：目標筆記的 vault 相對路徑。
   - `Last round`：最近回合編號、日期與觸發指令。
   - `Proven`：已確立的成果與結論（跨回合累積）。
   - `Open`：開放中的問題與未完成事項。
   - `Next`：下一步。
2. **回合序列**（依時序 append-only）：每回合以 Ask 區塊開始，其後穿插 RUN 事件與 WIP 檢查點，以 Reply 區塊收束。有 Ask 無 Reply 的回合是**開放回合**（中斷點）；下次開工不另開新回合，先寫一則 RUN 說明恢復點後在同回合內接續。

## 區塊定義

| 區塊 | 標題格式 | 內容 |
|---|---|---|
| Ask | `## A-###` | 使用者請求逐字（引用區塊），MUST NOT 改寫或摘要 |
| RUN | `### RUN-###` | 材料轉換事件一則（發生了什麼、產出是什麼） |
| WIP | `### WIP-###` | 檢查點，內容恰為已完成／進行中／下一步三項 |
| Reply | `### Reply` | 該輪指令的收尾回報，MUST 至少含 `#### SUMMARY` 節 |

編號 `A-###`、`RUN-###`、`WIP-###` 各自獨立遞增（全檔掃描現有最大值 +1）；Reply 不編號（隸屬所在回合）。

## 完整範例

`.zettel/logs/inbox/spaced-repetition.md`（預設佈局下對應 `inbox/spaced-repetition.md`）。此檔歷經三次指令執行：第一次建檔後中斷（A-001 停在 WIP-001，成為開放回合）；第二次開工偵測到開放回合，寫 RUN-002 恢復點後在 A-001 內接續並以 Reply 收束；第三次是新請求，開新回合 A-002：

```markdown
# devlog: inbox/spaced-repetition.md

## STATUS

- Note: inbox/spaced-repetition.md
- Last round: A-002（2026-09-08，/zk:fleeting）
- Proven: 想法已收斂為「間隔重複的效益來自提取難度而非重複次數」；draft 已經使用者確認寫回；反例檢驗（過難提取）已補入本文。
- Open: 「難度甜蜜點怎麼量化」尚未展開。
- Next: 使用者考慮補讀 Bjork 的 desirable difficulties 原文後再議。

## A-001

> /zk:fleeting 我覺得 anki 有用是因為它逼我想起來，不是因為它讓我看很多次

### RUN-001

蘇格拉底對話進行：釐清「逼我想起來」＝主動提取；證據線追問後使用者引述課堂經驗。

### WIP-001

- 已完成：想法傾倒與釐清、假設檢驗一輪。
- 進行中：推論線（如果提取是關鍵，重複次數的邊際效益遞減）。
- 下一步：收斂一句話陳述，出 draft。

### RUN-002

自 WIP-001 下一步恢復（開放回合接續）：收斂陳述並產出 draft，經使用者確認後寫回 inbox/spaced-repetition.md。

### Reply

#### SUMMARY

想法收斂為單句主張並寫回筆記；`updated` 已同步。未展開的量化問題記入 STATUS Open。

## A-002

> 繼續發展：如果提取太難、每次都想不起來呢

### RUN-003

反例檢驗：對話確立「難度有甜蜜點」但書，經使用者確認補入本文。

### Reply

#### SUMMARY

補入過難提取的但書一段；量化問題仍未展開，維持在 STATUS Open。
```
