[English](README.md) | 繁體中文

# zettel-skill

一個 Claude Code plugin marketplace，讓你在任何 Obsidian vault 裡跑一套 Zettelkasten（卡片盒筆記法）工作流。

## 這是什麼

`zettel-skill` 是一個 Claude Code plugin marketplace monorepo——單一 repo 可容納多個 plugin，各自放在 `plugins/<name>/` 下。目前只有一個 plugin：**zk**。

`zk` 是通用化的 Zettelkasten 工作流：指向任何 Obsidian vault，執行一次 `/zk:init`，該 vault 就會產生自己的設定與模板，供後續指令運作。筆記生命週期有兩個平行的入口——快速記錄想法、與精讀資料來源——兩者最終都匯入一個全自動的拆分歸檔步驟，另外還有一個唯讀的檢索指令可以查詢已歸檔的內容。除此之外還有一個獨立的記憶迴圈，讓 plugin 提出行為上的建議，讓你決定要晉升為固定慣例還是駁回。

## 前置需求

- Claude Code
- 一個 Obsidian vault
- [qmd](https://github.com/tobi/qmd)（選用）——若未安裝或未啟用，檢索會降級為純文字搜尋（Grep），而不是走語意搜尋。這種降級是預期行為，不是錯誤。

## 安裝

1. 在 Claude Code 中執行：
   ```
   /plugin marketplace add wiasliaw/zettel-skill
   ```
2. 接著安裝 plugin：
   ```
   /plugin install zk@zettel-skill
   ```
3. 在你的 Obsidian vault 根目錄執行 `/zk:init` 進行設定。

## 指令

| 指令 | 作用 |
|---|---|
| `/zk:init` | 對你進行訪談以了解 vault 環境，提出 scaffold（骨架）落檔計畫——待你核准後，才寫入 step 目錄、筆記模板、tagging convention（標籤慣例）、設定檔（`.zettel.json`）與 qmd 設定。 |
| `/zk:fleeting <path\|文字>` | 扮演蘇格拉底式的思考夥伴，協助你把一個初步想法發展成草稿，並在存檔前讓你審閱（draft-then-review）。 |
| `/zk:literature <path\|source>` | 精讀一份資料來源，透過問答釐清內容，寫成筆記，並對完稿進行對抗式審查（adversarial review）。 |
| `/zk:permanent <path>` | 全自動：把草稿拆分成一則則原子筆記、在筆記之間加入行內連結、將原始來源筆記歸檔進你的 reference 資料庫、更新檢索索引、驗證結果，並回報執行內容。 |
| `/zk:query <query>` | 唯讀檢索：根據已歸檔的內容回答你的問題，透過一跳漫遊（one-hop）帶出一則相關筆記作為意外發現，並回報它注意到的知識缺口。 |
| `/zk:adapt` | 逐條檢視待裁定的行為建議，讓你決定晉升為固定慣例，或是駁回刪除。 |

`/zk:init`、`/zk:fleeting`、`/zk:literature`、`/zk:permanent`、`/zk:adapt` 只在你明示呼叫時才會執行。`/zk:query` 為唯讀，也可能由情境自動觸發。

## 文件

- [使用教學](docs/zh-tw/tutorial.md)——完整工作流的實作走一遍
- [設計說明](docs/zh-tw/design.md)——plugin 的架構與設計理由
- [工作流程](docs/zh-tw/workflow.md)——日常使用的操作模式

## 授權

Apache-2.0
