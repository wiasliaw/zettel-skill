# Intent 掃描與離題捕獲

寫入型指令開工的 active intents 掃描協定，與工作全程有效的離題捕獲通道。本檔只定義共通協定；各指令的應用面（提問方向、精讀焦點、審查脈絡、計畫取捨）住各引用方流程。

## 掃描協定（開工執行）

讀 `.zettel/intents/` 下全部 `status: active` 檔案的全文，作為本次工作的用域約束。無 active intent（目錄不存在、目錄為空、或全為 `archived`）時不構成約束、照常執行，MUST NOT 因此停止或要求使用者先建 intent。

intent 檔案的新增、修改與封存一律由使用者發起，MUST NOT 代為增刪或改寫 `.zettel/intents/` 下任何檔案。

## 離題捕獲（全程有效）

工作中出現 active intents 之外的發現時，MUST 提供低成本通道——直接寫成 fleeting 筆記進 staging，不經對抗式審查：活讀 config `fleeting.templateFile` 所指模板建檔於 `fleeting.place`，tags 依 tagging convention，basename 查名唯一，`version` 活讀 config 現值（建檔操作見同目錄 `note-format.md`）。MUST NOT 因 intent 約束而靜默丟棄離題發現，MUST NOT 以離題為由拒絕記錄。
