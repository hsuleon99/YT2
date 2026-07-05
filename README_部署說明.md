# YT 筆記館 — 部署與使用說明

## 一、這是什麼

看完 YouTube 好影片 → 按「分享」→ 選「YT 筆記館」→ 網址自動帶入 → 寫心得、標三個時間重點、掛二層標籤 → 存入（日期自動記錄）。之後可用**日曆**或**標籤**找回來，按時間重點一鍵跳到影片該段。

## 二、部署到 GitHub Pages（與 YTToText 相同做法）

1. 到 GitHub 的 `hsuleon99.github.io` repo，建一個資料夾，例如 `ytmemo/`
2. 把這 6 個檔案全部上傳進去：
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `firebase-config.js`（跨裝置同步用，見第七章）
   - `icon-192.png`
   - `icon-512.png`
3. 等 1–2 分鐘，網址即為：`https://hsuleon99.github.io/ytmemo/`

## 三、手機安裝（關鍵一步）

**Android（Chrome）：**
1. 用 Chrome 開啟上面網址
2. 右上「⋮」→「加到主畫面」→「安裝」
3. 安裝後，YouTube App 裡按「分享」，清單中就會出現「YT 筆記館」——按下去網址自動帶入，直接寫筆記

**iPhone（Safari）：**
1. Safari 開啟網址 → 分享鍵 →「加入主畫面」
2. iOS 的分享選單不支援 PWA share target，改用：YouTube 按分享 →「拷貝連結」→ 開 YT 筆記館 → 按「📋 貼上」即可（一樣只按一個鍵）

## 四、功能對照

| 需求 | 位置 |
|---|---|
| 分享按鈕跳進 App | Android 分享選單（安裝後）／iPhone 用貼上鍵 |
| 看後心得輸入 | 新增頁「看後心得」 |
| 三行時間＋重點 | 新增頁「時間重點標記」（預設 3 行，可加行）|
| 二層標籤（如：法律→判決案例）| 新增頁下拉選；「更多」頁統一管理 |
| 保存日期自動記錄 | 存入時自動寫入，不必 key |
| 日期搜尋（日曆式）| 「日曆」頁，有存影片的日期會亮金點，點日期列出當天項目 |
| 標籤搜尋 | 「影片庫」頁，大標籤→子標籤兩層 chips |
| 按重點跳到影片該段 | 每張卡片的「▶ 12:34 重點」鍵，直接開 YouTube 到該秒 |

## 五、資料備份（重要）

資料存在手機瀏覽器本機（localStorage）。**清除瀏覽器資料會一併清掉筆記**，請養成習慣：
「更多」→「⬇ 匯出備份」，JSON 檔存到雲端硬碟。換手機時「⬆ 匯入備份」即還原（會智慧合併，不會蓋掉現有資料）。

另有「📊 匯出 Excel」：產生 .xlsx，工作表一「影片清單」一部影片一列（日期／標題／標籤／心得／重點）；工作表二「時間重點」一個重點一列，附可點的跳段連結，在電腦上點了直接開 YouTube 到該秒。Excel 適合列印與整理，但匯回 App 請用 JSON。

## 六、小提醒

- 時間格式：`12:34` 或 `1:02:45` 都可以
- 影片標題會自動抓取（YouTube oEmbed），抓不到時可手動輸入
- 縮圖點下去 = 從頭播放；時間重點鍵 = 跳到該段

## 七、跨裝置同步（Firebase 設定，一次搞定終身受用）

沒設定 Firebase 時，App 是「本機模式」，各裝置資料不互通。設定完成後：**手機存一筆，電腦馬上看到**（即時同步），而且離線也能記，回到網路自動補傳。

Firebase 免費方案（Spark）額度：每天 5 萬次讀取、2 萬次寫入——個人筆記用一輩子都用不完，**不需要綁信用卡**。

### 步驟 1：建立 Firebase 專案
1. 用您的 Google 帳號開啟 **console.firebase.google.com**
2. 按「**建立專案**」→ 專案名稱輸入 `yt-memo`（或任何名字）→ 繼續
3. 問要不要 Google Analytics → **關閉**（用不到）→ 建立專案，等它轉完

### 步驟 2：開啟 Firestore 資料庫
1. 左側選單「**建構 (Build)**」→「**Firestore Database**」→「**建立資料庫**」
2. 位置 (Location) 選 **asia-east1（台灣彰化機房）**
3. 模式選「**正式版模式 (production mode)**」→ 建立

### 步驟 3：設定安全規則（只有您本人能讀寫）
1. Firestore 頁面上方點「**規則 (Rules)**」分頁
2. 把內容整個換成下面這段，然後按「**發布 (Publish)**」：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

意思是：只有登入者本人（uid 相符）能碰自己的資料，其他人一律拒絕。

### 步驟 4：開啟 Google 登入
1. 左側「建構」→「**Authentication**」→「開始使用」
2. 「**Sign-in method**」分頁 → 點「**Google**」→ 開關切到「**啟用**」
3. 「專案支援電子郵件」選您自己的信箱 → 儲存

### 步驟 5：加入授權網域（很重要，漏了會登入失敗）
1. Authentication →「**設定 (Settings)**」分頁 →「**授權網域 (Authorized domains)**」
2. 按「新增網域」，輸入：`hsuleon99.github.io` → 新增

### 步驟 6：取得設定值
1. 回到專案首頁，點齒輪 ⚙ →「**專案設定**」
2. 往下捲到「您的應用程式」，點 **`</>`（網頁）** 圖示
3. 應用程式暱稱隨便填（如 `yt-memo-web`），**不要**勾 Firebase Hosting → 註冊應用程式
4. 畫面會出現一段 `const firebaseConfig = { apiKey: "...", ... }` — 這六個值就是您要的

### 步驟 7：貼進 firebase-config.js
1. 打開 `firebase-config.js`，把六個「請貼上您的…」換成剛剛的真實值（照抄，含引號）
2. 存檔，上傳到 GitHub 的 `yt-memo` 資料夾覆蓋

### 步驟 8：各裝置登入
1. 每台裝置打開 App →「更多」→ 最上面「雲端同步」→「🔐 用 Google 登入啟用同步」
2. 選同一個 Google 帳號
3. 首次登入會自動把該裝置的本機資料**合併**上雲端（兩邊都不會少）
4. 看到「☁️ 同步中：您的email」就完成了

之後任何一台裝置新增、修改、刪除、改標籤，其他裝置**即時**跟著變。

### 常見狀況
| 狀況 | 解法 |
|---|---|
| 按登入沒反應／視窗閃退 | 瀏覽器擋了彈出視窗，允許彈出式視窗後再按一次 |
| 登入顯示 unauthorized-domain | 步驟 5 的授權網域沒加到，補加 `hsuleon99.github.io` |
| 想換帳號 | 先「登出」再用另一帳號登入（不同帳號＝不同資料庫，互不相通）|
| 還是想純本機用 | 不設定 firebase-config.js 即可，App 照常運作 |
