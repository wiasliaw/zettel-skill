---
name: init
description: 於 vault root 以訪談初始化 Obsidian vault 的 Zettelkasten 工作環境：目錄佈局、templates 播種、tagging convention、qmd 設定與 .zettel.json。對已初始化的 vault 重跑只補缺與遷移。
disable-model-invocation: true
---

# /zk:init

本指令不觸碰日誌、不執行任何 git 指令、不修改 plugin 檔案；一切落檔動作皆在步驟 4 的確認閘核准之後。

## 流程

1. **確認執行位置**：以 `pwd` 呈現當前目錄，請使用者確認即 vault root（CWD 即 vault root 是本指令的硬前置）；否決則停止並請使用者於 vault root 重新執行。

2. **盤點既有狀態**：逐項檢查並記下存在與否，據以決定全新初始化或補缺遷移：
   - `.zettel.json`——存在則讀取，後續盤點以其各 step 的 `place`／`templateFile` 為準；不存在則以出廠預設路徑盤點（`inbox/`、`literature/`、`zettelkasten/`、`reference/`、`_template/*.md`）。
   - config 所指（或出廠預設）的各 step 目錄與模板檔。
   - `.zettel/`（`conventions/`、`proposals/`、`logs/`）與 `.qmd/`。
   - 依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md` 執行 conventions 開工掃描協定（既有慣例影響後續訪談預設值的呈現）與 proposals 順帶掃描協定；兩者各自獨立，一邊為空或不存在 MUST NOT 跳過另一邊。掃描的回報註記與待議提案提示併入步驟 6。

3. **訪談**：逐項取得下列決策輸入，每項綁可觀察訊號、MUST NOT 臆測代答；使用者的既有 vault 規劃優先，MUST NOT 假定預設目錄名適用。訪談全程有效：發現 config 欄位與 3b tag 方案皆未涵蓋的使用者慣例訊號（糾正、重複偏好、做事習慣）時先記下，列入步驟 4 計畫、於步驟 5f 落檔為 proposal，MUST NOT 於確認閘前寫入（初始播種本身不另建 proposal——出廠播種不是晉升）：

   a. **各 step 佈局**：逐 step（fleeting、literature、permanent、reference）確認 `place` 與 `templateFile`（reference 無 `templateFile`）。呈現出廠預設值供調整；步驟 2 盤點到的既有目錄為訪談的候選答案。

   b. **tag 方案**：呈現出廠 kind-tag 方案（`${CLAUDE_PLUGIN_ROOT}/templates/tagging.md` 內容，place 代入 3a 定案值），與使用者定出各 step 筆記如何上 tags。無論定案內容為何，MUST 滿足三條最小不變量：staging tags 可唯一判定 step；filed tags 可判定 step 與 topic；不把一種 kind 改寫為另一種。

   c. **qmd 啟用與否**：先偵測 CLI：

      ```bash
      command -v qmd
      ```

      - 查無 → `qmd.enabled: false`，跳過 3f 與 5e（檢索走 Grep 降級，不視為錯誤、不引導安裝）。
      - 有 → 詢問使用者是否啟用；否 → 同上 `enabled: false`；是 → 續 3f。

   d. **存量筆記（告知，非決策）**：既有筆記 MUST NOT 被本指令搬移或改寫；向使用者說明兩層模型的歸屬方式——staging 目錄的筆記日後由 lifecycle 指令處理，filed 目錄的筆記進索引與連結範圍。

   e. **初始 topic 分區（optional）**：可留空，由 `/zk:permanent` 首次歸檔時自然建立。

   f. **qmd-config 訪談段**（僅 3c 定為啟用時）：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/qmd-config.md` 訪談段執行——探測環境、定索引範圍（collections 必含 `permanent.place` 與 `reference.place`、必不含 staging 兩 place）、四條 operation args 字串、context；只產出決策，不落任何檔案。

4. **scaffold 計畫確認閘**：在對話呈現完整計畫，經使用者確認才進入步驟 5；確認前不寫任何檔案，否決則不寫入。計畫 MUST 含：
   - 將建立的目錄清單（step 目錄、`.zettel/` 三子目錄、啟用 qmd 時的 `.qmd/`）。
   - 將寫入的檔案與內容摘要：各 `templateFile` 的播種來源、tagging convention 全文、config `.zettel.json` 全文、qmd 設定值（`.qmd/index.yml` 全文與四條 args）。
   - 將跳過的既有檔案清單（不覆寫）。
   - 訪談中記下的使用者慣例訊號將寫入的 proposal 檔清單（無則不列此項）。

5. **落檔**（確認後依序）：

   a. 建立 3a 定案的各 step `place` 目錄與 `.zettel/conventions/`、`.zettel/proposals/`、`.zettel/logs/`（`mkdir -p`，已存在者跳過）。

   b. 從 `${CLAUDE_PLUGIN_ROOT}/templates/` 播種 `fleeting.md`、`literature.md`、`permanent.md` 至各 step 的 `templateFile` 路徑（模板內 `{{placeholder}}` 原樣保留，建檔時才代換）；既有同名檔 MUST NOT 覆寫，列入收尾回報。

   c. 依 3b 定案把 tag 方案寫入 `.zettel/conventions/tagging.md`（自 `${CLAUDE_PLUGIN_ROOT}/templates/tagging.md` 調整）；此播種是「proposals 為 convention 晉升唯一入口」的明定例外——出廠播種不是晉升。既有同名 convention MUST NOT 覆寫。

   d. 產出 `.zettel.json`：`version` 填當下 plugin 版號（讀 `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` 的 `version`），四個 step 鍵與 `qmd` 鍵填訪談定案值，恰六個頂層鍵、不多不少。完整格式範例（step 與 `qmd` 的值為出廠預設，均以訪談定案值為準；`version` 為示意值）：

      ```json
      {
        "version": "0.1.0",
        "fleeting":   { "place": "inbox",        "templateFile": "_template/fleeting.md" },
        "literature": { "place": "literature",   "templateFile": "_template/literature.md" },
        "permanent":  { "place": "zettelkasten", "templateFile": "_template/permanent.md" },
        "reference":  { "place": "reference" },
        "qmd":        { "enabled": true, "query": "", "search": "", "update": "", "embed": "" }
      }
      ```

      `reference` 無 `templateFile`（reference 筆記是 `/zk:permanent` 歸檔的產物，不從模板建立）。`qmd.enabled` 的語意是「qmd CLI 已安裝**且**使用者允許使用」；`qmd` 其餘四鍵為各 operation 的 CLI args 字串（passthrough，呼叫時原樣拼接，缺省空字串）。topic 清單由目錄結構即時推導，MUST NOT 寫入 config。

   e. **qmd 落地**（僅啟用時）：依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/qmd-config.md` 落地段執行——寫 `.qmd/index.yml`、驗證 project-local 索引路徑、`qmd update && qmd embed` 建初始索引、`qmd doctor` 驗證。

   f. 依步驟 3 記下的使用者慣例訊號，逐項依 `${CLAUDE_PLUGIN_ROOT}/skills/internals/memory.md`「proposal 寫入操作」寫入 proposal 檔；無記下訊號則跳過。

6. **收尾回報**：白話列出——建立了什麼、播種了什麼、跳過了哪些既有檔案（原因：不覆寫）、config 落定值、qmd 索引狀態（或「未啟用，檢索走 Grep 降級」）、寫入的 proposal 檔與步驟 2 掃描的待議提案提示（如有）。

## 冪等重跑

對已初始化的 vault 重跑時 MUST 只補缺與遷移：

- 逐項盤點（步驟 2），缺什麼補什麼（走同一條訪談→確認閘→落檔，僅涵蓋缺項）。
- `.zettel.json` 就地更新 `version` 至當下 plugin 版號、補齊缺鍵；這是唯一可就地更新的既有檔案。
- 其餘任何既有檔案 MUST NOT 覆寫、MUST NOT 觸碰筆記內容。
- 使用者要啟停 qmd 時亦走此路徑（更新 config `qmd` 鍵；啟用時補跑 qmd-config）。
