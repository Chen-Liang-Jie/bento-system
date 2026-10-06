# 鈺見自然便當店 - 響應式點餐與門市管理系統

店名：鈺見自然便當店（UI 標題、頁面 `<title>`、報表檔名等一律使用此店名）。

響應式點餐與門市管理系統，採無伺服器架構：GitHub Pages 免費靜態託管 + Supabase 雲端資料庫與 Realtime 即時推播。目標是 $0 月租、0% 抽成，店家電腦關機後系統仍能 24 小時在雲端運作。

## 技術棧

- 單一頁面應用：`index.html`
- 透過 CDN `<script>` 引入（Live Server 右鍵開啟與 GitHub Pages 皆須能直接運作，不依賴 build）：
  - Vue 3
  - Tailwind CSS
  - @supabase/supabase-js v2
  - SheetJS xlsx：一律使用官方 CDN `https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js`（npm registry 上的 0.18.5 有已知漏洞，勿使用）
  - qrcode.js
- 本機 `node_modules` 已透過 npm 安裝相同套件，僅供 VS Code 語法提示與未來模組化（Vite）擴充；新增 CDN 引入時版本需與 `package.json` 一致。例外：
  - Tailwind：CDN 用 v3.4.17 Play CDN（才支援 `tailwind.config` 寫法），npm 裝的是 v4；改 Vite 打包時需改用 v4 的 `@theme` 寫法。
  - QR Code：CDN 用 `qrcodejs`（davidshimjs，全域 `QRCode`），npm 裝的 `qrcode` 是不同套件、API 不同。

## 資料庫 RPC（定義於 `supabase_init.sql`）

- `create_bento_order`：下單。`p_items` 格式 `[{ product_id, qty, options: ["飯少", ...] }]`，回傳 `{ order_id, order_no, total_amount }`。
- `cancel_bento_order(p_order_id)`：取消訂單並歸還庫存。
- `adjust_product_stock(p_product_id, p_delta)`：以加減量原子調整庫存（歸 0 自動停售、從 0 補貨自動上架）。
- 本機展示模式（`index.html` 的 `makeLocalApi`）需與上述 RPC 的邏輯保持一致。

## 核心業務規則

1. 訂單佇列必須嚴格依照訂餐時間從早到晚（`created_at ASC`）排序。
2. 具備「本機展示模式（LocalStorage）」與「雲端連線模式（Supabase）」雙軌自動切換機制，方便隨時測試。
3. 價格計算與庫存扣除在雲端模式下必須透過 PostgreSQL RPC 函數 `create_bento_order` 進行悲觀鎖（`FOR UPDATE`）防超賣驗證，禁止信任前端傳入的金額。
4. 店家後台需具備 4 位數 PIN 碼鎖（預設 `8888`），防止顧客誤觸後台。
5. 所有 UI 元件需保持高可讀性、超大按鈕（適合長輩與餐期單指操作），主題色集中於 Tailwind config。

## 開發進度

- 階段 0：工具與 npm 套件已安裝；GitHub CLI 已登入帳號 `Chen-Liang-Jie`；Git 作者為 JayChen。
- 階段 1：已 `git init`（分支 `main`），建立 `.gitignore`、`.vscode/settings.json`、本檔。
- 階段 2：已建立 `supabase_init.sql`、`.github/workflows/keep-alive.yml`。SQL 只做過語法檢查，尚未在 Supabase 實際執行；使用者執行後應提供測試用 SQL 驗證 RPC。
- 階段 3：已完成 `index.html`（本機展示模式經瀏覽器端對端測試通過）。
- 階段 4：已用 CLI 建立 Supabase 組織 `Yujian Bento`、專案 `bento-system`（ref `mqcoayjbpghuwxvdpeda`，ap-northeast-1 東京），`supabase_init.sql` 已執行，URL 與 anon key 已寫入 `index.html`；以 anon key 測試讀取、下單、超賣防護、取消歸還庫存皆通過。DB 密碼與金鑰存於 `.env`（已 gitignore）。
- CLI 注意：在 Claude Code 環境下 Supabase CLI 會自動切成 JSON 非互動模式，指令需加 `--agent no`；執行 SQL 用 `npx supabase db query --file <檔案> --linked --project-ref mqcoayjbpghuwxvdpeda --agent no`。
- 階段 4（部署）：已推送至 GitHub 公開儲存庫 https://github.com/Chen-Liang-Jie/bento-system ，GitHub Pages（main 分支根目錄）上線於 https://chen-liang-jie.github.io/bento-system/ 。keep-alive 的 `SUPABASE_URL`、`SUPABASE_ANON_KEY` 已設為 GitHub Secrets，手動觸發測試成功。
- gh 注意：gh 安裝於 `C:\Program Files\GitHub CLI\gh.exe`，若終端機找不到 gh 需用完整路徑或重開 VS Code。
- 下一步：等待使用者提供下一階段指令。

## 開發注意事項

- `.vscode/settings.json` 已設定 Live Server 埠號 5500，並忽略 `.vscode/**`、`**/*.sql`、`**/*.md`、`.git/**` 的變更，避免不必要的重新整理。
- 機密資訊（如 Supabase service_role key）不得寫入前端或提交至 Git；前端只能使用 anon key，資料存取權限交由 RLS 控管。
