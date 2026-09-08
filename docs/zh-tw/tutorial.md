# 使用教學

本篇是給第一次使用 **zk** plugin 的人的完整走一遍：要裝什麼、如何初始化 vault，以及日常指令實際互動起來是什麼樣子——每個指令會問你什麼、會寫下什麼檔案、產出的筆記長什麼樣。

## 1. 前置需求與安裝

你需要：

- **Claude Code**，且支援 plugin。
- 一個 **Obsidian vault**（新舊皆可）——不限佈局，`/zk:init` 會順應既有規劃，不會強加預設目錄名。
- **qmd**（optional）——一套命令列語意檢索（semantic search）工具。未安裝，或你選擇不啟用，檢索就會走純文字搜尋的 Grep 降級（Grep fallback），這是預期行為而非錯誤，plugin 也不會因此提示你去安裝。

安裝 plugin：

```
/plugin marketplace add wiasliaw/zettel-skill
/plugin install zk@zettel-skill
```

接著在你的 Obsidian vault root 執行 `/zk:init` 完成設定。以下內容都假設這一步已完成。

## 2. 初始化 vault：`/zk:init`

`/zk:init` 必須在 vault root 目錄下執行——它一開始會用 `pwd` 顯示當前目錄，請你確認這正是 vault root。接著它會盤點既有狀態（既有的 `.zettel.json`、其所指的各目錄、`.zettel/`、`.qmd/`），據此判斷這是全新初始化還是補缺遷移。

### 訪談問什麼

`/zk:init` 逐項向你確認五件事，每一項都綁定可觀察的事實，不會替你臆測：

1. **各階段的佈局。** 針對 fleeting、literature、permanent、reference 四個階段，逐一確認其筆記存放位置（`place`），以及 fleeting／literature／permanent 三者建檔所用的模板檔（`templateFile`）。reference 沒有模板——reference 筆記從來不是直接建立的，只會是 `/zk:permanent` 歸檔時的產物。出廠預設是 `inbox`、`literature`、`zettelkasten`、`reference` 與 `_template/*.md`，但你既有的 vault 規劃優先，不會假定預設目錄名適用。

2. **tag 方案。** 你會看到出廠的 kind-tag 方案（見下方），可依需求調整，唯一限制是結果必須滿足三條最小不變量：暫存層（staging）筆記的 tags 必須能唯一判定它屬於哪個階段；歸檔層（filed）筆記的 tags 必須能判定其階段與 topic；任何流程都不可把一種 kind 改寫成另一種。

3. **要不要啟用 qmd。** 訪談會先執行 `command -v qmd` 偵測。查無此指令時，檢索直接定為 Grep 降級，qmd 相關問題全部跳過；偵測到時才詢問你是否啟用，回答「否」同樣落在 Grep 降級。

   若選擇啟用，還會進入一段子訪談：探測你的執行環境（embed 模型、運算後端，作為後續決策的參考資訊）；決定索引範圍（qmd 該索引哪些資料夾，見下方）；決定 qmd 四個 operation（`query`、`search`、`update`、`embed`）各自的 CLI 參數字串（無偏好可留空）；以及一段簡短的 vault 說明文字，加上——若你想指定——特定 topic 分區的說明。這一步都只是決策收集，還不落任何檔案，會併入下一步的計畫呈現。

4. **既有筆記怎麼處理。** 完全不會被搬動或改寫。`/zk:init` 只是說明兩層筆記模型，讓你了解既有筆記日後的概念歸屬：位於暫存層目錄（fleeting／literature）的筆記，之後會由日常指令接手處理；已經在歸檔層目錄（permanent／reference）的筆記，則直接視為卡片盒（slip-box）的一部分，可被索引與連結。

5. **初始 topic 分區。** Optional，可以留空，讓 `/zk:permanent` 第一次歸檔時自然長出 topic 目錄。

### scaffold 計畫確認閘

在寫入任何檔案之前，`/zk:init` 會在對話中呈現完整計畫：即將建立的每個目錄、`.zettel.json` 的完整內容、tagging convention 的完整文字，以及（若啟用 qmd）qmd 設定值與 `.qmd/index.yml` 的內容。經你核准前不會落任何檔；否決的話就什麼都不寫。

### 落檔產物

核准後會依序落檔：

- 你訪談中定案的各階段目錄，加上 `.zettel/conventions/`、`.zettel/proposals/`、`.zettel/logs/`。
- 筆記模板，從 plugin 的出廠模板播種到各階段的 `templateFile` 路徑；若該路徑已有同名檔案，絕不覆寫，只會列入收尾回報。
- tagging convention，寫入 `.zettel/conventions/tagging.md`。這是「convention 只能經由 proposal 核准晉升」規則唯一的內建例外——出廠播種本身不算一次晉升。
- `.zettel.json` 本身。以下是出廠預設的完整結構（各值即你訪談中定案的內容；`version` 會填入你當下安裝的 plugin 版本號）：

  ```json
  {
    "version": "0.2.1",
    "fleeting":   { "place": "inbox",        "templateFile": "_template/fleeting.md" },
    "literature": { "place": "literature",   "templateFile": "_template/literature.md" },
    "permanent":  { "place": "zettelkasten", "templateFile": "_template/permanent.md" },
    "reference":  { "place": "reference" },
    "qmd":        { "enabled": true, "query": "", "search": "", "update": "", "embed": "" }
  }
  ```

  注意 `reference` 沒有 `templateFile`；`qmd.enabled` 的意思是「qmd 已安裝**且**你選擇使用它」——之後每個指令都只靠這一個欄位判斷該走語意檢索（semantic retrieval）還是 Grep 降級。

- 若你啟用了 qmd：`.qmd/index.yml`，記載 qmd 要索引哪些資料夾（一定含 `permanent.place` 與 `reference.place`，可視需要再加，但絕不含兩個暫存層目錄——暫存層筆記從不進索引），接著執行一次 `qmd update && qmd embed` 建立初始索引，並用 `qmd doctor` 驗證健康狀態。

### 重跑 `/zk:init`

對已初始化的 vault 重跑 `/zk:init`，只會補缺與遷移——更新 `.zettel.json` 的 `version` 欄位、補齊缺項，絕不覆寫既有檔案、也絕不觸碰筆記內容。事後想開啟或關閉 qmd，同樣是重跑這個指令、在 qmd 那題給出不同答案即可。

## 3. 日常流程

`/zk:fleeting`（發展一則想法）與 `/zk:literature`（研讀一個來源）是兩個平行的入口，負責把新素材帶進 vault；兩者都匯入 `/zk:permanent`——這是草稿變成可連結、永久保存的卡片盒（slip-box）內容唯一的路徑。`/zk:query` 唯讀地檢索這個卡片盒，不改動任何東西。`/zk:adapt` 則是另一條迴圈，用來裁定 plugin 觀察到、關於你工作習慣的提案（proposal）。

每個寫入型指令開工都先過同兩項前置：確認 `.zettel.json` 存在（不存在就停下並請你先跑 `/zk:init`——它絕不會代你建立 config），以及若啟用 qmd 就重新索引一次（qmd 未啟用或索引已是最新時，這步等於沒做）。每次開工也都會掃描你的 conventions 與待議 proposals；`/zk:fleeting` 與 `/zk:literature` 另外會為該筆記讀取或建立一份**工作日誌（devlog）**，存放於 `.zettel/logs/`（詳見第 4 節）。有了它，發展或研讀某篇筆記的過程即使被打斷，回頭再跑指令也能從中斷處接續。

### `/zk:fleeting`——發展一則想法

**角色：** 蘇格拉底式提問（Socratic questioning）的思考夥伴。**寫入範圍：** 你的 `fleeting.place`。**互動閘門：** draft-then-review（先出草稿、經你確認才寫入）。

呼叫時可帶既有 fleeting 筆記的路徑，也可以直接帶一段原始想法文字：

```
/zk:fleeting 我覺得間隔重複有效是因為它逼我主動回想，不是因為重複次數本身
```

```
/zk:fleeting inbox/spaced-repetition.md
```

它會先讓你把想法倒出來，再用 Paul 與 Elder 的六類問題交叉詰問——釐清、假設、證據、觀點、推論、後設（這件事為什麼重要）——但不是機械式按固定順序覆蓋六類：每一輪只挑當下看起來最薄弱的一條線索追問，預設立場是挑戰而非附和。停止條件有二：想法已收斂成一句話陳述，或對話中浮現需要處理的矛盾。

你準備好時，它會呈現完整修訂後的全文，等你確認才寫入（draft-then-review）——否決就什麼都不存。確認後：既有筆記就地修訂（同時更新 `updated` 時間戳）；純文字輸入則依 `fleeting.templateFile` 建立新筆記，`version` 即時讀自 `.zettel.json`，tags 依你的 tagging convention 設定，檔名並會核對全 vault 唯一性。

收尾回報會告訴你寫了什麼、想法收斂到哪個結論，以及 devlog 狀態，還有沿路注意到的相關 conventions 或待議 proposals。

### `/zk:literature`——研讀一個來源

**角色：** 先精讀來源，接著問答，再成稿，成稿後立刻對抗式審查（adversarial review）那份草稿。**寫入範圍：** 你的 `literature.place`。**互動閘門：** 建檔確認閘，加上成稿階段的 draft-then-review（只有你明確要求「直接寫入」才會跳過）。

呼叫時帶一個 source（URL 或書目出處），或帶既有 literature 筆記的路徑：

```
/zk:literature https://example.com/some-paper
```

```
/zk:literature literature/some-paper.md
```

每篇 literature 筆記恰有一個 source，記在 frontmatter 的 `source` 欄，不能為空——它是精讀階段唯一的錨點。如果你要求對一篇已有 source 的筆記「再加一個來源」，它不會合併進去，而是另建一篇獨立筆記；日後從這兩篇各自拆出的原子筆記（atomic note），彼此之間再用內連結（inline wikilink）串起來。

對於真正的新來源，除非你的指示已經明確表達了建檔意圖，否則它會先呈現建檔計畫（擬用的檔名、標題、source 原文、已知的閱讀聚焦）並等你確認，才會真的建檔。

筆記建好後，精讀階段會派工（dispatch）一個唯讀的 research subagent，嚴格錨定在該篇的 `source` 上。如果你已表明閱讀聚焦（無論是寫在筆記的 Intent 節，還是在對話中說明），agent 會在聚焦範圍內做到 full-coverage（讀者不需回頭讀原文），範圍外則以結構性摘要帶過；沒有指明聚焦時就對整份來源做 full-coverage。關鍵是：如果這個 agent 無法取得來源全文，它會直接中止並回報原因——絕不會退而求其次去用二手摘要或自己的訓練知識來充數。

接著你可以提問，回答會錨定在 source 與精讀成果上；若引用了來源之外的知識，一定會明確標示。當你給出收尾信號（說「寫成筆記」或同義的話）時，對話內容會被綜合寫入本文，同時依實際聊到的內容填寫或更新 Intent 節。預設會先給你看草稿、等你確認才寫入；只有你明確說「直接寫入」之類的話，才會跳過確認直接落檔。

草稿一寫入，同一次指令執行內立刻會有一個對抗式審查（adversarial review）subagent 對它進行審查——不會拖到最後才做，也不會被省略。這個 agent 的預設立場是不信任：把本文拆成一條條主張，逐條嘗試推翻，推翻不了才判定忠實。每條主張最終恰好歸入「忠實」「事實錯誤」「存疑」三者之一。它自己絕不動筆修改任何檔案——只回報，對「事實錯誤」的項目附上建議修正與引出的原文依據。接下來由你逐條裁定要不要採用建議；`updated` 欄位會在你套用修正的那一次編輯統一更新。如果審查發現某個存疑點值得再深入讀，它可能建議另開一篇新的 literature 筆記處理——但這只是提議，要你核准才會真的建檔，絕不會自動執行。

研讀過程中，你也可以隨時要求審查（說「幫我審一遍」）而不出草稿——它會針對筆記當下的全文進行審查，並只在對話中回報。

收尾回報一律完整帶出審查報告的全部細節——總條數、各分類的統計、每一條非忠實主張的文字與判定依據——即使 `/zk:literature` 是被包在一個更大的請求裡呼叫，也不會被壓縮成一行摘要。

### `/zk:permanent`——拆分與歸檔

**角色：** 全自動。**會動到：** 新的 permanent 筆記、源筆記的位置、索引、它自己的 devlog。**互動閘門：** 沒有——刻意設計為零確認。

```
/zk:permanent inbox/spaced-repetition.md
```

輸入必須是一篇既有的暫存層筆記——目前位於你的 `fleeting.place` 或 `literature.place` 下。（歸檔層路徑只用來偵測並續跑一次被中斷的執行，絕不能當成新的拆分輸入。）

一旦開始，就會不停下來問你任何問題，依序跑完：

1. **拆分。** 一個 subagent 讀完源筆記全文，完全自主判斷把它拆成一概念一檔的原子筆記（atomic note），為每篇選定一個 topic（「top tag」）——優先套用既有 topic 目錄，真的沒有合適的才新增分區——並在拆出的筆記之間，凡本文行文確實提及另一篇概念的地方，加上內連結（inline wikilink）。
2. **歸檔源筆記。** 移入 `reference.place/<topic>/`——絕不刪除——`<topic>` 取這批原子筆記多數落在哪個分區。它的 tags 會正規化為歸檔層方案（kind 加 topic），但絕不改變它原本是哪種 kind。
3. **刪除 devlog。** 歸檔一旦成功，這篇源筆記在 `.zettel/logs/` 下的日誌就會被刪除——這是 `/zk:permanent` 對 devlog 唯一會做的事。
4. **重建索引。** 若啟用 qmd，會執行 `qmd update && qmd embed`，讓新的原子筆記與剛歸檔的源筆記變得可被檢索到；若 qmd 未啟用，這步直接跳過，因為沒有索引需要維護。
5. **驗證。** 另一個對抗式審查 subagent 會核對每篇新原子筆記對源筆記（此時已在新位置）的忠實度，也逐條驗證每個內連結是否真的連到行文有支撐的地方——同樣只回報不修改，由你決定要採用哪些建議修正。

如果源筆記裡每個概念的素材都不足以獨立成篇，上面這一切都不會發生——源筆記與其 devlog 原地保留不動，回報只會逐一說明每個概念為何被略過（你可能會想回到 `/zk:fleeting` 或 `/zk:literature` 補充素材）。

因為這是無人看管跑完的流程，它也設計成能扛得住中斷：萬一跑到一半停了（比方拆分完但還沒歸檔，或歸檔完但還沒刪日誌），之後再跑一次能準確偵測並從正確的那一步接續，不會重做已完成的工作，也不會拆出重複的筆記；它絕不會對已經拆過的源筆記再拆一次。

收尾回報一律並列呈現兩份完整內容：歸檔結果（每篇新筆記的檔名與 topic、是否新增了分區、每個內連結及理由、源筆記最終去向、任何被略過的概念、devlog 與重建索引的狀態），以及驗證報告（各分類的主張總數、每條非忠實主張的判定依據、每個連結通過與否、以及對每項建議修正——是否真的被採用）。

### `/zk:query`——向卡片盒提問

**角色：** 唯讀。絕不寫入任何檔案、不更新索引、不觸碰任何 devlog。

```
/zk:query 我寫過哪些關於「刻意提高學習難度」的筆記？
```

它只搜尋歸檔層——`permanent.place` 與 `reference.place`——啟用 qmd 時走語意檢索（semantic retrieval），否則走 Grep 降級，並且無論哪種都會額外加一輪精確詞 Grep，補上語意檢索對識別碼、專有名詞等字面 token 較弱的部分。作答前，它會讀完答案實際會依賴的那 2 到 5 篇候選全文——單靠相似度分數或命中次數，絕不足以構成引用的理由。

答案會用 `[[note]]` 引用它實際讀過的筆記。接著它會回報一則**一跳漫遊（serendipity）**發現：從它讀過全文的那些筆記裡收集出鏈，排掉本來就在檢索結果裡的目標，讀剩下的最多五篇，回報其中與查詢主題有實質但非顯而易見關聯的部分——若什麼都沒找到，會明寫「無」，而不是省略這一節。最後它會回報**知識缺口**：查詢觸及卻沒能完整回答的地方，寫成「這可以是下一篇筆記的方向」，而不是判定「這個查詢無法回答」。

### `/zk:adapt`——裁定提案

**角色：** 一次一條的對話。**寫入範圍：** 僅 `.zettel/conventions/` 與 `.zettel/proposals/`。

```
/zk:adapt
```

在日常使用中，其他指令有時會注意到一些關於你工作方式、值得記住的訊號——你糾正了兩次的東西、你表達過的偏好——並把它寫成一份**提案（proposal）**檔。`/zk:adapt` 就是用來處理這些提案的地方：它會讀完每一份待議提案的全文（包含其 User review 節裡已有的回饋），逐條向你呈現討論，絕不批次帶過、也絕不代你決定。每一條你恰要二擇一：

- **駁回**——刪除該提案檔；刪除即封存，沒有另外的「已駁回」資料夾。
- **晉升為慣例（convention）**——建立或就地更新 `.zettel/conventions/` 下對應主題的檔案，一主題一檔，每檔 frontmatter 帶一句話的觸發條件。

在真正動筆寫入或刪除之前，它會再重讀一次該提案與相關 convention，以防兩者自你上次看到後已經變動——絕不會依據過期的版本套用裁定。

晉升的 convention 從下一次寫入型指令開工掃描起生效，不會回頭套用到已經寫好的內容上。

## 4. Vault 側產物與心智模型

### 暫存層 vs. 歸檔層

每篇筆記恰處於兩層之一，各層對應的目錄就是你在 `.zettel.json` 裡設定的值——不假定它們一定叫 `inbox`、`literature`、`zettelkasten` 或 `reference`。

- **暫存層（staging）**（`fleeting.place`、`literature.place`）：扁平存放、不分區。暫存層筆記從不進 qmd 索引，也從不能被當成連結目標——vault 裡任何地方都不允許 `[[連結]]` 指向一篇暫存層筆記。
- **歸檔層（filed）**（`permanent.place/<topic>/`、`reference.place/<topic>/`）：依 topic 分區存放。歸檔層筆記會被索引（啟用 qmd 時）、也可以被連結。

歸檔——暫存層變成歸檔層——是筆記唯一會發生的搬遷方向，絕不會逆向。

### `.zettel/` 裡面是什麼

- **`conventions/`**——你的長期偏好，一主題一檔，每檔帶一句話的觸發條件。每個寫入型指令開工都會讀這裡。
- **`proposals/`**——等待你在 `/zk:adapt` 裡裁定的提案。它們的存在只會被提及，絕不會被自動採用。
- **`logs/`**——每篇暫存層筆記各有一份工作日誌（devlog），路徑鏡射該筆記本身（例如 `inbox/foo.md` 的日誌就在 `.zettel/logs/inbox/foo.md`）。devlog 記錄這篇筆記跨 session 發生的事——最初的請求、關鍵事件、目前狀態——好讓 `/zk:fleeting` 與 `/zk:literature` 在被打斷後能準確接續。這篇筆記一旦被 `/zk:permanent` 成功歸檔，devlog 就會立刻被刪除；歸檔層筆記從來沒有 devlog。

### 模板檔是 schema 的權威

一篇筆記該有哪些 frontmatter 欄位，權威定義是該階段 `templateFile` 所指的模板檔內容——不是 plugin 裡任何硬編碼的東西。你可以自行編輯這些模板檔來增減欄位，plugin 會讀取你調整後的版本。任何超出內建欄位的自訂欄位都會被當作不透明內容保留：沒有任何指令會剝除或改寫它不認識的 frontmatter 欄位（例如你拿來給 Obsidian Base 用的自訂屬性）。

### plugin 絕不碰的東西

plugin 從不執行任何 git 指令——你的 vault 版本控制完全由你自己管理，它一律把當前 working tree 視為最新事實。它也絕不修改自己已安裝的檔案；本篇描述的每個行為都是 plugin 安裝後原樣的行為，任何行為變動只會透過 plugin 更新發生。

## 5. 常見疑問

**如果我一直不開 qmd 會怎樣？** 不會壞。檢索會退回 Grep 純文字搜尋歸檔層筆記，並另加一輪精確詞比對。速度較慢、也較不語意化，但這是完整受支援的模式，不是降級錯誤——plugin 也不會因此提示你去裝 qmd。

**我手動改名或搬移了一篇暫存層筆記，它的 devlog 會怎樣？** devlog 完全靠鏡射筆記路徑來找到，若你在 zk 指令之外改名或搬移筆記，鏡射關係就斷了。下次對這篇筆記在新路徑上跑日常指令時，會找不到對應的 devlog，於是視為新建——之前的 session 歷程無法復原。這是已知且可接受的限制，不是一個需要回報的錯誤。

**對已經初始化過的 vault 重跑 `/zk:init` 安全嗎？** 安全。重跑只會補齊缺項、更新記錄的 plugin 版本號，絕不覆寫既有檔案、也絕不觸碰筆記內容。事後想開啟或關閉 qmd，也是走這條路徑。
