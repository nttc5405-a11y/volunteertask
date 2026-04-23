# 志工服務管理系統 — Google Sheets 結構 & 部署指引

---

## 一、Sheets 工作表結構

### 📋 volunteers（志工主檔）
| 欄位 | 說明 | 範例 |
|------|------|------|
| vol_id | 系統自動產生 | V-1A2B3C |
| name | 姓名 | 陳志明 |
| id_number | 身分證字號 | A123456789 |
| phone | 手機號碼 | 0912345678 |
| email | 電子信箱 | chen@email.com |
| address | 居住地址 | 台東市中山路1號 |
| emergency_name | 緊急聯絡人 | 陳小明（父） |
| emergency_phone | 緊急聯絡電話 | 0922111222 |
| group_id | 所屬群組 ID | G-XXXXXX |
| status | 待命 / 上工中 / 忙碌 / 休息 | 待命 |
| role | volunteer / admin | volunteer |
| total_hours | 累計服務時數 | 42 |
| level | bronze / silver / gold | silver |
| join_date | 加入日期 | 2025-04-01 |
| notes | 備註 | — |
| password_hash | 加密後密碼（勿手動修改）| h1a2b3c4 |

---

### 🗂️ groups（群組）
| 欄位 | 說明 |
|------|------|
| group_id | 系統產生 |
| group_name | 群組名稱 |
| icon | Emoji 圖示 |
| leader | 負責人姓名 |
| member_count | 成員數（自動計算）|
| description | 群組說明 |
| status | active / inactive |
| created_date | 建立日期 |

---

### 📋 cases（案件）
| 欄位 | 說明 |
|------|------|
| case_id | 系統產生 |
| title | 案件名稱 |
| location | 任務地點 |
| case_date | 任務日期 YYYY-MM-DD |
| case_time | 任務時間 HH:MM |
| required_count | 需求人數 |
| accepted_count | 已接受人數（自動更新）|
| priority | normal / high / urgent |
| status | 待派案 / 人員確認中 / 執行中 / 已結案 |
| assigned_group | 指派群組 ID |
| description | 任務說明 |
| creator | 建立者 |
| created_date | 建立時間 |
| notes | 備註 |

---

### 📍 checkin（報到記錄）
| 欄位 | 說明 |
|------|------|
| record_id | 系統產生 |
| vol_id | 志工 ID |
| vol_name | 志工姓名（快查用）|
| checkin_time | 報到時間 ISO 格式 |
| checkout_time | 結束時間 ISO 格式（空白=仍上工）|
| hours | 計算時數（結算後填入）|
| case_id | 關聯案件 |
| task | 任務說明 |
| verified | TRUE / FALSE |
| date | 日期 YYYY-MM-DD |

---

### 🎯 skills（專長，多對多）
| 欄位 | 說明 |
|------|------|
| vol_id | 志工 ID |
| skill | 專長名稱（如：急救CPR）|
| updated_date | 最後更新時間 |

> 同一志工可有多列（每項專長一列）

---

### 📢 messages（廣播訊息）
| 欄位 | 說明 |
|------|------|
| msg_id | 系統產生 |
| type | emergency / normal / remind |
| title | 訊息標題 |
| content | 訊息內容 |
| target | all / group_id |
| sender | 發送者姓名 |
| sent_date | 發送時間 ISO 格式 |
| read_count | 已讀人數 |

---

### 💬 replies（案件回覆）
| 欄位 | 說明 |
|------|------|
| reply_id | 系統產生 |
| case_id | 關聯案件 |
| vol_id | 志工 ID |
| vol_name | 志工姓名 |
| reply_type | accept / reject / ask |
| content | 回覆內容 |
| replied_date | 回覆時間 |

---

### 📝 log（操作記錄）
| 欄位 | 說明 |
|------|------|
| log_id | 系統產生 |
| action | 操作名稱（如：addCase）|
| operator | 操作者 |
| target_id | 目標 ID |
| detail | 詳細說明 |
| timestamp | 操作時間 |

---

## 二、GAS 部署步驟

### Step 1：建立 Google Sheets
1. 開啟 https://sheets.new 建立新試算表
2. 命名為「志工服務管理系統」
3. 記下試算表 URL 中的 Sheet ID（`/d/SHEET_ID/edit` 這段）

### Step 2：建立 Apps Script
1. 點選選單：**延伸功能 → Apps Script**
2. 將 `gas-backend.js` 的全部內容貼入（取代原有的 `myFunction`）
3. 修改設定區：
   ```javascript
   const CONFIG = {
     TOKEN: 'vol_2025_自訂密碼',   ← 設定一個安全字串
     SHEET_ID: '',                  ← 留空（會自動使用當前試算表）
   };
   ```

### Step 3：初始化工作表
1. 在 Apps Script 編輯器選擇函式 `initAllSheets`
2. 點擊 **執行**（▶️）
3. 首次執行需要授權，請同意所有權限
4. 等待確認視窗顯示「所有工作表初始化完成」

### Step 4：部署為網頁應用程式
1. 點擊右上角 **部署 → 新增部署**
2. 選擇類型：**網頁應用程式**
3. 設定：
   - 說明：志工系統後端
   - 執行身分：**我（你的 Google 帳號）**
   - 存取權限：**所有人** ← 這步驟非常重要！
4. 點擊 **部署**
5. 複製「網頁應用程式 URL」

### Step 5：設定前端
1. 開啟 `volunteer-system-v2.html`
2. 點擊頂部黃色提示條或右上角設定
3. 填入：
   - GAS 部署 URL：`https://script.google.com/macros/s/XXXX/exec`
   - Token：`vol_2025_自訂密碼`（與 Step 2 相同）
4. 點「測試連線」確認 ✅
5. 儲存設定

---

## 三、API 端點說明

### GET（讀取）
```
GET {GAS_URL}?action=XXX&token=TOKEN&...

action=getVolunteers   取得所有志工（可加 status= 過濾）
action=getCases        取得所有案件（可加 status= 過濾）
action=getGroups       取得所有群組
action=getCheckin      取得報到記錄（可加 vol_id= 過濾）
action=getMessages     取得廣播訊息
action=getReplies      取得案件回覆（可加 case_id= 過濾）
action=getStats        取得統計數字
action=ping            測試連線
```

### POST（寫入）
```
POST {GAS_URL}
Content-Type: text/plain  ← 避免 CORS 預檢問題
Body: { "action":"XXX", "token":"TOKEN", "data":{...} }

action=login            志工登入驗證
action=addVolunteer     新增志工
action=updateVolunteer  更新志工資料
action=addCase          新增案件
action=updateCase       更新案件
action=checkIn          志工報到
action=checkOut         志工結束上工
action=updateStatus     更新志工狀態
action=addReply         新增案件回覆
action=addMessage       新增廣播訊息
action=addGroup         新增群組
```

---

## 四、初始管理員設定

1. 在 `volunteers` 工作表中手動新增一列：
   - name：管理員
   - phone：0900000000（你的手機）
   - role：**admin**
   - password_hash：在 Apps Script 執行 `hashSimple('你的密碼')` 取得

2. 或在 Apps Script 執行 `seedDemoData()` 產生示範資料

---

## 五、注意事項

| 項目 | 說明 |
|------|------|
| CORS | POST 使用 `text/plain` 避免預檢問題（同火場系統做法）|
| 存取權限 | 必須設「所有人」，否則 GitHub Pages / 本機無法呼叫 |
| 密碼 | 目前用簡易 hash，正式環境建議加 salt 或改用 SHA-256 |
| 重新部署 | 每次修改 GAS 程式碼後需「管理部署 → 編輯 → 部署新版本」|
| Token 更換 | 直接修改 CONFIG.TOKEN 並重新部署，前端也同步修改 |
