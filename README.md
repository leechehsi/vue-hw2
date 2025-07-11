# YouBike 站點收藏小工具

這是一個使用 Vue 3 + Composition API + Axios 所撰寫的小工具，可以即時查詢 YouBike 站點資訊、搜尋特定站點，並加入到本地收藏清單（使用 localStorage 永久保存）。

## 🔧 功能介紹

- ✅ 即時取得台北市 YouBike 2.0 公開 API 資料
- 🔍 搜尋站點（可依站名或地址）
- 💾 收藏站點資訊（儲存在 localStorage）
- 👁️ 點擊站點可展開詳細資訊（地址、可租借/歸還數量）
- ♻️ 收藏清單可取消收藏

## 📦 安裝與執行

### ✅ 安裝依賴

```bash
npm install
```

### ▶️ 開發模式執行

```bash
npm run dev
```

### 🔨 打包生產環境

```bash
npm run build
```
