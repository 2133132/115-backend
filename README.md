# 115-backend

後端課程 Agent Runtime 實作紀錄。

## Week 1：Single Call

### 1. 實作內容

- 程式檔案：`week1_single_call.py`
- 使用 Python 的 `requests` 套件呼叫 NVIDIA Chat Completions API。
- 將問題送給 AI 模型，並在終端機顯示 HTTP 狀態碼、JSON 回應與模型回答。
- 本週實作的是**一次模型呼叫（single call）**，尚未實作模型產生 tool call、執行工具並將結果回填的循環。

### 2. HTTP Request 觀察

| 欄位 | 內容 |
| --- | --- |
| Method | `POST` |
| URL | `https://integrate.api.nvidia.com/v1/chat/completions` |
| Authorization | `Bearer` + 從環境變數 `NVIDIA_API_KEY` 讀取的金鑰 |
| Content-Type | `application/json` |
| Model | `minimaxai/minimax-m3.1-flash` |

Request Body 使用 JSON，主要包含 `model`、`messages`、`max_tokens`、`temperature` 和 `top_p`。

其中 `messages` 送出的使用者問題為：「用一句話說明什麼是 HTTP」。

### 3. HTTP Response 與測試紀錄

**第一次測試：401 Unauthorized**

```text
status: 401
{
  "status": 401,
  "title": "Unauthorized",
  "detail": "Invalid JWT serialization: Missing dot delimiter(s)"
}
```

這表示請求已送達伺服器，但授權驗證沒有通過。檢查時發現，一開始複製的憑證不是預期的 `nvapi-` 開頭格式，於是回到 NVIDIA 模型頁面重新產生 API Key。

**第二次測試：200 OK**

```text
status: 200
```

重新設定環境變數後，再執行同一支 Python 程式，得到 `200`，代表 HTTP 請求已成功處理。程式也會使用 `resp.json()` 讀取 JSON 回應，並從 `choices[0].message.content` 顯示模型回覆。

### 4. 除錯與安全措施

- **錯誤來源：** 這次遇到的是驗證憑證問題，而不是 Python 語法錯誤。
- **處理方式：** 檢查 API Key 的基本格式、重新產生金鑰、更新 PowerShell 的環境變數後重試。
- **安全措施：** 透過 `os.environ["NVIDIA_API_KEY"]` 讀取金鑰，沒有在程式內寫死金鑰，也不將完整金鑰加入 GitHub 或紀錄中。
- **執行環境：** Python 3.12.11；使用虛擬環境安裝 `requests`。

### 5. 小結

本週完成一次 HTTP POST 模型呼叫，實際觀察到 `401` 與 `200` 的差異，也練習根據伺服器回傳的錯誤訊息排查問題。
