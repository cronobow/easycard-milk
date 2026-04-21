# 🥛 悠遊卡換鮮乳條碼產生器

台北市／新北市「生生喝鮮乳」政策輔助工具。忘記帶悠遊卡時，輸入識別碼/學號與16碼卡號，立即生成超商可掃描的 Code 128 一維條碼。

**網址：** [https://gh.maxlab.tw/easycard-milk](https://gh.maxlab.tw/easycard-milk)

---

## 功能

- 輸入識別碼/學號 + 16碼卡號 → 生成兩組 Code 128 條碼
- 16碼卡號輸入時自動每4碼插入空格（儲存時自動去除）
- 支援多筆悠遊卡資料儲存（瀏覽器 localStorage），可重複叫用
- 愛心卡無識別碼時，引導至[悠遊卡多元兌換平台](https://mepweb.easycard.com.tw/portal/about)查詢
- 純靜態，零後端，完全離線可用
- 手機直向優先設計

## 使用方式

1. 開啟 [https://gh.maxlab.tw](https://gh.maxlab.tw)
2. 點「＋ 新增悠遊卡」
3. 填入暱稱、識別碼/學號、16碼卡號
4. 點「儲存並生成條碼」
5. 截圖保存條碼頁，至超商出示掃描

## 如何找到識別碼與卡號

| 號碼 | 位置 |
|------|------|
| 識別碼／學號 | 卡片**正面**右上角或左下角（幼兒數位卡在左下方） |
| 16碼卡號 | 卡片**背面**中央印刷數字 |

> **兒童優惠卡（愛心卡）** 正面無識別碼，請至[悠遊卡多元兌換平台](https://mepweb.easycard.com.tw/portal/about)，選縣市 + 行政區，輸入16碼卡號查詢。

## 資料安全

- 所有資料僅存於**自己裝置的瀏覽器 localStorage**，不傳送至任何伺服器
- 換裝置或清除瀏覽器資料後需重新輸入
- 共用裝置建議使用無痕模式

## 技術規格

| 項目 | 說明 |
|------|------|
| 架構 | 單一 `index.html`（純 Vanilla JS + 內嵌 CSS） |
| 條碼 | [JsBarcode](https://github.com/lindell/JsBarcode) 3.11.6，Code 128 格式 |
| 儲存 | `localStorage`（key: `easycard_profiles`） |
| 部署 | GitHub Pages + 自訂網域（HTTPS） |

## 部署

```bash
git push origin main
```

GitHub Pages 設定：Settings → Pages → Source: `main` / `(root)`

自訂網域：在 User Pages repo（`cronobow.github.io`）設定 CNAME `gh.maxlab.tw`，本 repo 即自動掛載於 `gh.maxlab.tw/easycard-milk`

## License

[MIT](LICENSE) © 2026 cronobow
