# 🏫 課務通 APP · 萬用校園配置開源庫

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com) 
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

本專案旨在透過社群的力量，收集全台灣各大大專院校的校園資訊系統配置檔 (`config.json`)。

## 📲 下載 課務通 APP

| 平台 | 連結 |
|------|------|
| **iOS**（App Store） | [https://apps.apple.com/tw/app/%E8%AA%B2%E5%8B%99%E9%80%9A/id6761055559](https://apps.apple.com/tw/app/%E8%AA%B2%E5%8B%99%E9%80%9A/id6761055559) |
| **Android**（APK） | [https://uni-campus-app.pages.dev/apk/v1.0.1.apk](https://uni-campus-app.pages.dev/apk/v1.0.1.apk) |

你不必懂原生 iOS / Android 開發，**只要會基礎的 JavaScript (DOM 操作或 API Fetch)**，就能透過配置 JSON，讓 **課務通 APP** 完美支援你的學校，提供包含：**自動登入、課表匯入、成績查詢、校園公告、待領信件、常用捷徑** 等強大功能！

---

## 🚀 如何為你的學校新增支援？ (貢獻指南)

我們非常歡迎你為自己的學校提交配置檔！請遵循以下步驟：

1. **下載並打開「課務通 APP」**：進入「設置」>「配置精靈」。
2. **填寫與測試**：依序完成基礎、登入、課表、成績、公告、信件的設定，並務必在 **課務通 APP** 內的 **「沙盒環境 」** 測試通過。
3. **匯出配置碼**：測試無誤後，在最後一步點擊「複製分享配置碼 (JSON)」。
4. **Fork 本專案**：點擊右上角的 Fork 將本儲存庫複製到你的 GitHub。
5. **建立資料夾**：在儲存庫**根目錄**下，以**你學校的網域**為資料夾名稱 (例如：`ncue.edu.tw/`)，與 `config.json` 內 `schoolId` 一致。
6. **上傳檔案**：將剛剛複製的 JSON 存成 `config.json`。
7. **提交 Pull Request (PR)**：發送 PR 給我們，審核通過後，你的學校就會出現在 **課務通 APP** 的支援清單中！🎉

---

## ⚙️ 核心運作邏輯與開發必讀

**課務通 APP** 的底層核心依賴 **WebView 輪詢注入** 與 **API 請求**，請在撰寫腳本時務必遵守以下規範：

### 1. 輪詢頻率 (Polling) 與防連點鎖 (Lock)
課務通 APP 在背景執行任務（如自動登入、爬取課表）時，會 **每 500 毫秒 (500ms)** 將你的腳本注入 WebView 執行一次。
> ⚠️ **極度重要**：請務必在腳本最外層使用鎖定變數（如 `window._isProcessing = true` 或 `sessionStorage`），防止按鈕被瘋狂連點或資料被重複送出！

### 2. 與課務通 APP 通訊 (`postMessage`)
所有的腳本（API 解析 或 WebView 爬蟲），最終都必須透過 `postMessage` 將特定格式的資料回傳給課務通 APP。
**基本語法：**
```javascript
window.mobileApp.postMessage({ 
  detail: { 
    type: '對應的成功訊號', 
    payload: 陣列資料 // 依各模組規定
  } 
});
````

支援的訊號 Type：

  - `AUTH_SUCCESS`：登入成功
  - `DATA_SUCCESS`：課表／成績解析成功
  - `NEWS_SUCCESS`：公告解析成功
  - `MAIL_SUCCESS`：信件解析成功
  - `SESSION_EXPIRED`：憑證過期 (請課務通 APP 重新導向登入頁)
  - `ERROR`：發生錯誤

-----

## 📖 `config.json` 完整規格與撰寫範例

一份完整的 `config.json` 包含以下頂層結構：

```json
{
  "schoolId": "學校網域 (如 ncue.edu.tw)",
  "schoolName": "彰化師範大學",
  "logoPath": "https://.../logo.png",
  "calendar": { "type": "pdf|image|web", "url": "https://..." },
  "auth": { ... },
  "schedule": { ... },
  "score": { ... },
  "news": { ... },
  "mail": { ... },
  "links": [ ... ]
}
```

### 本庫實際配置檔

本儲存庫根目錄已收錄三校之 `config.json`，以下模組範例皆從中摘錄；完整內容請直接打開對應檔案。

| 學校 | `schoolId` | 檔案路徑 |
|------|------------|----------|
| 國立高雄科技大學 | `nkust.edu.tw` | [`nkust.edu.tw/config.json`]|
| 彰化師範大學 | `ncue.edu.tw` | [`ncue.edu.tw/config.json`] |
| 國立臺中科技大學 | `nutc.edu.tw` | [`nutc.edu.tw/config.json`] |

  - **高科大**：課表為 Kendo 表格；成績為 **PDF 非同步解析引擎** (`by_term` 模式)；信件腳本含 **多頁合併**。
  - **彰師大**：課表為學年學期下拉；成績為 **一般 HTML 表格** (`all` 模式)；公告為 API 模式。
  - **臺中科大**：登入後導向 AIS；成績為 **單列包含上下學期之特殊表格** (`all` 模式)；課表讀取全域變數。

-----

### 📆 校曆 (`calendar`)

首頁顯示之行事曆來源。

  * **`type`**：`pdf`、`image` 或 `web`。
  * **`url`**：PDF／圖片網址，或行事曆網頁連結。

<!-- end list -->

```json
"calendar": {
  "type": "pdf",
  "url": "[https://example.edu.tw/path/to/calendar.pdf](https://example.edu.tw/path/to/calendar.pdf)"
}
```

### 🔐 1. 登入模組 (`auth`)

負責處理入口系統的自動登入與驗證。

  * **`entryUrl`**：登入頁或導向登入之入口完整網址。
  * **`loginIndicators` / `successIndicators`**：陣列。網址特徵 `{ "type": "contains|startsWith|exact", "value": "關鍵字" }`。
  * **`usernameSelector` / `passwordSelector` / `submitSelector`**：帳、密、送出鈕的 CSS 選擇器。
  * **`autoFill` / `autoSubmit`**：是否自動填帳密、是否自動送出表單。
  * **`verifyScript`**：(選填) 補強用；例如偵測已出現「登出」且密碼欄已消失時回報 `AUTH_SUCCESS`。

<!-- end list -->

```json
"auth": {
  "entryUrl": "https://...",
  "successIndicators": [ { "type": "contains", "value": "..." } ],
  "loginIndicators": [ { "type": "contains", "value": "..." } ],
  "usernameSelector": "#user",
  "passwordSelector": "#pass",
  "submitSelector": "#loginBtn",
  "autoFill": true,
  "autoSubmit": false,
  "verifyScript": "<見下方腳本範例，或留空字串 \"\">"
}
```

#### `verifyScript` 範例

**範例 A — 偵測登出連結（高科大）：**

```javascript
(function () {
  var logoutBtn = document.querySelector('a[href="/student/Account/Logout"]');
  var pwdInput = document.querySelector('input[name="Password"]');
  if (logoutBtn && !pwdInput && window.mobileApp) {
    window.mobileApp.postMessage({ detail: { type: 'AUTH_SUCCESS' }});
  }
})();
```

**範例 B — 中介頁再導向目標系統（臺中科大）：**

```javascript
(function () {
  try {
    if (location.href.indexOf('MyArea.aspx') !== -1) {
      if (window._nutcRedirecting) return;
      var links = document.querySelectorAll('a');
      for (var i = 0; i < links.length; i++) {
        if (links[i].textContent.trim() === '學生管理系統' && links[i].href.indexOf('Ticket=') !== -1) {
          window._nutcRedirecting = true;
          location.href = links[i].href;
          return;
        }
      }
    }
    if (location.href.indexOf('ais.nutc.edu.tw/student/home.aspx') !== -1) {
      if (window.mobileApp) window.mobileApp.postMessage({ detail: { type: 'AUTH_SUCCESS' }});
    }
  } catch (e) {}
})();
```

-----

### 📅 2. 課表模組 (`schedule`)

處理學生課表爬取，支援自動切換學期。

  * **`syncMode`**：設定為 `auto` (內建背景同步用)。
  * **`entryUrl`**：課表功能入口網址。
  * **`targetUrlIndicators`** / **`sessionExpiredIndicators`**：網址條件陣列。
  * **`timeSlots`**：`[{ "s": "節次代號", "t": "HH:mm~HH:mm" }]`，須與學校節次一致。
  * **`switchScript`**：(選填) 切換學年學期；可用占位符 `{{TARGET_YEAR}}`、`{{TARGET_SEM}}`。
  * **`parseScript`**：解析 DOM，成功後以 `DATA_SUCCESS` 回傳。

> ⚠️ **Payload 強制規格**：`payload` 陣列長度**必須等於** `timeSlots` 數量。每筆為 `{ "slot": "代號", "days": [ 週一～週日共 7 格 ] }`。空堂為 `{}`，有課為 `{ "name", "room", "teacher" }`。

```json
"schedule": {
  "syncMode": "auto",
  "entryUrl": "https://...",
  "targetUrlIndicators": [ { "type": "contains", "value": "..." } ],
  "sessionExpiredIndicators": [ { "type": "contains", "value": "..." } ],
  "timeSlots": [
    { "s": "1", "t": "08:10~09:00" },
    { "s": "2", "t": "09:10~10:00" }
  ],
  "switchScript": "<無需切換學期可為空函式>",
  "parseScript": "<見各校 config.json 原始碼>"
}
```

-----

### 🎓 3. 成績模組 (`score`)

處理學生歷年或單學期成績爬取

  * **`enabled`**：是否啟用成績模組。
  * **`fetchMode`**：
      * `all`：進入頁面後**一次性抓取歷年所有成績**，覆寫本機資料。
      * `by_term`：App 會指定特定學期（如 112-1），並將其取代至 `switchScript` 的 **`{{TARGET_TERM}}`**，抓取後與本機其他學期資料**合併**。
  * **`sessionExpiredIndicators`**：登入憑證過期時需回報 `SESSION_EXPIRED`。
  * **`switchScript`**：分次抓取 (`by_term`) 時必填，用於操作網頁上的下拉選單。
  * **`parseScript`**：解析成績資料，透過 `DATA_SUCCESS` 回報。

> ⚠️ **Payload 強制規格**：必須 `return` 一個陣列。每筆物件為 `{ term, subject, credit, score, isPass: true/false/null, status }`。`isPass: null` 可用於「成績未到」或「修業中」等中立狀態。

#### 成績 `parseScript` 範例

**① 彰師大（`ncue.edu.tw`）— 標準 HTML 歷年表格 (`fetchMode: "all"`)**
此網頁會一次顯示四年所有表格，因此不需 `switchScript`。只需處理前端的強制跳轉與 Bootstrap 彈窗。

```javascript
(function() {
    if (window._hasParsedScore) return;
    var url = location.href;

    // 1. 若在中繼頁則自動點擊跳轉
    if (url.indexOf('LoginSSO') !== -1) {
        var targetLink = document.querySelector('a[href*="SD100/Index"]');
        if (targetLink) targetLink.click();
        return;
    }

    if (url.indexOf('SD100') !== -1) {
        // 2. 點掉阻擋彈窗
        var modal = document.getElementById('mySD100');
        if (modal && modal.classList.contains('show')) {
            modal.querySelector('button[value="確認SD100"]').click();
            return;
        }

        // 3. 抓取表格
        var tables = document.querySelectorAll('#MyDiv table');
        if (tables.length === 0) return; 

        var results = [];
        tables.forEach(function(table) {
            var rows = table.querySelectorAll('tbody tr');
            rows.forEach(function(tr) {
                var tds = tr.querySelectorAll('td');
                if (tds.length >= 7 && tr.innerText.indexOf('無資料') === -1) {
                    var scoreNum = parseInt(tds[6].innerText.trim());
                    results.push({
                        term: tds[0].innerText.trim() + '-' + tds[1].innerText.trim(),
                        subject: tds[4].innerText.trim(),
                        credit: tds[5].innerText.trim(),
                        score: tds[6].innerText.trim(),
                        isPass: isNaN(scoreNum) ? null : (scoreNum >= 60),
                        status: isNaN(scoreNum) ? tds[6].innerText.trim() : (scoreNum >= 60 ? "及格" : "不及格")
                    });
                }
            });
        });

        window._hasParsedScore = true;
        window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: results } });
    }
})();
```

**② 臺中科大（`nutc.edu.tw`）— 上下學期合併在同一列 (`fetchMode: "all"`)**
特殊的表格結構，必須把一列 (tr) 裡面的上學期 (td[5]) 跟下學期 (td[6]) 拆開獨立計算。

```javascript
(function() {
  if (window._hasParsedScore) return;
  var rows = document.querySelectorAll('tr.tr_data');
  if (rows.length === 0) return;

  var results = [];
  rows.forEach(function (row) {
    var tds = row.querySelectorAll('td');
    if (tds.length < 8) return;

    var yearLabel = tds[1].innerText.replace(/[\n\r\s]+/g, '');
    var subject = tds[2].innerText.trim();
    var credit = tds[4].innerText.trim();

    // 內部函式：將上下學期拆開處理
    function processSemester(td, semSuffix) {
      var passStatus = td.getAttribute('data-pass_status');
      if (!passStatus || passStatus === '-') return;

      var term = yearLabel;
      if (yearLabel.indexOf('上') === -1 && yearLabel.indexOf('下') === -1) {
        term = yearLabel.replace('年)', '') + semSuffix + ')'; // 自動補上(上)或(下)
      }

      results.push({
        term: term,
        subject: subject,
        score: td.getAttribute('data-score') || passStatus,
        credit: credit,
        type: tds[3].innerText.trim(),
        isPass: (passStatus.indexOf('P') !== -1 || passStatus.indexOf('T') !== -1) ? true : (passStatus.indexOf('F') !== -1 ? false : null),
        status: passStatus.indexOf('P') !== -1 ? '通過' : '修業中'
      });
    }

    processSemester(tds[5], '上');
    processSemester(tds[6], '下');
  });

  window._hasParsedScore = true;
  window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: results }});
})();
```

**③ 高科大（`nkust.edu.tw`）— PDF 非同步解析引擎 (`fetchMode: "by_term"`)**
因為成績以 PDF 外掛呈現，DOM 抓不到文字。此腳本透過動態載入 `pdf.js`，在背景下載成績單二進位檔並用正則表達式還原分數。因為是分次抓取，所有的 `{{TARGET_TERM}}` 都會由 App 動態替換為實際學期！

```javascript
(function () {
  if (window._isPdfParsing) return;
  var embed = document.querySelector('#pdf embed');
  if (!embed || !embed.src) return;

  window._isPdfParsing = true;
  window.isP = true; // 鎖住 App 輪詢

  window.mobileApp.postMessage({ detail: { type: 'STATUS', payload: '載入 PDF 解析引擎...' } });

  // 動態注入 PDF.js
  var script = document.createElement('script');
  script.src = '[https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js](https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js)';
  document.head.appendChild(script);

  script.onload = function () {
    window.pdfjsLib.GlobalWorkerOptions.workerSrc = '[https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js](https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js)';
    
    fetch(embed.src)
      .then(function (res) { return res.arrayBuffer(); })
      .then(function (buffer) { return window.pdfjsLib.getDocument({ data: buffer }).promise; })
      .then(function (pdf) {
        var promises = [];
        for (var i = 1; i <= pdf.numPages; i++) {
          promises.push(pdf.getPage(i).then(function (page) { return page.getTextContent(); }));
        }
        return Promise.all(promises);
      })
      .then(function (pages) {
        var fullText = pages.map(function (page) {
          return page.items.map(function (item) { return item.str; }).join(' ');
        }).join('\n');

        var results = [];
        // 使用正則表達式精準萃取文字
        var regex = /(?:^|\s)([^\s]{1,40})\s+(必修|選修|通識|博雅)\s+(\d+\.\d+)\s+([0-9]{1,3}|通過|不及格|抵免|退選|免修)/g;
        var match;

        while ((match = regex.exec(fullText)) !== null) {
          var scoreNum = parseInt(match[4]);
          results.push({
            term: '{{TARGET_TERM}}', // APP 會自動替換這個變數
            subject: match[1].trim().replace(/\[.*?\]/g, ''),
            score: match[4],
            credit: match[3],
            type: match[2],
            className: '',
            isPass: !isNaN(scoreNum) ? (scoreNum >= 60) : (match[4] === '通過' || match[4] === '抵免'),
            status: !isNaN(scoreNum) ? (scoreNum >= 60 ? '及格' : '不及格') : match[4]
          });
        }
        window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: results } });
      })
      .catch(function (err) {
        window.mobileApp.postMessage({ detail: { type: 'ERROR', message: 'PDF解析失敗: ' + err.message } });
      });
  };
})();
```

-----

### 📢 4. 公告模組 (`news`) 與 5. 信件模組 (`mail`)

兩者皆支援 **`mode`: `api`** 與 **`mode`: `webview`**。

**共通欄位（節錄）：**

  * **API 模式**：`apiUrl`、`apiMethod`、`apiHeaders`、`apiBody`、`apiRequestScript`、`parseScript`（回傳陣列）。
  * **Webview 模式**：`webviewUrl`、`targetUrlIndicators`、`webviewScript`（於頁面內 `postMessage`）。

> ⚠️ **Payload 規格**：
>
>   - 公告：`[{ title, date, department, link }]`
>   - 信件：`[{ name, department, date, campus }]`

#### 模式 A：API 請求（通用）

透過底層直接發送 HTTP 請求。`apiUrl` / `apiBody` 中可使用 `{{KEY}}`，由 `apiRequestScript` 回傳之物件替換。

```javascript
// parseScript 解析回傳 JSON
var list = [];
(responseData.items || []).forEach(function(n) {
  list.push({ title: n.title, date: n.date, department: n.dept, link: n.url });
});
return list;
```

#### 模式 B：Webview 爬蟲

`webviewUrl` 指向列表頁，於 **`webviewScript`** 內爬取 DOM。

```javascript
(function() {
  if (window._hasParsedNews) return;
  var items = document.querySelectorAll('.d-item');
  if (items.length === 0) return;

  var result = [];
  // ... 略：爬取節點塞入 result

  window._hasParsedNews = true;
  window.mobileApp.postMessage({ detail: { type: 'NEWS_SUCCESS', payload: result } });
})();
```

*(進階技巧：如需跨頁合併資料，可將當前頁資料存入 `sessionStorage` 並呼叫「下一頁按鈕」的 click 事件，直到末頁再統一發送 Payload。範例可參考 `nkust.edu.tw` 信件配置。)*

-----

### 🔗 6. 捷徑連結模組 (`links`)

首頁常用服務，可自訂多群組與樣式。

  * **`title`**：群組標題。
  * **`style`**：`pill`（橢圓標籤）或 `list`（條列）。
  * **`items`**：`title`、`url`、`icon` (可省略)。

<!-- end list -->

```json
"links": [
  {
    "title": "常用服務",
    "style": "pill",
    "items": [
      { "title": "教務系統", "url": "[https://example.edu.tw/](https://example.edu.tw/)", "icon": "book" }
    ]
  }
]
```

-----

## 🤝 貢獻規範 (Code of Conduct)

1.  **不包含惡意腳本**：所有提交的 `webviewScript` 都會經過人工審核，嚴禁任何竊取使用者帳密、Cookie 或發送至第三方伺服器的行為。
2.  **盡量使用 Vanilla JS**：因為是注入到各校不同的網頁中，請盡量使用原生 JavaScript (`document.querySelector`)，避免過度依賴 jQuery，以免舊版網頁不相容。

感謝您的熱心參與！因為有你，開源校園生態系才能更加繁榮！
