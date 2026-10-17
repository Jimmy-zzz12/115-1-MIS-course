# 校園設備報修系統

這是一個適合 MIS 課堂使用的最小前後端範例。學生透過網頁表單建立報修單，Flask 後端接收 JSON 資料、產生案件編號，再將結果回傳網頁。

## 系統需求

- Windows 10/11
- Python 3.13.x
- 瀏覽器（Edge、Chrome 皆可）

## 啟動方式

### 方法一：使用 uv（推薦）

專案已配置 `pyproject.toml` 與 `.python-version`（指定 Python 3.13）。

若已安裝 `uv`，在專案資料夾直接執行：

```powershell
uv run app.py
```

`uv` 會自動安裝 Python 3.13、建立虛擬環境、同步依賴並啟動 Flask 伺服器。

若需要手動同步依賴或新增套件：
```powershell
uv sync
uv add <套件名稱>
```

---

### 方法二：傳統 Python venv

在此專案資料夾開啟 PowerShell，依序執行：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe app.py
```

第二次以後啟動：

```powershell
.\.venv\Scripts\python.exe app.py
```

---

## 存取與操作

啟動後，以瀏覽器開啟：
- **前台報修頁面**：<http://127.0.0.1:5000>
- **管理後台位址**：<http://127.0.0.1:5000/admin>（進入時會要求驗證身分，預設帳號：`admin`、密碼：`admin`，登入後可查看案件、更新狀態或刪除案件，亦提供登出功能）

按 `Ctrl + C` 可停止伺服器。

## 觀察重點

1. 在 `templates/index.html` 找到表單欄位。
2. 在 `static/app.js` 找到 `fetch("/api/tickets")`：前端在這裡呼叫後端 API。
3. 在 `app.py` 找到 `@app.post("/api/tickets")`：Python 在這裡接收、驗證並回傳資料。
4. 在 `templates/login.html` 與 `app.py` 找到 `/login`、`/logout` 與 Session 驗證機制。
5. 在 `templates/admin.html` 與 `static/admin.js` 看管理員如何透過 `PATCH /api/tickets/<案件編號>` 更新狀態，以及透過 `DELETE /api/tickets/<案件編號>` 刪除案件。

## AI Agent 輔助開發

本專案包含兩份供 AI Agent 使用的輔助文件：

- **`AGENTS.md`**：定義 Agent 操作本專案時應遵守的行為準則，包含禁止修改的檔案清單、安全的修改範圍，以及程式碼修改規範。所有自動化修改均須先閱讀此文件。
- **`skills/project/SKILL.md`**：定義可重複使用的報修功能驗證步驟（共七步），涵蓋建立報修單、必填欄位防呆、案件列表查詢、Session 保護、狀態更新與案件刪除。每次修改報修相關程式碼後，Agent 應執行此 Skill 確認功能仍正常。

## 注意事項

本範例把案件資料存在 Python 的記憶體中，停止或重啟程式後，資料會清空。
