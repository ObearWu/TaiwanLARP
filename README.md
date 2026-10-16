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

### 後台頁面（admin.html）

`admin.html` 是獨立的後台頁面，前台沒有任何連結指向它（前台畫面維持不變），網址是 `https://obearwu.github.io/TaiwanLARP/admin.html`。頁面加了 `noindex`，不會被搜尋引擎收錄；但真正擋住未授權的人的是後端——每支後台 API 都會檢查登入與權限，頁面本身被看到也沒關係。

- **登入**：Google 登入按鈕 → 拿 Google ID Token 換後端發的 session token，存在瀏覽器的 `localStorage`，下次打開不用重新登入。Google 登入成功、但 email 不在使用者列表裡的人，會停在登入畫面看到「此帳號尚未獲得授權」。後端暫時連不上（例如 Render 剛好在喚醒）時不會清掉 session，重新整理就會再試一次。
- **分頁**：店家、縣市、私團主、壽星優惠、餐飲規範、原創劇本、頁面文字、使用者（使用者分頁只有管理員看得到）。
- **每個內容分頁**：列表、搜尋、新增、編輯、刪除（刪除前確認）。表單欄位對應後端 `content/resources.py` 的欄位定義；輸入錯誤時顯示後端回傳的訊息（例如「店名不能空白」「網址必須是 http:// 或 https:// 開頭」）。新增的資料預設排在最後。
- **刻意的保護**：縣市底下還有店家時不能刪除；不能停用自己或移除自己的管理員權限；停用的使用者會立刻被登出。

驗證方式（2026-10-16，本機）：用一組臨時的管理員 session 實際操作每個分頁——新增、編輯、搜尋、刪除，以及空白必填欄位、錯誤網址、刪除使用中的縣市、停用自己這幾種該被擋下的情況，並確認改動確實出現在 `/api/public/all`；手機寬度（375px）沒有整頁橫向捲動、深色模式正常。測試資料事後全部刪除，並重新比對資料庫內容與原始資料完全一致。真正的 Google 登入需要網站擁有者本人操作。

### API 網址怎麼決定

`index.html` 裡的 `API_BASE`：網址是 `localhost` 或 `127.0.0.1`（本機測試）時打 `http://127.0.0.1:5001`，其他情況打 Render 上的正式後端。

### 檔案結構

```
index.html          前台（樣式、畫面、互動程式都在這一個檔案）
admin.html          後台（同樣是單一檔案）
assets/bear.jpg     黑熊主視覺圖片（頁首大圖、頁尾頭像共用）
assets/favicon-32.png、assets/apple-touch-icon.png
                    分頁圖示與手機加到主畫面的圖示，由主視覺圖只裁頭部做成（整隻熊縮到 32px 臉會看不清楚）
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

瀏覽器打開 `http://localhost:8000/`，應該看到完整的網站；後台在 `http://localhost:8000/admin.html`。用 `localhost` 而不是 `127.0.0.1`：後台的 Google 登入不接受 IP 位址當來源。

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

### The Admin Page (admin.html)

`admin.html` is a standalone admin page with no links to it from the public site (the public page stays unchanged); its URL is `https://obearwu.github.io/TaiwanLARP/admin.html`. The page carries `noindex` so search engines don't list it; but what actually keeps unauthorized people out is the backend — every admin API checks sign-in and permissions, so it doesn't matter if someone sees the page itself.

- **Sign-in**: the Google sign-in button → exchange the Google ID Token for a backend-issued session token, kept in the browser's `localStorage` so you don't have to sign in again next time. Someone whose Google sign-in succeeds but whose email isn't in the users list stays on the sign-in screen with "此帳號尚未獲得授權" (this account isn't authorized). If the backend is temporarily unreachable (say, Render is waking up), the session isn't cleared; refreshing tries again.
- **Tabs**: stores, cities, private hosts, birthday deals, food rules, original scripts, page texts, and users (the users tab is visible to admins only).
- **Each content tab**: list, search, add, edit, delete (with confirmation). Form fields mirror the field definitions in the backend's `content/resources.py`; input errors show the backend's message (e.g. "店名不能空白" — store name can't be blank; "必須是 http:// 或 https:// 開頭" — must start with http:// or https://). New entries are placed last by default.
- **Deliberate safeguards**: a city that still has stores can't be deleted; you can't disable yourself or remove your own admin role; disabled users are signed out immediately.

How it was verified (2026-10-16, locally): using a temporary admin session, every tab was exercised for real — add, edit, search, delete, plus the cases that should be blocked (blank required fields, a bad URL, deleting a city in use, disabling yourself) — confirming each change showed up in `/api/public/all`; at phone width (375px) the page doesn't scroll sideways, and dark mode works. All test data was deleted afterward, and the database was re-compared against the original data and matched exactly. Real Google sign-in has to be done by the site owner in person.

### How the API URL Is Chosen

`API_BASE` in `index.html`: when the page is served from `localhost` or `127.0.0.1` (local testing) it calls `http://127.0.0.1:5001`; otherwise it calls the production backend on Render.

### File Layout

```
index.html          The public site (styles, markup, and interaction code all in this one file)
admin.html          The admin interface (also a single file)
assets/bear.jpg     The bear hero image (shared by the header image and the footer avatar)
assets/favicon-32.png, assets/apple-touch-icon.png
                    The tab icon and the home-screen icon, cropped from the hero image to just the head (at 32px the whole bear's face becomes illegible)
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

Open `http://localhost:8000/` in a browser and you should see the full site; the admin interface is at `http://localhost:8000/admin.html`. Use `localhost`, not `127.0.0.1`: Google sign-in on the admin page doesn't accept an IP address as an origin.

> Note: you can't just double-click `index.html` and open it via `file://` — that gives the page no real origin (its Origin is `null`), the backend's CORS settings block the API request, and the page gets stuck on "黑熊搬資料時跌倒了".

## Part B. Deploy to GitHub Pages

1. Open `https://github.com/ObearWu/TaiwanLARP` → **Settings** → **Pages** in the left sidebar.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Set **Branch** to `main` with the `/ (root)` folder, and click **Save**.
4. Wait 1–2 minutes, then open `https://obearwu.github.io/TaiwanLARP/` to confirm the site works. Every later push to `main` redeploys automatically.

> Note: GitHub Pages on the free plan requires a Public repo, which is why this repo is Public. The frontend is a page meant for everyone to see anyway, and it contains no secrets.
