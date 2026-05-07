# SafeCard — 專案架構說明

> 在新對話中，Claude 只需讀取此檔案即可快速掌握整個專案。

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

### 卡片尺寸變數

```css
/* 緊急聯絡小卡 */
--hw: 90mm;   /* 寬 */
--hh: 54mm;   /* 高（標準名片） */
--hs: 1.6;    /* 預覽縮放倍率（桌機）；行動版改為 1.0 */

/* 寵物個資卡 */
--pw: 74mm;   /* 寬 */
--ph: 105mm;  /* 高（A7） */
--ps: 1.6;    /* 預覽縮放倍率；行動版改為 1.2 */
```

---

## 頁面結構

```
<header class="banner no-print">          ← sticky，頁面頂部
<nav class="tab-bar no-print">            ← sticky，tab 切換
<div class="main" id="tab-human">         ← 緊急聯絡小卡 tab
  <aside class="form-panel">              ← 左欄表單（420px 固定寬，自身捲動）
  <section class="preview-panel">         ← 右欄即時預覽（sticky）
<div class="main" id="tab-pet" hidden>    ← 寵物卡 tab（結構相同）

<!-- 列印區塊：螢幕上隱藏，列印時顯示 -->
<div class="print-human">                 ← #print-front / #print-back
<div class="print-pet">                   ← #print-pet-front / #print-pet-back
```

### `.no-print` / `.print-only` 規則

- 表單、預覽面板都有 `no-print`，列印時隱藏
- `body[data-tab="human"]` → 隱藏 `.print-pet`，顯示 `.print-human`（反之亦然）
- 切換 tab 時同步更新 `document.body.dataset.tab`

---

## 兩張卡片的欄位

### 緊急聯絡小卡（emergency-card，90×54mm）

**正面** input IDs：`name`, `blood`（select）, `dob`, `idnum`, `allergy`, `medical`
- 版面：左欄 fields + 右上角 22×22mm 照片框（`card-photo-box`）

**背面** input IDs：`c1name`, `c1rel`, `c1tel`, `c2name`, `c2rel`, `c2tel`, `loc1`, `loc2`

### 寵物個資與健康資料卡（pet-card，74×105mm）

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

### 即時同步

```js
sync()      // 重建人類卡 HTML → 注入 #preview-front/back 和 #print-front/back
syncPet()   // 重建寵物卡 HTML → 注入 #preview-pet-front/back 和 #print-pet-front/back
```

每個 input 都掛 `oninput="sync()"` 或 `oninput="syncPet()"` 觸發即時更新。

### 卡片 HTML 生成

```js
buildFront()     // 人類卡正面 → 回傳 HTML string
buildBack()      // 人類卡背面 → 回傳 HTML string
buildPetFront()  // 寵物卡正面 → 回傳 HTML string
buildPetBack()   // 寵物卡背面 → 回傳 HTML string
```

內部 helper：
- `V(v, ph)` — 有值顯示值，無值顯示灰色佔位符（人類卡）
- `PV(v, ph)` — 同上（寵物卡）
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
let petXf   = {x:0, y:0, s:1, minS:undefined}  // 寵物卡照片 transform 狀態
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
  .card-header         ← navy 底，白字，顯示「緊急聯絡小卡」+ 「正/背面」
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

### 寵物卡

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

## 行動版斷點

```css
@media (max-width: 768px) {
  .main { grid-template-columns: 1fr; }   /* 單欄：表單在上，預覽在下 */
  .preview-panel {
    position: static;
    --hs: 1.0;   /* 人類卡縮放降為 1.0 */
    --ps: 1.2;   /* 寵物卡縮放降為 1.2 */
  }
}
```

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
