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

設計依這幾個 skill（裝在 `~/.claude/skills/`）審查與改寫，改 UI 前可再跑一次：
`/apple-design`（Apple HIG 審查）、`/frontend-design`（視覺方向、避免模板感）、`/ui-ux-pro-max`（UX 規則與 pre-delivery checklist）。

視覺概念取材自遊戲的「能量膠囊」：
- 招牌元素是結果面板裡的**能量膠囊分組量表**（`.meter` + `updateMeter()`）：分組由下（0）往上（1M），
  液面高度 = 目前分組 index + 在該組內的進度，金色刻度是你的位置。改 `groupRows` 時量表會自動跟著變。
- 深藍 `--night` 只用在結果面板與手機底部結果列（整頁唯一的重色區塊）；輸入區是一整張紙（`.inputs`），
  三個段落用分隔線區隔，不要再拆成多張一樣的卡片。
- 數字（預估分組、總能量、小計、分組表、步驟編號）用 Google Fonts「Baloo 2」（`--font-display`），
  內文用系統中日韓字型。離線時會退回系統字型。
- `--capsule`（膠囊藍）= 你累積的能量；`--gold` = 你的結果。

- 版面順序對應使用流程：步驟 1 基礎能量 → 步驟 2 剩餘道具 → 步驟 3 結果（預估分組是最大的視覺焦點）。
  頂部三個步驟是連結，會跳到對應區塊。
- 桌機（> 900px）：左側輸入、右側 sticky 結果面板。
- 手機（≤ 900px）：單欄；底部固定結果列（總能量 + 預估分組 + 「看結果」），結果區進入畫面時自動隱藏。
- **一種顏色只代表一種意思**（HIG `color.md`）：
  - `--gold`：只用在「你的結果」（預估分組、目前所在組、進度條、主要動作按鈕）。
  - `--c-base` / `--c-magic` / `--c-normal`：三種能量來源。區塊加上 `src-base` / `src-magic` / `src-normal` class，
    道具格填值後會用該來源色亮起，與結果面板的比例條、圖例對應。
  - 署名「© 由 大佬群CSL 製作」是安靜的灰字，不要改回彩色徽章（branding 要讓位給內容）。
- 支援淺色/深色模式（`prefers-color-scheme`），所有顏色都是 `:root` token，新增顏色要同時給 light 與 dark 值，
  文字對比至少 4.5:1（token 旁的註解有算好的對比值）。另支援 `prefers-contrast: more` 與 `prefers-reduced-motion`。
- 字級一律用 `rem`；iOS 會透過 `font: -apple-system-body` 跟隨系統文字大小。道具格用
  `repeat(auto-fill, minmax(min(100%, Xrem), 1fr))`，文字放大時自動減少欄數。輸入框字級至少 16px（避免 iOS 自動放大）。
- 觸控目標 ≥ 44px；placeholder 顏色用 `--placeholder`（≥ 4.5:1）。
- 輸入會存在 `localStorage["calculatorInputs"]`（玩家切回遊戲看數字再回來，頁面被重新載入也不會遺失）。
- 「清除全部」不跳確認視窗，改成清除後顯示 8 秒的「復原」列。
- 修改後要在 375px 與 1280px 寬度、淺色與深色、200% 文字大小下實際檢查，確認沒有水平捲軸、
  越南文/泰文等長字串不會溢出。

## 慣例

- 介面文字預設繁體中文；commit 訊息沿用 `fix:` / `feat:` 等前綴。
- 本工具為非官方玩家工具，頁尾免責聲明要保留。
