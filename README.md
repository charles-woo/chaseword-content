# 字洞洞 內容包（唔使更新 App 都可以加聖誕節之類嘅活動）

呢個 `content/` 資料夾就係會放上 GitHub Pages 嘅嘢。App 每次開都會：

1. 先用部機上次存低、已經檢查過嘅內容（冇網都得）；
2. 如果 App 入面自帶嘅內容（build App 嗰陣 `content/` 嘅版本）比較新，就用自帶嗰份；
3. 然後喺背後靜靜雞去你個 GitHub Pages 睇 `manifest.json`。版本號大過部機嗰份，先會逐個檔案下載，
   對 sha256、對大小、檢查晒格式，**全部啱晒**先會一次過換新。

任何一步出錯（冇網、404、檔案壞咗、hash 唔對、太大、格式錯、App 版本太舊），App 都會繼續用舊嘅內容，
小朋友乜都唔會見到，遊戲照玩。App 永遠唔會等網絡先開始玩。

## 資料夾裏面有乜

```
content/
  manifest.json          由 build 程式自動寫，唔好手改
  .nojekyll              俾 GitHub Pages 用，唔好刪
  packs/xmas-2026.json   一個「內容包」：聖誕活動、聖誕小鎮島、聖誕帽、馴鹿、聖誕字表
  packs/_draft.json      檔名 _ 開頭 = 草稿，唔會發佈
  voice/...              （之後放錄音檔，見最後一節）
  README.md              就係呢份
```

## 第一次設定（只做一次）

1. 喺 GitHub 開一個新 repo，例如 `chaseword-content`（public）。
2. 將成個 `content/` 入面嘅嘢（`manifest.json`、`.nojekyll`、`packs/`）放落去個 repo 嘅最頂層，commit + push。
3. 個 repo 嘅 Settings → Pages → Source 揀 "Deploy from a branch"，branch 揀 `main`，folder 揀 `/ (root)`，Save。
4. 等一兩分鐘，個網址會係 `https://你個用戶名.github.io/chaseword-content/`。用瀏覽器開
   `https://你個用戶名.github.io/chaseword-content/manifest.json`，見到內容就得。
5. 喺 App 專案嘅 `.env.local` 加一行（**結尾要有 /**）：
   `VITE_CONTENT_BASE=https://你個用戶名.github.io/chaseword-content/`
6. 重新 build App，上 App Store / Google Play **一次**。之後所有活動都唔使再更新 App。

注意：網址一定要 `https://`，而且要係最終網址（App 唔會跟 redirect）。如果之後轉用自己嘅 domain，
要再改 `.env.local` 再出一次 App。

## 加一個新活動（例如農曆新年）

1. 複製 `packs/xmas-2026.json`，改名做 `packs/cny-2027.json`。
   檔名只可以用細階英文、數字同 `-`，而且**一定要同檔案入面嘅 `"id"` 一樣**（`"id": "cny-2027"`）。
2. 改 `events` 入面嗰個活動：
   - `"id"`：活動名，例如 `"cny-2027"`
   - `"from"` / `"to"`：開始同結束日子（包埋嗰兩日），格式 `"2027-02-01"`，最長 62 日。用部機自己嘅日期。
   - `"banner"`：大堂頂嗰條橫額。
     - `"zh"`：中文。如果廣東話同普通話寫法唔同，就寫 `["廣東話", "普通話"]`，例如
       `["新年小鎮開咗喇！", "新年小鎮開放了！"]`；一樣嘅就寫一句 `"新年快樂！"`。
     - `"en"`：簡單英文（英國串法，6 歲小朋友睇得明），例如 `"Happy New Year!"`
     - `"icon"`：一個 emoji，例如 `"🧧"`
     - `"bg"`、`"bg2"`：底色（漸變），`"fg"`：字色。全部用 `#rrggbb`，例如 `"#c8102e"`
     - `"snow": true` 會有雪花飄，唔要就寫 `false`
   - `"theme"`：活動個島（下面講點整）。橫額上面會有「去玩 Play」掣，撳咗就玩呢個島，返大堂就變返正常。
   - `"replaces"`（可以唔寫）：例如 `["ice"]` = 活動期間第 4 關（冰雪島）自動變成活動島。普通情況唔建議用。
   - `"quests"`：活動任務（最多 5 個）。`"event"` 只可以係
     `correct`（答啱）、`combo`（連擊）、`eatMonster`（食怪洞）、`levelDone`（玩完一局）、`write`（寫字）、
     `coins`（賺金幣）、`perfect`、`checkpoint`；`"goal"` 1 至 100；`"reward"` 0 至 300 金幣；`"id"` 要 `ev_` 開頭。
   - `"items"`：活動限定嘅帽同寵物（喺 `cosmetics` 入面定義）。
   - `"list"`：活動字表（喺 `lists` 入面定義）。
3. 唔想要嘅部分（例如唔整新島）就成段刪走：`themes`、`cosmetics`、`lists` 都可以冇。
4. 檢查：喺 App 專案度行 `node scripts/content/build.mjs --check`。有錯會話你邊行邊個字錯，改好再試。
5. 喺自己部電腦試玩（見下面「試玩」）。
6. 發佈（見下面「發佈」）。

> 現時嘅 App：活動橫額、活動島、字表資料已經用得。活動任務、帽同寵物會喺下一個版本嘅 App 先出現喺
> 任務同商店度（資料已經準備好，下一版 App 會讀）。

## 整一個活動島（theme）

島嘅顏色同地面：

- `"sky"`、`"fogColor"`（天）、`"sea"`、`"seaDeep"`（下面嘅海）、`"cloud"`、`"gate"`（閘口）、
  `"hazard"` / `"hazardAlt"`（危險棒嘅兩隻色）、`"bridge"`（橋板顏色，2 至 6 個）、`"cliff"`（島邊）
- `"ground"`：`"base"` 地面底色、`"alt"` 斑點色、`"path"` 路、`"pathEdge"` 路邊、`"line"` 路中線、
  `"studHi"` / `"studLo"`（積木凸點嘅光同影，可以用 `rgba(255,255,255,0.8)`）、
  `"pattern"` 地面花紋：`tiles`（城市地磚）、`sprinkles`（糖粒）、`cracks`（熔岩裂紋）、`snow`（雪地）、`plain`；
  `"pathStyle"` 路面：`bars`（朱古力格）、`cobbles`（石仔）、`plain`；
  `"patches"`（斑點顏色，最多 8 個）、`"sprinkles"`（小點顏色，最多 8 個）
- `"music"`：用邊首內置音樂：`city`、`candy`、`lava`、`ice`

### 3D 物件（types）

每樣物件都係用積木咁砌出嚟：盒（box）、圓柱（cylinder）、圓錐（cone）、球（sphere）、多面球（ico）、圈（torus）。

```json
"snowman": { "size": 0.85, "height": 2.6, "points": 6, "parts": [
  { "shape": "sphere", "size": [0.62], "color": "#ffffff", "pos": [0, 0.55, 0] },
  { "shape": "cone", "size": [0.07, 0.38], "color": "#ff8a3d", "pos": [0, 1.95, 0.48], "rot": [90, 0, 0] }
] }
```

- `"size"`：物件佔地嘅半徑（米），決定個洞要幾大先食得落。`"height"`：大約幾高。`"points"`：食咗得幾多分。
- `"kind"`（可以唔寫）：`"vehicle"`（車，會喺路上行）或 `"building"`（屋）。
- 每個 part：
  - `"shape"` 同 `"size"`：
    `box` = `[闊, 高, 深]`；`cylinder` = `[頂半徑, 底半徑, 高]`；`cone` = `[半徑, 高]`；
    `sphere` / `ico` = `[半徑]`；`torus` = `[圈半徑, 管粗]`（可以加 `"arc": 180` 整半個圈，例如拐杖糖個勾）
  - `"color"`：`#rrggbb`
  - `"pos"`：`[x, y, z]` 位置，y 係高度，0 = 地面，z 正數 = 向前（向住鏡頭）。**物件嘅中心喺 pos**，
    所以一個高 1 嘅盒要放喺 y = 0.5 先會企喺地上。
  - `"rot"`：`[x, y, z]` 轉幾多**度**（唔係弧度），例如 `[90, 0, 0]` 令圓錐打橫向前。
  - `"scale"`：`[x, y, z]` 拉扁拉長，例如 `[1, 1.2, 0.5]`
  - `"seg"`：圓柱 / 圓錐 / 圈有幾多邊（3 至 24），4 = 金字塔形屋頂
  - `"ring": { "n": 6, "r": 0.8 }`：喺 pos 周圍圍一圈放 6 個（例如聖誕樹上面嘅波波）

**大細階梯（一定要有，否則 build 程式會唔畀過）**：個洞會由細食到大，所以每個島都要有啱佢大細嘅嘢。

| 用途 | size | points（參考） |
| --- | --- | --- |
| 第一啖（至少一樣） | 0.3 至 0.5 | 2 至 6 |
| 小嘢（至少兩樣 ≤ 0.9） | 0.5 至 0.9 | 6 |
| 第 1 個島中心 | 0.8 至 1.3 | 14 至 18 |
| 第 2 個島 | 1.1 至 2.7 | 18 至 75 |
| 第 3 個島 | 1.75 至 3.8 | 30 至 220 |
| 第 4 個島 | 2.3 至 5 | 45 至 420 |

- `"street"`：路邊擺乜（物件名，至少一樣 size ≤ 1.3）
- `"movers"`：四個島各自喺路上行嘅嘢，例如 `[["reindeer"], ["sleigh"], ["toy_train"], ["toy_train"]]`。
  第 1 個島只可以用 size ≤ 1.3 嘅，第 2 個 ≤ 2.7，第 3 個 ≤ 3.8。
- 想睇下啲物件砌成點：開 dev server 之後去 `http://localhost:5173/scripts/dev/n3-models.html?pack=你個包名`

## 帽同寵物（cosmetics）

- 帽：`"id"` 要 `ev_` 開頭（例如 `"ev_santa_hat"`），`"price"` 1 至 5000 金幣，`"rarity"`：`common` / `rare` / `epic` / `legendary`，
  `"parts"` 同上面物件一樣（原點 = 頭髮頂中間，帽大約闊 1）。
- 寵物：用一隻現有寵物（`"base"`：dog、cat、bunny、chick、panda、penguin、fox、frog、unicorn、dino、robot、dragon）
  換顏色：`"recolor": { "#原本顏色": "#新顏色" }`，可以加 `"parts"`（例如鹿角、紅鼻）。

**活動完咗都唔好刪啲帽同寵物**：有小朋友已經買咗，App 要靠呢啲資料先畫得出。活動可以刪或者改日子，
`cosmetics` 就留低。已經賣出嘅 id 永遠唔好改名。

## 字表（lists）

```json
{ "text": "雪", "jyutping": "syut3", "pinyin": "xuě", "en": "snow", "word": "雪人", "emoji": "⛄", "distractors": ["雨", "雲"] }
```

每個字都要有粵拼（有聲調數字）、拼音（有聲調符號）、英文、詞語。`distractors` = 易混字（可以唔寫，唔可以係個字自己）。
一個字表 4 至 200 個字。

## 試玩（喺自己部電腦）

```
npm run dev
```

然後開：`http://localhost:5173/?content=local&date=2026-12-10&contentdebug`

- `content=local`：用你部電腦嘅 `content/` 資料夾（唔使上 GitHub）
- `date=2026-12-10`：扮今日係呢日，試活動開始、最後一日（`2026-12-26`）、完咗（`2026-12-27`）
- `contentdebug`：右下角會顯示內容版本同錯誤

（呢三個設定只喺 dev 有效，正式 App 唔會理。）

## 發佈

1. `node scripts/content/build.mjs`
   - 檢查晒所有包，有錯就唔會寫任何嘢，話你知錯喺邊。
   - 冇錯就寫 `manifest.json`：版本號自動 +1、每個檔案嘅 sha256 同大小。
   - 如果新內容要新版 App 先用得，加 `--min-app 0.2.0`（舊 App 就會繼續用舊內容）。
2. 將 `manifest.json`、`.nojekyll` 同 `packs/` 抄去 GitHub repo，commit + push。
   **`manifest.json` 同 `packs/` 一定要一齊上**，否則 hash 對唔到，App 會唔要（安全，但新內容唔會出）。
3. GitHub Pages 大約一兩分鐘更新。App 下次開（或者放咗喺背景 6 個鐘之後再開）就會攞新內容。

改錯嘢：改好個包，再行一次 build（版本會再 +1），再上傳。版本號只會加，唔好手動改細。
想即刻收起一個活動：將 `"to"` 改做昨日，build，上傳。

## 安全規則（App 會強制執行，build 程式都會檢查）

- 只可以係純文字：唔可以有 `<` `>`、網址、`www.`、`javascript:` 之類，唔可以放任何連結。
- 顏色只可以係 `#rrggbb`。
- 每個包最多 256 KB；全部加埋最多 1 MB；最多 20 個包；manifest 最多 16 KB。
- 每個島最多 40 樣物件、每樣最多 80 個 part、成個島最多 900 個 part。
- 任務獎勵最多 300 金幣；帽最多 5000 金幣；冇任何真錢嘢。
- 唔好放任何小朋友或者家長嘅個人資料。GitHub 上面嘅嘢係公開嘅。
- App 下載內容唔會帶 cookie、唔會帶任何小朋友資料；GitHub 只會見到部機嘅 IP（同開任何網頁一樣）。

## 錄音檔（之後用）

包入面可以加：

```json
"voice": { "yue": { "雪": "voice/yue/syut3.mp3" }, "cmn": { "雪": "voice/cmn/xue3.mp3" } }
```

錄音檔放喺 `content/voice/...`（只可以 mp3 / m4a / aac），build 程式會檢查個檔案存在。

## 點解冇用「更新成個 App 程式」（live update）？

有個叫 `@capgo/capacitor-updater` 嘅插件可以喺唔經 App Store 嘅情況下換走成個 App 嘅程式碼。
我哋評估過，**而家唔用**，原因：

1. Apple 規則 2.5.2 唔准下載「會加入或者改變 App 功能」嘅程式碼；我哋想用佢做嘅正正係「資料做唔到嘅新功能」。
   兒童 App 被拒或者被下架嘅代價好大。
2. 個插件預設會將 App 狀態、當機、JavaScript 錯誤等資料傳去 Capgo 嘅伺服器（要逐個設定關），
   同我哋「冇資料離開部機」嘅原則衝突，一個設定錯就出事。
3. 如果有人入咗你個 GitHub 戶口，佢可以換走成個 App 嘅程式碼（加連結、加追蹤）。要用加密簽名先安全，
   要管理私人鎖匙，對一個人維護太易出錯。
4. 自動還原只係喺新程式「開唔到」嗰陣先生效；開到但玩到一半壞咗就唔會還原。
5. 呢度冇 iOS / Android 專案同 Xcode 可以測試，唔可以放未測試嘅原生程式落兒童 App。

活動、島、帽、寵物、任務、字表、錄音全部都用上面嘅「內容包」做到，唔使換程式碼。
