# 中文

## 臺灣劇本殺哪裡玩？ — 前端

「臺灣劇本殺哪裡玩？」的前端網站，部署在 GitHub Pages（`https://obearwu.github.io/TaiwanLARP/`），資料來自後端 API `ObearWu/TaiwanLARP-backend`（Private repo）。這份文件分成兩個獨立的部分，請依需求閱讀：

- **[專案報告](#專案報告)**：前台怎麼從「資料寫死在 HTML」改成「從 API 讀資料」，以及為什麼這樣改。
- **[架設 SOP](#架設-sop)**：本機怎麼跑起來測試、怎麼部署到 GitHub Pages。

這兩部分刻意分開，不要交叉閱讀；報告是背景知識，SOP 是操作手冊。系統整體架構、資料庫、API 的完整說明見 `TaiwanLARP-backend` 的 README。

---

## 專案報告

### 這是什麼

全臺劇本殺資訊整理網站的前台：店家列表（含縣市分布地圖）、私團主、壽星優惠、餐飲規範、臺灣原創劇本五個分頁。純 vanilla JavaScript，沒有框架、沒有建置流程，直接把檔案交給 GitHub Pages 服務。

### 只換資料來源，畫面完全不變

網站原本是一個單一 HTML 檔，資料寫死在 JavaScript 陣列裡（`STORES`、`HOSTS`、`HOST_URLS`、`BDAY`、`FOOD`、`SCRIPTS`、`CITY_ORDER`、`CITY_POS`）。搬到資料庫之後，改動刻意壓到最小：

- 拿掉那些寫死的陣列，改成載入時呼叫一次 `GET /api/public/all`。後端回傳的欄位格式跟原本的陣列一模一樣（見後端 `content/public.py`），拿到之後直接指定給同名變數。
- 原本畫面與互動的程式碼一行都沒改，只是整段包進 `start()`，等資料到了才執行。
- 頁面上三段說明文字（壽星優惠、餐飲規範標題旁的更新日期與來源、「暫不提供生日優惠」那行）原本寫死在 HTML，改成從 API 的 `texts` 填入，讓它們也能在後台修改。
- 原本嵌在 HTML 裡的黑熊主視覺圖片（base64，約 70KB 的文字），抽成 `assets/bear.jpg`。

驗證方式：在本機同時開原本的 HTML 跟新版頁面，用程式比對兩邊畫出來的 HTML。比對範圍包含統計數字、店家卡片、縣市與類型篩選、地圖、私團主、壽星優惠、餐飲規範表、劇本與標籤，以及顯示已歇業、點縣市、點類型、搜尋店家、搜尋私團主、點劇本標籤這幾種操作之後的結果。7 種狀態 × 14 個區塊全部完全一致，截圖也看不出差異。

### 唯一新增的畫面：載入中與載入失敗

資料要等 API 回應才有，所以多了一個載入狀態：資料到之前，各分頁顯示「黑熊正在搬資料，請稍候…」；API 連不上時改顯示「黑熊搬資料時跌倒了，請稍後重新整理頁面。」，沿用原本「找不到資料」的同一種樣式。

### API 網址怎麼決定

`index.html` 裡的 `API_BASE`：網址是 `localhost` 或 `127.0.0.1`（本機測試）時打 `http://127.0.0.1:5001`，其他情況打 Render 上的正式後端。

### 檔案結構

```
index.html          整個網站（樣式、畫面、互動程式都在這一個檔案）
assets/bear.jpg     黑熊主視覺圖片（頁首大圖、頁尾頭像共用）
```

### 已知限制

- 資料載入完成之前（以及載入失敗時），上方分頁按鈕、回到頂端按鈕都還不能用——原本的程式整段包在 `start()` 裡，這些互動也一起等資料到了才綁定。刻意不拆開，是為了讓原本的程式碼保持一行不改。
- 公開資料 API 有 60 秒瀏覽器快取：後台改了資料，訪客最慢 60 秒後才看得到。
- 頁尾的「資料來源：臺灣劇本殺哪裡玩？（Google 試算表）」連結保留原樣。資料現在以資料庫為準，試算表不再維護，這行之後要改成什麼，等網站擁有者決定。

---

# 架設 SOP / Setup Guide

## Part A. 本機測試

前台要搭配後端一起跑，後端的設定方式見 `TaiwanLARP-backend` README 的 Part A。

### 1. 啟動後端

開一個終端機：

```bash
cd TaiwanLARP/backend
source venv/bin/activate
python app.py
```

看到 `Running on http://127.0.0.1:5001` 代表後端已啟動。

### 2. 啟動前台

另開一個終端機：

```bash
cd TaiwanLARP/web
python3 -m http.server 8000
```

瀏覽器打開 `http://localhost:8000/`，應該看到完整的網站。用 `localhost` 而不是 `127.0.0.1`：後台的 Google 登入不接受 IP 位址當來源。

> 註：不能直接雙擊 `index.html` 用 `file://` 打開——那樣頁面沒有正常的來源（Origin 是 `null`），後端的 CORS 設定會擋下 API 請求，畫面會停在「黑熊搬資料時跌倒了」。

## Part B. 部署到 GitHub Pages

1. 打開 `https://github.com/ObearWu/TaiwanLARP` → **Settings** → 左側 **Pages**。
2. **Build and deployment** 的 **Source** 選 **Deploy from a branch**。
3. **Branch** 選 `main`，資料夾選 `/ (root)`，按 **Save**。
4. 等 1~2 分鐘，打開 `https://obearwu.github.io/TaiwanLARP/` 確認網站正常。之後每次 push 到 `main` 都會自動重新部署。

> 註：免費方案的 GitHub Pages 要求 repo 是 Public，這個 repo 因此是 Public。前端本來就是公開給所有人看的網頁，裡面沒有任何密鑰。

---

# English

## Where to Play LARP in Taiwan — Frontend

The frontend for *Where to Play LARP in Taiwan* ("臺灣劇本殺哪裡玩？"), deployed on GitHub Pages (`https://obearwu.github.io/TaiwanLARP/`), with data from the backend API `ObearWu/TaiwanLARP-backend` (a Private repo). This document has two independent parts; read whichever you need:

- **[Project Report](#project-report)**: how the public site went from "data hard-coded in HTML" to "data read from an API", and why it was done this way.
- **[Setup Guide](#setup-guide)**: how to run it locally for testing and how to deploy it to GitHub Pages.

The two parts are deliberately separate and shouldn't be read interleaved: the report is background knowledge, the guide is an operations manual. For the overall system architecture, database, and API, see the README in `TaiwanLARP-backend`.

---

## Project Report

### What This Is

The public site for a collection of LARP (murder mystery / 劇本殺) information across Taiwan, with five tabs: stores (with a map of their distribution by city), private hosts, birthday deals, food and drink rules, and original Taiwanese scripts. Pure vanilla JavaScript — no framework, no build step; the files are handed straight to GitHub Pages.

### Only the Data Source Changed; the Page Looks Exactly the Same

The site started as a single HTML file with its data hard-coded in JavaScript arrays (`STORES`, `HOSTS`, `HOST_URLS`, `BDAY`, `FOOD`, `SCRIPTS`, `CITY_ORDER`, `CITY_POS`). Moving to a database, the changes were deliberately kept to a minimum:

- The hard-coded arrays were removed and replaced with a single `GET /api/public/all` call on load. The backend returns fields in exactly the same shape as the original arrays (see the backend's `content/public.py`), which are assigned straight to variables of the same names.
- Not one line of the original rendering and interaction code changed; it's just wrapped in `start()` so it runs once the data arrives.
- Three pieces of page text that were hard-coded in the HTML (the update dates and sources next to the birthday-deal and food-rule headings, and the "暫不提供生日優惠" line) are now filled from the API's `texts`, so they can be edited in the admin interface too.
- The bear hero image that was embedded in the HTML (base64, about 70KB of text) was extracted to `assets/bear.jpg`.

How it was verified: the original HTML and the new page were opened side by side locally, and a script compared the HTML each one rendered. The comparison covered the stats, store cards, city and type filters, the map, private hosts, birthday deals, the food-rule table, scripts and tags — plus the results after showing closed stores, clicking a city, clicking a type, searching stores, searching hosts, and clicking a script tag. All 7 states × 14 sections matched exactly, and screenshots showed no difference.

### The Only New UI: Loading and Load Failure

The data now arrives with the API response, so there's a loading state: until the data arrives, each tab shows "黑熊正在搬資料，請稍候…" ("The bear is moving the data, please wait…"); if the API can't be reached, it shows "黑熊搬資料時跌倒了，請稍後重新整理頁面。" ("The bear tripped while moving the data; please refresh later."), using the same style as the original "nothing found" message.

### How the API URL Is Chosen

`API_BASE` in `index.html`: when the page is served from `localhost` or `127.0.0.1` (local testing) it calls `http://127.0.0.1:5001`; otherwise it calls the production backend on Render.

### File Layout

```
index.html          The whole site (styles, markup, and interaction code all in this one file)
assets/bear.jpg     The bear hero image (shared by the header image and the footer avatar)
```

### Known Limitations

- Until the data finishes loading (and if loading fails), the tab buttons and the back-to-top button don't work yet — the original code is wrapped in `start()` as a whole, so those interactions are bound only once the data arrives. They're deliberately not split out, to keep the original code unchanged line for line.
- The public data API has a 60-second browser cache: edits made in the admin interface reach visitors within 60 seconds.
- The footer link "資料來源：臺灣劇本殺哪裡玩？（Google 試算表）" (data source: the Google Sheet) is kept as is. The database is now the source of truth and the Sheet is no longer maintained; what this line should say going forward is up to the site owner.

---

# Setup Guide

## Part A. Local Testing

The frontend has to run alongside the backend; see Part A of the `TaiwanLARP-backend` README for setting up the backend.

### 1. Start the Backend

Open a terminal:

```bash
cd TaiwanLARP/backend
source venv/bin/activate
python app.py
```

`Running on http://127.0.0.1:5001` means the backend is up.

### 2. Start the Frontend

Open another terminal:

```bash
cd TaiwanLARP/web
python3 -m http.server 8000
```

Open `http://localhost:8000/` in a browser and you should see the full site. Use `localhost`, not `127.0.0.1`: Google sign-in on the admin page doesn't accept an IP address as an origin.

> Note: you can't just double-click `index.html` and open it via `file://` — that gives the page no real origin (its Origin is `null`), the backend's CORS settings block the API request, and the page gets stuck on "黑熊搬資料時跌倒了".

## Part B. Deploy to GitHub Pages

1. Open `https://github.com/ObearWu/TaiwanLARP` → **Settings** → **Pages** in the left sidebar.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Set **Branch** to `main` with the `/ (root)` folder, and click **Save**.
4. Wait 1–2 minutes, then open `https://obearwu.github.io/TaiwanLARP/` to confirm the site works. Every later push to `main` redeploys automatically.

> Note: GitHub Pages on the free plan requires a Public repo, which is why this repo is Public. The frontend is a page meant for everyone to see anyway, and it contains no secrets.
