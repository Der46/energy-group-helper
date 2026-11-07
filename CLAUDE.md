# CLAUDE.md

Guidance for Claude Code when working in this repository.

## 專案概要

Coin Master 活動道具能量換算器：玩家輸入基本/額外能量與剩餘活動道具，換算成總能量並預估下一週分組。
純靜態網頁（無建置、無相依套件），部署在 GitHub Pages，可「加入主畫面」當 PWA 使用。

## 檔案結構

- `index.html`：唯一頁面，HTML / CSS / JS 全部 inline
- `manifest.webmanifest`：PWA 名稱、圖示、主題色（`#0f172a`）
- `images/`：App 圖示與各道具圖示（檔名 = `items[].icon` + `.png`）、`total-energy-demo.png`（步驟 1 範例圖）、`energy.png`（能量圖示）
- `.claude/launch.json`：本機預覽用的 `python -m http.server 5173`

## 本機預覽

```bash
python -m http.server 5173
```

開 http://localhost:5173 。不要直接用 `file://` 開來測試，部分瀏覽器會擋圖片或 localStorage。

## 程式結構（index.html `<script>`）

所有資料都是資料驅動，改設定通常只動陣列：

- `items`：道具清單。`mul` = 1 個道具換幾點能量；`div` = 幾個道具換 1 點能量；
  `section` 為 `magic`（魔法王國）或 `normal`（一般活動）；`decimal: true` 允許小數與 K/M（目前只有紅/藍藥水）。
  道具欄位由 `renderItems()` 依此陣列產生，**HTML 裡沒有寫死的道具欄位**。
- `groupRows`：分組區間（`min`/`max` 為閉區間）。`rangeLabel` 用於比對，不要翻譯；顯示用 `displayRange()`（最高組改用 `topRange` 翻譯）。
  注意區間之間有空隙（例如 4,901–4,999），落在空隙會顯示「不在區間內」並提示到下一組還差多少。
- `translations`：`zh-Hant` / `en` / `vi` / `ja` / `ko` / `th` / `id` / `es` / `pt` 共九語。新增 UI 文字時每個語言都要加 key；
  新增語言要同時改 `<select id="languageSelect">` 的 option 與 `formatNumber()` 的 `localeMap`。
  泰文沒有空格，標題等長字串在詞與詞之間插入零寬空格（`\u200b`）避免斷在字中間；
  靜態文字用 `data-i18n="key"`，placeholder 用 `data-i18n-placeholder`。缺 key 會 fallback 到繁中。
- `calculateAll()`：每次輸入都重算總能量、小計、能量來源比例條、預估分組、下一組進度與分組對照表。
- 計算精度：道具能量相加後才 `Math.floor`，加總順序依 `items` 陣列順序（與舊版逐項相加一致）。

## 新增道具的步驟

1. 圖示放到 `images/<icon>.png`
2. 在 `items` 加一筆 `{ id, icon, section, mul | div }`
3. 在三個語言的 `translations` 加上 `<id>` 名稱
4. 更新 `README.md` 的換算比例表

## UI / UX 原則

- 版面順序對應使用流程：步驟 1 基礎能量 → 步驟 2 剩餘道具 → 步驟 3 結果（預估分組是最大的視覺焦點）。
- 桌機（> 900px）：左側輸入、右側 sticky 結果面板。
- 手機（≤ 900px）：單欄；底部固定結果列（總能量 + 預估分組 + 「看結果」），結果區進入畫面時自動隱藏。
- 設計 token 都在 `:root`（深藍 `--navy`、能量琥珀色 `--energy`、能量來源三色 `--c-base/--c-magic/--c-normal`），
  不要再加紫色漸層或裝飾性背景。
- 輸入框字級維持 16px（避免 iOS 自動放大），觸控目標 ≥ 44px。
- 修改後要在 375px 與 1280px 寬度實際檢查，確認沒有水平捲軸、越南文/泰文等長字串不會溢出。

## 慣例

- 介面文字預設繁體中文；commit 訊息沿用 `fix:` / `feat:` 等前綴。
- 本工具為非官方玩家工具，頁尾免責聲明要保留。
