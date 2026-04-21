# Security Policy

## 資料安全聲明

本工具為純靜態前端網頁，**不傳送任何資料至伺服器**：

- 所有悠遊卡暱稱、識別碼、卡號僅儲存於使用者**自己瀏覽器的 localStorage**
- 無後端、無資料庫、無 API 呼叫
- 唯一的外部請求為載入 JsBarcode 函式庫（cdn.jsdelivr.net）

## 隱私建議

- 共用裝置請使用完畢後手動清除瀏覽器 localStorage，或使用無痕模式
- 識別碼與卡號屬個人資料，請勿在公共螢幕上截圖分享

## 通報漏洞

若發現安全問題，請透過 GitHub Issues 私下聯繫，或直接開啟 issue 並標記 `security`。

## Supported Versions

| 版本 | 支援狀態 |
|------|---------|
| main branch | ✅ 持續維護 |
