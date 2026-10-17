# AGENTS.md — AI Agent 行為準則

本文件說明 AI Agent 在操作此專案時應遵守的規範。任何對此專案的自動化修改、程式碼生成或審查行為，均須先閱讀並遵循以下規則。

---

## 1. 專案結構概覽

```
115-1-MIS-course/
├── app.py                  # Flask 後端主程式（路由、API、Session）
├── software-spec.md        # 系統規格書（問題、功能、限制）
├── AGENTS.md               # 本文件：Agent 行為準則
├── templates/
│   ├── index.html          # 前台報修頁面
│   ├── admin.html          # 管理員後台頁面
│   └── login.html          # 管理員登入頁面
├── static/
│   ├── app.js              # 前台互動邏輯（fetch API）
│   ├── admin.js            # 後台互動邏輯（PATCH / DELETE）
│   └── style.css           # 全站樣式
├── skills/
│   └── project/
│       └── SKILL.md        # 可重複使用的報修功能驗證步驟
├── pyproject.toml          # uv 專案設定
├── requirements.txt        # pip 依賴清單
└── README.md               # 啟動說明
```

---

## 2. 閱讀專案的順序

進入此專案時，Agent 應依以下順序理解系統：

1. **`software-spec.md`**：先了解系統要解決什麼問題、有哪些角色、功能與限制。
2. **`app.py`**：了解後端路由結構、API 端點與驗證邏輯。
3. **`templates/index.html` + `static/app.js`**：了解前台如何呼叫 `POST /api/tickets` 與 `GET /api/tickets`。
4. **`templates/admin.html` + `static/admin.js`**：了解後台如何呼叫 `PATCH` 與 `DELETE`。
5. **`skills/project/SKILL.md`**：執行報修功能驗證前，先閱讀此 Skill。

---

## 3. 禁止修改的檔案

以下檔案**不得在未經明確指示的情況下修改**，因其為課程示範骨架：

| 檔案 | 禁止原因 |
|------|----------|
| `README.md` | 供學生閱讀的操作說明，不應隨意更動 |
| `pyproject.toml` | 依賴管理設定，變更可能破壞環境 |
| `requirements.txt` | 與 `pyproject.toml` 同步，不應單獨修改 |
| `uv.lock` | 自動生成的鎖定檔，絕對不可手動編輯 |

---

## 4. 安全的修改範圍

以下修改是被允許的，且通常安全：

- 在 `app.py` 中**新增**路由或 API 端點（不得刪除或重命名現有端點）。
- 在 `templates/` 中新增 HTML 頁面或修改現有頁面的顯示邏輯（不改變表單欄位名稱）。
- 在 `static/` 中修改 JavaScript 行為或 CSS 樣式。
- 新增或修改 `software-spec.md`、`AGENTS.md`、`skills/` 下的文件。

---

## 5. 程式碼修改規範

1. **保留所有現有中文註解**：`app.py` 中的教學說明註解是課程重點，不得刪除或改寫。
2. **不改變 API 契約**：端點路徑、HTTP 方法、回應格式（JSON 結構）不可在未告知的情況下更動。
3. **不引入新的外部依賴**：除非使用者明確要求，否則不得新增 `pip install` 套件。
4. **不改變驗證邏輯**：`update_ticket` 與 `delete_ticket` 的 Session 驗證不得被移除或繞過。
5. **保持 debug=True**：`app.run(debug=True)` 是教學用設定，不應改為 `False` 或移除。

---

## 6. 執行驗證的方式

每當修改與報修功能相關的程式碼後，Agent 應執行 `skills/project/SKILL.md` 中定義的驗證步驟，確認功能仍正常運作，再結束任務。
