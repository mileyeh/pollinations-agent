# Cute image generator

使用你自己的 Pollinations API Key 免費生成可愛動物圖片。BYOP (Bring Your Own Pollen) 模式，無伺服器端，完全靜態託管。

## 功能特色

- 🔐 Device Flow OAuth 授權（無需後端）
- 🖼️ 多模型選擇：Flux, Turbo, OpenAI, GPT Image, Seedream
- 📐 自定義圖片尺寸（256 - 2048）
- 🌱 固定種子（Seed）重複生成
- 💾 本地歷史記錄（localStorage）
- 📊 即時餘額顯示
- 🎨 簡潔響應式 UI（Tailwind CSS）

## 部署步驟

### 1. 推送 GitHub

```bash
cd cute-generator
git init
git add .
git commit -m "Initial commit - BYOP Device Flow"
gh repo create pollinations-agent --public --source=. --remote=origin
git push -u origin main
```

### 2. Vercel 部署

1. 登入 [Vercel](https://vercel.com)
2. New Project → 選擇你的 `pollinations-agent` 倉庫
3. Framework preset: **Other**
4. Build Command: (留空)
5. Output Directory: (留空)
6. Deploy

### 3. 取得 App URL

部署完成後，URL 類似 `https://pollinations-agent.vercel.app`

### 4. 提交 Pollinations Tier App（可選）

打開 https://github.com/pollinations/pollinations/issues/new?template=tier-app-submission.yml
填寫表單，App URL 填入你的 Vercel URL。

## 使用說明

1. 點擊「開始連接」
2. 在瀏覽器打開顯示的 `enter.pollinations.ai/device`
3. 輸入 6 位驗證碼
4. 登入並授權你的 Pollinations 帳戶
5. 返回页面，狀態自動更新
6. 輸入提示詞，生成可愛動物圖片！

## 注意

- 你的 API Key 只儲存在瀏覽器 localStorage
- 每次授權有效期 30 天，可重新授權
- 圖片消耗 pollen，請確保帳戶有餘額

## 技術棧

- HTML5 + Tailwind CSS (CDN)
- Vanilla JavaScript (ES6+)
- Pollinations Device Flow API

## License

MIT
