# SafeCard — 專案架構說明

> 在新對話中，Claude 只需讀取此檔案即可快速掌握整個專案。

---

## 更新紀錄

### 2026-06-07 — Direction C 全站改版（PR #2）

從 Claude Design 接手稿實作，整合多輪迭代後的最終設計，已 merge 進 `main` 上線：

- **三張卡片**（原為兩張）：`human` 避難聯絡卡、`med` **醫療資訊卡（新增）**、`pet` 動物夥伴卡。三者以 segmented 產品選擇器分頁切換。
- **版面改為 Direction C「卡片為主角」**：進站先看**全頁封面**（`.cover`，深色＋警示斜紋，依「要幫誰準備」推薦卡片）；進入後為**左側深色 Hero**（包住即時預覽卡＋橘色編號徽章＋**正面／背面切換**）、**右側表單**的雙欄版面。
- **用語統一**：全站「寵物／毛孩」→「動物夥伴」。
- **易用性功能落地**（見下方 Backlog 對應項）：字級切換（UX-1）、首次引導 Onboarding（UX-2）、實際尺寸預覽＋信用卡校正（UX-4）。
- **表單美化**：橘色序號徽章（01／02…）、用藥／餐次群組小卡。
- **行動版斷點改為 1000px**（原 768px），預覽改為**底部抽屜（bottom sheet）**＋浮動鈕，抽屜內可直接列印。
- 套用 18 處 UX Writing 修正；新增 `.gitignore` 排除本機 `.claude/`。

> ⚠️ 以下章節部分仍描述舊版（兩張卡、768px 斷點等），逐步更新中。新版的權威來源以 `index.html` 為準；關鍵差異已在本紀錄與下方對應章節標註。

---

## 專案概述

台灣防災避難聯絡卡產生器。使用者在瀏覽器填寫資料後，直接列印成可放入避難包的小卡。

- **線上版**：https://safecard.claire-cheng.com
- **GitHub**：https://github.com/Legendream/safecard
- **本地開發**：`python3 -m http.server 4321 --directory /Users/Claire/Claude/safecard`

---

## 技術架構

**純靜態單一 HTML 檔案**——所有 HTML + CSS + JS 都內嵌於 `index.html`，無框架、無 npm、無後端、可離線使用。

```
/Users/Claire/Claude/safecard/
├── index.html   ← 唯一的程式碼檔案
├── README.md
└── PLAN.md      ← 本文件
```

**隱私設計**：零後端、零儲存、零外連。資料只存在瀏覽器記憶體，重整即消失。

---

## CSS 設計系統

### 色票變數（方案 A：暖炭 + 磚橙）

```css
:root {
  --navy:    #1C1917;   /* 暖黑炭：Banner、卡片 header、section badge 背景 */
  --navy-dk: #0C0A09;   /* 更深暖黑：hover 狀態 */
  --blue:    #C2410C;   /* 磚橙：強調色、focus、active tab 底線、血型徽章 */
  --blue-lt: #FED7AA;   /* 淡橙：tab-bar 底線、分隔線、左側 border */
  --text:    #1C1917;   /* 主文字 */
  --mid:     #78716C;   /* 次要文字、欄位 label、card-key/pet-key */
  --light:   #D6D3D1;   /* 細邊框、輸入框 border */
  --bg:      #FFF7ED;   /* 頁面底色（暖米） */
  --white:   #ffffff;
  --font:    system-ui, 'Noto Sans TC', 'PingFang TC', sans-serif;
}
```

### 卡片尺寸變數（現行：三張卡）

```css
/* 01 避難聯絡卡（human）*/
--hw: 90mm;   --hh: 54mm;   --hs: 1.2;   /* 名片；--hs 行動版抽屜內改 0.82 */

/* 03 動物夥伴卡（pet）*/
--pw: 74mm;   --ph: 105mm;  --ps: 1.25;  /* A7；--ps 行動版改 0.96 */

/* 02 醫療資訊卡（med，本次新增）*/
--mw: 74mm;   --mh: 105mm;  --ms: 1.25;  /* --ms 行動版改 0.96 */

--stick: 132px;  /* sticky 偏移：banner + tab-bar 高度，供右欄 Hero 定位 */
```

---

## 頁面結構（Direction C）

```
<div class="cover no-print" id="cover">   ← 全頁封面（深色＋警示斜紋，進站先顯示）
                                            三張產品卡 + 「要幫誰準備」推薦器
<header class="banner no-print">          ← sticky，頁面頂部（含字級切換 S/M/L）
<nav class="tab-bar no-print">            ← sticky，segmented 產品選擇器（3 分頁）

<div class="main" id="tab-human">         ← 01 避難聯絡卡 tab
  <section class="preview-panel">         ← 左欄：深色 Hero 預覽（grid-column 1，sticky）
                                            橘框＋編號徽章＋正面/背面切換
  <aside class="form-panel">              ← 右欄：表單（grid-column 2，自身捲動）
<div class="main" id="tab-med" hidden>    ← 02 醫療資訊卡 tab（結構相同）
<div class="main" id="tab-pet" hidden>    ← 03 動物夥伴卡 tab（結構相同）

<!-- 模態：封面以外的引導 -->
<div class="onboard-overlay">             ← 首次進入引導（localStorage 記住）
<div class="calib-overlay">               ← 實際尺寸校正（信用卡對照）

<!-- 列印區塊：螢幕上隱藏，列印時依 data-tab 顯示對應卡 -->
<div class="print-human"> / <div class="print-med"> / <div class="print-pet">
```

> 重要：表單在 HTML 裡排在預覽**之前**，靠 `grid-column` + `grid-row: 1` 把預覽釘在左欄第一列。若漏掉 `grid-row: 1`，Grid 稀疏排版會把預覽擠到第二列（表單下方）。

### `.no-print` / `.print-only` 規則

- 表單、預覽面板都有 `no-print`，列印時隱藏
- `body[data-tab="human"]` → 隱藏 `.print-pet`，顯示 `.print-human`（反之亦然）
- 切換 tab 時同步更新 `document.body.dataset.tab`

---

## 三張卡片的欄位

### 01 避難聯絡卡（emergency-card，90×54mm）

**正面** input IDs：`name`, `dob`, `idnum`, `allergy`, `medical`, `instruction`（急救指示）
- 版面：左欄 fields + 右上角 22×22mm 照片框（`card-photo-box`）
- 註：血型已移到 02 醫療資訊卡（`m_blood`）

**背面** input IDs：`c1name`, `c1rel`, `c1tel`, `c2name`, `c2rel`, `c2tel`, `loc1`, `loc2`

### 02 醫療資訊卡（medical-card，90×54mm，本次新增）

input IDs（`m_` 前綴）：`m_name`, `m_blood`, `m_dob`, `m_drug_allergy`, `m_food_allergy`, `m_condition`, `m_medication`, `m_instruction`, `m_implant`, `m_organ_donor`, `m_nhi`（健保卡號）, `m_doctor`, `m_hospital`, `m_hospital_tel`, `m_cname`/`m_crel`/`m_ctel`（緊急聯絡）

### 03 動物夥伴卡（pet-card，74×105mm）

**正面** input IDs：`p_name`, `p_species`, `p_gender`, `p_breed`, `p_dob`（出生年，number）, `p_desc`（外觀，建議≤20字）, `p_chip`, `p_chip_id`, `p_tag`, `p_tag_desc`, `p_id_other`
- 版面：左欄 fields + 右側 28×28mm 照片框（`pet-photo`）
- **用藥狀況**：動態欄位，以 `#pet-med-list` 容器管理，每筆 `.med-entry` 包含：
  - `.f-condition`（健康狀況，連結 `<datalist id="pet-conditions-list">`，附 12 個常見病症快選標籤）
  - `.f-med`（藥名）、`.f-purpose`（用途）、`.f-freq`（頻率）
  - 第二筆起顯示「✕ 移除」按鈕；底部有「＋ 新增用藥」按鈕

**背面** input IDs：`p_owner`, `p_owner_tel`, `p_owner_contact`, `p_owner_addr`, `p_neutered`, `p_drug_allergy`, `p_vac1`, `p_vac1_date`, `p_vac2`, `p_vac2_date`, `p_food_note`, `p_vet_hospital`, `p_vet_name`, `p_vet_tel`, `p_vet_addr`
- **飲食習慣**：動態欄位，以 `#pet-meal-list` 容器管理，每筆 `.meal-entry` 包含 `.f-time`（餐次）和 `.f-desc`（食物與份量）；底部有「＋ 新增餐次」按鈕
- **就診診所**：電話與地址各佔獨立一行（`p_vet_addr` 已從兩欄並排改為全寬獨立欄位）

---

## JavaScript 核心函數

### 即時同步（三張卡）

```js
sync()      // 重建避難聯絡卡 HTML → 注入 human 預覽/列印區
syncMed()   // 重建醫療資訊卡 HTML（本次新增）
syncPet()   // 重建動物夥伴卡 HTML → 注入 pet 預覽/列印區
```

每個 input 都掛對應的 `oninput="sync()" / "syncMed()" / "syncPet()"` 觸發即時更新。

### 卡片 HTML 生成

```js
buildFront()     / buildBack()     // 避難聯絡卡 正/背面 → HTML string
buildMedFront()  / buildMedBack()  // 醫療資訊卡 正/背面（本次新增）
buildPetFront()  / buildPetBack()  // 動物夥伴卡 正/背面
```

### 封面 / 引導 / 正反面切換（本次新增）

```js
enterCard(tab)        // 從封面進入指定卡片，隱藏 .cover、設定 body.dataset.tab
toggleAud(btn)        // 封面「要幫誰準備」多選
enterRecommended()    // 依選擇推薦並進入對應卡片
setFace('front'|'back') // 切換預覽正/背面（body[data-face]）
setFontSize('S'|'M'|'L')// 字級切換（UX-1）
dismissOnboard()      // 關閉首次引導，狀態存 localStorage（UX-2）
confirmActualSize()   // 完成信用卡校正，進入 1:1 實際尺寸預覽（UX-4）
```

內部 helper：
- `V(v, ph)` — 有值顯示值，無值顯示灰色佔位符（人類卡）
- `PV(v, ph)` — 同上（動物夥伴卡）
- `R(k, v)` — 快速產生 `<div class="pet-row">` 一行

### 動態欄位

用藥與飲食兩區的資料不再用固定 ID 讀取，改為**DOM query**：

```js
// buildPetFront() 內
const medEntries = Array.from(document.querySelectorAll('#pet-med-list .med-entry')).map(e => ({
  condition: e.querySelector('.f-condition')?.value.trim() || '',
  med:       e.querySelector('.f-med')?.value.trim() || '',
  purpose:   e.querySelector('.f-purpose')?.value.trim() || '',
  freq:      e.querySelector('.f-freq')?.value.trim() || '',
}));

// buildPetBack() 內
const mealEntries = Array.from(document.querySelectorAll('#pet-meal-list .meal-entry')).map((e, i) => ({
  time: e.querySelector('.f-time')?.value.trim() || '',
  desc: e.querySelector('.f-desc')?.value.trim() || '',
  idx:  i,
}));
```

新增／移除函數：

```js
addMedEntry()       // 在 #pet-med-list 尾端插入新 .med-entry，focus .f-condition
removeMedEntry(btn) // 移除最近的 .med-entry，重跑 syncPet()
addMealEntry()      // 在 #pet-meal-list 尾端插入新 .meal-entry，focus .f-time
removeMealEntry(btn)// 移除最近的 .meal-entry，重跑 syncPet()
pickCondition(btn, condition)
// 點擊標籤 → 讀取同 .field 內的 .f-condition，將 condition 以「、」附加到現有值
```

**注意**：`downloadBackup()` 用 index-based 複製所有 input 值（`srcInputs[i] → dstInputs[i]`），動態新增的欄位也會被完整備份，開啟備份檔後 `syncPet()` 會正確讀取 DOM 重建卡片。

### 照片系統

```js
// 狀態變數
let humanPhoto = null   // base64 dataURL 或 null
let petPhoto   = null

let humanXf = {x:0, y:0, s:1, minS:undefined}  // 人類卡照片 transform 狀態
let petXf   = {x:0, y:0, s:1, minS:undefined}  // 動物夥伴卡照片 transform 狀態
// x,y = 位移 px；s = 縮放比例；minS = cover-fit 最小比例（上傳後計算）

function loadPhoto(input, type)
// FileReader 讀取 → 存入 humanPhoto/petPhoto → 呼叫 sync/syncPet
// 上傳後用 Image.onload 計算 cover-fit 初始縮放：
//   s = Math.max(boxW / img.naturalWidth, boxH / img.naturalHeight)
// 同時設定 xf.minS = s（作為縮放下限，防止縮小後留白）

function applyXf(img, xf)
// 將 transform 套用到 <img>：
//   translate(calc(-50% + Xpx), calc(-50% + Ypx)) scale(S)

function reapplyXf()
// 每次 sync/syncPet 重建 innerHTML 後呼叫，
// 重新找到 img 元素並套用已儲存的 xf 狀態
```

**拖曳縮放事件**（document-level，只綁一次）：
- `mousedown / mousemove / mouseup` → 拖曳移動
- `wheel` (passive:false) → 滾輪縮放，`deltaY > 0` = 縮小，`< 0` = 放大
- `touchstart / touchmove / touchend` → 單指拖曳、雙指縮放（pinch）
- 縮放範圍：`[xf.minS, 6]`

**重要**：因為 sync/syncPet 每次都用 `innerHTML` 完整重寫卡片 DOM，所以：
1. 每次 sync 後必須呼叫 `reapplyXf()` 補回照片 transform
2. 事件監聽器用 document-level delegation，不需重新綁定

### 照片框 hover 提示

CSS `::after` 偽元素，有照片時 hover 顯示「拖曳移動 / 滾輪縮放」：
```css
.card-photo-box:has(img)::after,
.pet-photo:has(img)::after { /* 半透明黑底白字遮罩，opacity:0→1 on hover */ }
/* @media print 時隱藏此遮罩 */
```

### 其他功能

```js
switchTab(tab)    // 切換 tab，更新 aria-selected 和 body.dataset.tab
downloadBackup()  // 克隆當前 DOM，注入欄位值，Blob 下載為 .html 備份檔（印刷店用）
```

---

## CSS 卡片 class 層次

### 人類卡

```
.emergency-card
  .card-header         ← navy 底，白字，顯示「避難聯絡卡」+ 「正/背面」
  .card-body
    .card-front-row    ← flex row（正面專用）
      .card-front-fields  ← flex column，欄位
      .card-photo-box     ← 22mm×22mm，右上角，cursor:move
    .card-section-title  ← navy 底白字 badge（背面區段標題：緊急聯絡人、避難地點）
    .card-row
      .card-key        ← 欄位名（mid 灰）
      .card-val        ← 欄位值（有底線）
    .card-divider      ← 橫線分隔
    .blood-badge       ← 橙色血型徽章
```

### 動物夥伴卡

```
.pet-card
  .card-header
  .pet-body
    .pet-top-row       ← flex row（正面頂部）
      .pet-top-fields  ← flex column
        .pet-section-title  ← 「基本資料」（navy badge，無左側線）
        .pet-row
      .pet-photo       ← 28mm×28mm，cursor:move
    .pet-section       ← 每個區段（身份辨識、用藥、照顧者…）
      .pet-section-title  ← navy badge，浮在左側線上方（不碰線）
      .pet-section-rows   ← border-left: 2.5px solid var(--blue-lt)，內含 pet-row
        .pet-row
          .pet-key     ← min-width:10mm
          .pet-val
```

**注意**：`.pet-section-title` 在 `.pet-section` 外層（不在 `.pet-section-rows` 內），因此左側線條不會碰到標題。

---

## 列印機制

```css
@media print {
  .no-print { display: none !important; }    /* 隱藏表單、預覽、banner */

  body[data-tab="human"] .print-pet   { display: none !important; }
  body[data-tab="human"] .print-human { display: block !important; }
  /* 反之亦然 */

  .emergency-card { transform: none; position: static; width: var(--hw); height: var(--hh); }
  .pet-card       { transform: none; position: static; width: var(--pw); height: var(--ph); }
  /* 列印時移除 scale transform，以實際 mm 尺寸輸出 */

  -webkit-print-color-adjust: exact; print-color-adjust: exact;  /* 保留色塊背景 */
}
```

列印設定建議：A4 直向、縮放 100%（實際大小）、開啟背景圖形。

---

## 行動版斷點（現行：1000px + 底部抽屜）

斷點自 768px 改為 **1000px**；預覽不再放在表單下方，而是改為**底部抽屜（bottom sheet）**：

```css
@media (max-width: 1000px) {
  .main { grid-template-columns: 1fr; }   /* 單欄：只剩表單 */

  .preview-fab { display: flex; }          /* 右下角浮動「預覽卡片」鈕 */
  .preview-panel {                         /* 變成由下滑入的深色抽屜 */
    position: fixed; bottom: 0;
    transform: translateY(101%);           /* 預設收合；body.sheet-open 時 translateY(0) */
    --hs: 0.82; --ps: 0.96; --ms: 0.96;    /* 抽屜內三卡縮放 */
  }
  .sheet-footer { ... }                    /* 抽屜底部固定「列印此卡」橘鈕 */
}
```

互動：點浮動鈕 → `body.sheet-open` 滑出抽屜（深色、橘框卡片、正反面切換、列印鈕）；點遮罩或按 Esc 關閉。

---

## 常見開發陷阱

1. **瀏覽器快取**：修改 JS 後若無效，需完全重啟 server（`pkill -f "http.server 4321"`），或加 `?v=timestamp` query string。

2. **照片 transform 被清除**：每次 `sync()` 重寫 `innerHTML` 後，必須立刻呼叫 `reapplyXf()`，否則照片縮放位置會重置。

3. **縮放下限**：縮放最小值必須用 `xf.minS`（cover-fit 比例），不可用固定值（如 0.3），否則大圖的初始比例（~0.03）遠低於下限，導致縮小時反而跳大。

4. **Git push 被拒**：GitHub Pages 可能自動新增 CNAME commit，需 `git pull --rebase origin main` 再 push。

---

## 部署

- **自動部署**：push 到 `main` branch 後，GitHub Pages 自動部署（約 1~2 分鐘）
- **自訂域名**：`safecard.claire-cheng.com`（Cloudflare DNS CNAME → `legendream.github.io`）
- **CNAME 檔**：repo 根目錄的 `CNAME` 檔內容為 `safecard.claire-cheng.com`，不可刪除

---

## 待辦事項 Backlog

> 優先序尚未排定，依討論結果調整。

### 易用性改善（來自使用者測試回饋）

#### UX-1｜字型大小調節
- **問題**：使用者需要手動調整瀏覽器字級才能舒適填寫，部分使用者不知道怎麼調
- **方向**：在頁面上加 S / M / L 字型切換按鈕，調整表單 `font-size` 與 `--hs`/`--ps` 卡片縮放比例
- **狀態**：✅ **已完成**（2026-06-07）。Banner 內小／中／大三段切換，按鈕本身以漸層大小示意級距，`setFontSize()`。

#### UX-2｜首次進入引導（Onboarding）
- **問題**：新使用者不知道這頁在幹嗎，需要開發者解說
- **方向**：首次進入顯示一個簡短說明彈窗或橫幅（用途 + 3 步驟流程），可關閉，關閉狀態存 `localStorage`
- **狀態**：✅ **已完成**（2026-06-07）。`.onboard-overlay` 三步驟引導 + 全頁封面 `.cover`，`dismissOnboard()`，關閉狀態存 `localStorage`。

#### UX-3｜A5 輸出選項
- **問題**：名片尺寸適合放皮夾，但防災包希望有較大版本
- **方向**：新增「A5 列印」模式，將正反兩張卡並排印在 A5 紙上，或提供單張放大版（A6）
- **狀態**：未開始

#### UX-4｜螢幕上顯示實際尺寸對照
- **問題**：使用者無法預判印出來的物理大小
- **方向**：強化預覽區下方的尺寸提示，加上「與信用卡 / 悠遊卡 相近」視覺對照圖示；或提供一個「實際尺寸預覽」切換模式（移除 scale，以 1:1 mm 顯示）
- **狀態**：✅ **已完成**（2026-06-07）。`.calib-overlay` 以信用卡／悠遊卡校正瀏覽器縮放後，提供 1:1 實際尺寸預覽，`confirmActualSize()`。（手機版維持隱藏，依設計討論結論）

---

### 待評估功能

#### FEAT-1｜藥袋拍照自動辨識用藥
- **功能**：使用者拍攝藥袋後，自動辨識藥名、劑量、頻率，填入「目前用藥」欄位（醫療卡 `m_medication` 或動物夥伴卡用藥欄）
- **評估方向**：
  - **方案 A｜Tesseract.js（純前端 OCR）**：符合靜態 HTML 原則，但需載入 5–10MB WASM、中文辨識率不穩、藥袋格式複雜
  - **方案 B｜Claude Vision API**：辨識準確、能理解中文用藥說明，但 API Key 需放前端（安全疑慮）或需小型後端
  - **方案 C｜引導使用者用手機相機 + Google Lens 後貼入**：零開發成本，但需使用者多一步操作
- **建議**：短期先做方案 C（說明文字引導），有後端後再評估方案 B
- **狀態**：評估中，未開始
