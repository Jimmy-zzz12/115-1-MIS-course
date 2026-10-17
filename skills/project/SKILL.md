# SKILL: 驗證報修功能

本 Skill 定義一組可重複使用的驗證步驟，用於確認「校園設備報修系統」的報修相關 API 功能是否正常運作。每次修改報修相關程式碼後，均應執行以下所有步驟。

---

## 前置條件

執行驗證前，確認以下事項：

- Flask 伺服器已啟動（`uv run app.py` 或 `python app.py`）
- 伺服器正在監聽 `http://127.0.0.1:5000`

---

## 步驟一：驗證建立報修單（正常流程）

**目標**：確認送出完整欄位時，後端正確建立案件並回傳 `201`。

**操作**：
```
POST http://127.0.0.1:5000/api/tickets
Content-Type: application/json

{
  "location": "資訊大樓 302",
  "equipment": "投影機",
  "description": "無法開機",
  "contact": "王小明 / 0912-345-678",
  "urgency": "急件"
}
```

**預期結果**：
- HTTP 狀態碼：`201 Created`
- 回應 JSON 包含 `"message"` 與 `"ticket"` 兩個欄位
- `ticket.id` 格式為 `T-001`（或遞增編號）
- `ticket.status` 為 `"待處理"`
- `ticket.created_at` 格式為 `YYYY-MM-DD HH:MM`

---

## 步驟二：驗證必填欄位缺失時的防呆機制

**目標**：確認缺少必填欄位時，後端正確拒絕並回傳 `400`。

**操作（測試缺少 `contact`）**：
```
POST http://127.0.0.1:5000/api/tickets
Content-Type: application/json

{
  "location": "資訊大樓 302",
  "equipment": "冷氣",
  "description": "不製冷"
}
```

**預期結果**：
- HTTP 狀態碼：`400 Bad Request`
- 回應 JSON 的 `"message"` 欄位中包含缺少欄位名稱（如「聯絡方式」）

**同樣測試以下缺少欄位的情境**：
- 缺少 `location` → message 含「地點」
- 缺少 `equipment` → message 含「設備」
- 缺少 `description` → message 含「問題描述」
- 缺少 `contact` → message 含「聯絡方式」

---

## 步驟三：驗證案件列表查詢

**目標**：確認 `GET /api/tickets` 能正確回傳剛才建立的案件。

**操作**：
```
GET http://127.0.0.1:5000/api/tickets
```

**預期結果**：
- HTTP 狀態碼：`200 OK`
- 回應為 JSON 陣列
- 陣列中包含步驟一建立的案件（以 `id` 對應）

---

## 步驟四：驗證未登入時無法更新案件狀態

**目標**：確認 `PATCH` 端點的 Session 保護有效。

**操作（不帶 Session Cookie）**：
```
PATCH http://127.0.0.1:5000/api/tickets/T-001
Content-Type: application/json

{ "status": "處理中" }
```

**預期結果**：
- HTTP 狀態碼：`401 Unauthorized`
- 回應 JSON 的 `"message"` 含「未授權」

---

## 步驟五：驗證登入後可正常更新案件狀態

**目標**：確認管理員登入後，`PATCH` 端點能正確更新狀態。

**操作**：
1. 先以 `POST /login`（表單）或直接用瀏覽器登入管理後台（帳號：`admin`，密碼：`admin`），取得有效 Session。
2. 帶著 Session Cookie 送出：
   ```
   PATCH http://127.0.0.1:5000/api/tickets/T-001
   Content-Type: application/json

   { "status": "處理中" }
   ```

**預期結果**：
- HTTP 狀態碼：`200 OK`
- 回應 JSON 的 `ticket.status` 為 `"處理中"`

**同樣測試其他合法狀態值**：
- `"待處理"` → `200 OK`
- `"已完成"` → `200 OK`
- `"其他文字"` → `400 Bad Request`（狀態不合法）

---

## 步驟六：驗證刪除案件

**目標**：確認管理員登入後，`DELETE` 端點能正確刪除案件。

**操作（帶 Session Cookie）**：
```
DELETE http://127.0.0.1:5000/api/tickets/T-001
```

**預期結果**：
- HTTP 狀態碼：`200 OK`
- 回應 JSON 含「已成功刪除」訊息
- 再次呼叫 `GET /api/tickets`，確認該案件已不存在於列表中

---

## 步驟七：驗證刪除不存在的案件

**操作**：
```
DELETE http://127.0.0.1:5000/api/tickets/T-999
```

**預期結果**：
- HTTP 狀態碼：`404 Not Found`
- 回應 JSON 含「找不到指定案件」

---

## 驗證完成標準

所有步驟均通過後，方可視為報修功能驗證完成。若有任何步驟失敗，應回頭檢查 `app.py` 中對應的函式，並參照 `software-spec.md` 確認預期行為。
