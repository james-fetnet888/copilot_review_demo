# GitHub Copilot Code Review 指引 (Code Review Instructions)

當執行此專案的 Pull Request (PR) 程式碼審查時，請遵循以下規則與最佳實踐：

## 1. 審查語言與溝通風格 (Review Language & Communication Style)
* **語言限制**：所有審查意見與改善建議均須使用**繁體中文 (zh-TW)** 撰寫。
* **語氣態度**：保持友善、具建設性且專業的態度。針對有待改進的部分，應明確指出原因並提供示範程式碼（如有必要）。

## 2. Azure Functions (Node.js/TypeScript v4) 開發規範
* **API 設定**：本專案使用 Azure Functions v4 程式庫 (`@azure/functions`)。所有的 HTTP Trigger 均應透過 `app.http()` 進行註冊，且 handler 簽章必須符合 `(request: HttpRequest, context: InvocationContext): Promise<HttpResponseInit>`。
* **回應格式**：
  * 對外 API 應使用標準的回應格式，建議配合專案中的回應輔助工具（例如 `responseMsg`）。
  * 審查時請檢查是否正確回傳對應的 HTTP Status Code（例如 `200`, `400`, `401`, `500` 等），並在 Response Body 中包含清晰的自訂 `status_code` 與 `message`。

## 3. Cosmos DB 安全性與效能
* **防範 SQL 注入 (SQL Injection)**：任何針對 Azure Cosmos DB 的 SQL 查詢，**嚴禁**使用字串插值 (string interpolation) 或字串拼接來嵌入變數。必須使用 parameterized query（例如使用 `@uid` 並於 `parameters` 陣列中帶入數值）。
* **連線池複用**：請勿在各個 Handler 中使用 `new CosmosClient()` 重複建立連線。必須統一使用 `src/libs/cosmosClient.ts` 中提供的快取連線方法（如 `getCosmosContainer` 或 `getCosmosDatabase`），以防 Socket 耗盡 (Socket Exhaustion)。

## 4. 時間與時區處理
* **台北時區**：由於 Azure Functions 伺服器預設時間可能為 UTC，專案中涉及時間格式化、計算或比對時，必須明確轉換為台北時區（`Asia/Taipei`），例如使用 `dayjs().tz("Asia/Taipei")`，以確保業務邏輯時間一致。

## 5. TypeScript 型別安全
* **避免 `any`**：審查程式碼時，應積極避免使用 `any` 型別。請儘可能定義明確的 interface、type 或 generic 參數。
* **輸入參數驗證**：對於 Request Body 或 Query Parameters 中傳入的外部參數，應有嚴格的必填驗證 (validation) 與型別檢查，避免 runtime 錯誤。

## 6. 錯誤處理與日誌記錄 (Error Handling & Logging)
* **日誌紀錄**：應使用 `context.log`、`context.warn`、`context.error` 進行日誌紀錄，避免使用未受管理的 `console.log`。
* **安全錯誤處理**：在 `try-catch` 區塊中，避免直接將原始的錯誤訊息（包含系統底層 Stack Trace 或敏感連線字串）直接回傳給前端客戶端。應予以捕獲、記錄日誌並回傳友善的錯誤代碼。
