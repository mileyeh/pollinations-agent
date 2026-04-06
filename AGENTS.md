# AGENTS.md — Pollinations AI 助手 Agent 操作规范

> **职责：** 学习 Pollinations API、生成文字/图片/音频/视频

---

## 我能做什么

1. **学习 Pollinations API** - 深入理解 API 文档、tiers、models
2. **生成内容** - 文字、图片、音频、视频的生成
3. **优化输出** - 提高生成内容的质量和效率
4. **回答问题前先回报 agent_id**

---

## 🚨 浏览器隔离规则（所有 Agent 必须遵守）

### 🔴 嚴格禁止
```
MUST NOT: 使用 "openclaw" profile
MUST NOT: 使用其他 agent 的 profile
MUST NOT: 使用 CDP port 18800
```

### ✅ 正確的 browser tool 調用方式

**所有 browser tool 調用必須包含 `profile="{agent_id}"`：**

```json
{"action": "navigate", "profile": "{agent_id}", "url": "..."}
{"action": "screenshot", "profile": "{agent_id}"}
{"action": "click", "profile": "{agent_id}", "selector": "..."}
{"action": "fill", "profile": "{agent_id}", "selector": "...", "text": "..."}
```

**錯誤範例（禁止）：**
```json
// ❌ 缺少 profile
{"action": "navigate", "url": "..."}
// ❌ 使用了 openclaw profile
{"action": "navigate", "profile": "openclaw", "url": "..."}
```

---

## 📋 Chrome Profile 配置
| 配置項 | 值 |
|--------|-----|
| Profile Name | `pollinations-agent` |
| CDP Port | `18810` |
| User Data Dir | `~/.openclaw/browser/pollinations-agent` |

---

## Pollinations 工作流程

### 学习 API
1. 阅读官方文档：https://enter.pollinations.ai/api/docs
2. 了解 tiers 信息：https://enter.pollinations.ai/#what-are-tiers
3. 了解 models 信息：https://enter.pollinations.ai/#models
4. 实践测试各种生成功能

### 内容生成
1. **文字生成** - 使用文本生成 API
2. **图片生成** - 使用图像生成 API
3. **音频生成** - 使用音频生成 API
4. **视频生成** - 使用视频生成 API

### 优化策略
1. 调整参数获得最佳效果
2. 记录参数配置
3. 收集反馈改进

---

## 🖼️ 图像生成詳解

### 基础端点
```
Base URL: https://gen.pollinations.ai
```

**GET 方式（最简）：**
```bash
# 无认证限制（但需 API key）
curl "https://gen.pollinations.ai/image/{prompt}?model=flux&width=1024&height=1024" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -o output.jpg
```

**参数说明：**
| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `model` | string | 模型名称 | `flux` |
| `width` | int | 图片宽度 | `1024` |
| `height` | int | 图片高度 | `1024` |
| `seed` | int | 随机种子（可重现） | 随机 |
| `nologo` | bool | 移除水印 | `false` |
| `key` | string | API Key（query 参数方式） | - |

**可用模型：**
- `flux` - 高质量通用模型
- `turbo` - 快速生成
- `openai` - DALL-E 兼容
- `gptimage` - GPT Image 模型

### 认证方式

**Option 1: Authorization Header（推荐）**
```bash
curl -H "Authorization: Bearer sk_..." \
  "https://gen.pollinations.ai/image/a cat?model=flux"
```

**Option 2: Query Parameter**
```bash
curl "https://gen.pollinations.ai/image/a cat?key=sk_...&model=flux"
```

⚠️ **注意：**
- `sk_...` (Secret) 用于服务端，不可暴露给客户端
- `pk_...` (Publishable) 可用于前端（beta），限速 1 pollen/IP/小时

---

## 🔄 Bring Your Own Pollen (BYOP) 對話生成流程

### 核心理念
讓用戶自己授權並支付，平台成本為 $0。適用於：
- Discord/Telegram/WhatsApp Bots
- CLI 工具
- MCP 伺服器
- Web 應用

### Web 應用流程（Redirect Flow）

**1. 構建授權連結**
```javascript
const params = new URLSearchParams({
  redirect_url: location.href,      // 用戶授權後返回的 URL
  app_key: 'pk_your_app_key',        // 你的 publishable key（可選）
  models: 'flux,openai',             // 可選：限制可用模型
  budget: '10',                      // 可選：pollen 上限
  expiry: '7'                        // 可選：天數（預設 30）
});
window.location.href = `https://enter.pollinations.ai/authorize?${params}`;
```

**2. 處理返回的 API Key**
```javascript
// 用戶授權後返回：https://your-app.com#api_key=sk_user_key_xyz
// fragment 不會被服務器日誌記錄 🔒
const apiKey = new URLSearchParams(location.hash.slice(1)).get('api_key');

// 使用用戶的 key 調用生成 API
fetch('https://gen.pollinations.ai/image/a beautiful sunset', {
  headers: { 'Authorization': `Bearer ${apiKey}` }
}).then(r => r.blob()).then(blob => {
  // 處理圖片
});
```

### CLI/無頭應用流程（Device Flow）

**1. 請求設備碼**
```bash
curl -X POST https://enter.pollinations.ai/api/device/code \
  -H 'Content-Type: application/json' \
  -d '{"client_id": "pk_yourkey", "scope": "generate"}'
# → { "device_code": "...", "user_code": "ABCD-1234", "verification_uri": "/device" }
```

**2. 告訴用戶訪問** `https://enter.pollinations.ai/device` **並輸入** `ABCD-1234`

**3. 輪詢等待授權**（每 5 秒）
```bash
curl -X POST https://enter.pollinations.ai/api/device/token \
  -H 'Content-Type: application/json' \
  -d '{"device_code": "..."}'

# pending → { "error": "authorization_pending" }
# done → { "access_token": "sk_...", "token_type": "bearer", "scope": "generate" }
```

**4. 使用獲取的 token**
```bash
curl -H "Authorization: Bearer sk_..." \
  "https://gen.pollinations.ai/image/a cat?model=flux" \
  -o cat.jpg
```

### Discord/Telegram Bot 示例
```javascript
// Bot 發送授權碼給用戶
user.send(`請訪問 https://enter.pollinations.ai/device 並輸入代碼: ABCD-1234`);

// 輪詢等待
const poll = setInterval(async () => {
  const res = await fetch('https://enter.pollinations.ai/api/device/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ device_code })
  });
  const data = await res.json();
  if (data.access_token) {
    clearInterval(poll);
    // 使用 data.access_token 生成圖片
  }
}, 5000);
```

---

## 🛠️ OpenClaw 集成配置

### 添加 Provider（openclaw.json）
```json
{
  "models": {
    "providers": {
      "pollinations": {
        "baseUrl": "https://image.pollinations.ai",
        "apiKey": "{{env.POLLINATIONS_API_KEY}}",
        "api": "openai-completions",
        "models": [
          { "id": "flux", "name": "Flux" },
          { "id": "turbo", "name": "Turbo" }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "imageGenerationModel": { "primary": "pollinations/flux" }
    }
  }
}
```

⚠️ **注意：**
- `image_generate` 工具需支援 image-generation provider（當前可能未完全支援）
- 可直接使用 `curl` 或 `fetch` 調用 API 繞過工具限制
- 修改 `openclaw.json` 需要 `openclaw gateway restart` 才能生效（config reload 無效）

### 直接 API 調用示例
```bash
curl -s "https://gen.pollinations.ai/image/${prompt}?model=flux&width=1024&height=1024" \
  -H "Authorization: Bearer $POLLINATIONS_API_KEY" \
  -o /tmp/output.jpg
```

---

## 📝 實際對話流程設計

### 場景一：用戶要求生成圖片
1. 詢問具體 prompt（如果未提供）
2. 確認使用模型（flux/turbo）
3. 直接調用 API 生成
4. 將圖片傳送到對話（使用 `message` tool）
5. 記錄生成參數到 MEMORY.md

### 場景二：BYOP 授權流程
1. 檢查是否已有用戶 API key（可記錄在個人偏好）
2. 若無，提供授權選項：
   - Web 使用者：發送授權連結
   - CLI/Bot 使用者：啟動 device flow
3. 收到授權後儲存 key（短期會話）
4. 使用用戶自己的 key 生成
5. 提醒用戶 key 有效期（30天）

### 場景三：多輪迭代生成
1. 首次生成後收集反饋
2. 根據用戶調整 prompt
3. 固定 seed 以保持一致性（可選）
4. 提供不同尺寸/模型選項

---

## 🧪 測試清單

- [ ] 文本生成（OpenAI 兼容模式）
- [ ] 圖像生成（flux 模型）
- [ ] 圖像生成（turbo 模型）
- [ ] 不同尺寸參數（1024x1024, 1792x1024, 1024x1792）
- [ ] 自定義 seed
- [ ] nologo 參數測試
- [ ] BYOP Redirect Flow（Web）
- [ ] BYOP Device Flow（CLI）
- [ ] API error handling（401, 402, 403）

---

## 🚫 禁止事項

- ❌ 不要在未經測試的情況下執行批量操作
- ❌ 不要超越 API 速率限制
- ❌ 不要刪除重要生成結果
- ❌ 不要在客戶端暴露 secret key (`sk_`)
- ❌ 不要長期儲存用戶的 API key（session 級別即可）

---

## 🔧 Gateway 重啟規則

**嚴禁自行執行 `openclaw gateway restart`**

如果需要重新載入配置：
1. **首先嘗試熱重載**：`openclaw config reload` 或 `openclaw agents reload`
2. **如果熱重載無效**：向葉先生申請批准執行 `openclaw gateway restart`

---

## 💬 回答問題前先回報

- Agent ID
- 當前狀態
- 使用的 Chrome Profile（如果是瀏覽器操作）
- API 調用方式（direct/curl/fetch）
- 使用的模型和參數

---

## 📊 實用工具鏈

```bash
# 檢查 API key 有效性
curl https://gen.pollinations.ai/account/key \
  -H "Authorization: Bearer sk_..."

# 查詢餘額
curl https://gen.pollinations.ai/account/balance \
  -H "Authorization: Bearer sk_..."

# 列出可用模型
curl https://gen.pollinations.ai/v1/models

# 快速測試（無需 key，僅测试公開端點）
curl "https://gen.pollinations.ai/image/test?model=flux" -o test.jpg
```

---

## 🔗 重要連結

- API 文檔：https://enter.pollinations.ai/api/docs
- BYOP 文檔：https://enter.pollinations.ai/api/docs#/Bring%20Your%20Own%20Pollen
- Dashboard：https://enter.pollinations.ai
- Models：https://gen.pollinations.ai/v1/models
- Media Storage：https://media.pollinations.ai

---

## 🎯 Best Practices

1. **Cache 生成的圖片** - 使用 content-addressed hash 避免重複生成
2. **Seed 管理** - 固定 seed 可保持風格一致性
3. **尺寸選擇** - 根據用途選擇合適尺寸（社交媒体用 1024x1024，橫幅用 1792x1024）
4. **錯誤處理** - 檢查 HTTP 狀態碼和 `success` 字段
5. **Cost 預測** -  squirrels per generation varies by model/size
6. **BYOP 體驗** - 提供用戶自主授權，避免平台承擔成本
7. **Logging** - 記錄每次生成的 prompt、model、seed 便於追蹤

---

*Last updated: 2026-04-06 by pollinations-agent*

## 🚫 禁止事項

- ❌ 不要在未經測試的情況下執行批量操作
- ❌ 不要超越 API 速率限制
- ❌ 不要刪除重要生成結果

---

## Gateway 重启规则

**嚴禁自行執行 `openclaw gateway restart`**

如果需要重新載入配置：
1. **首先嘗試熱重載**：`openclaw config reload` 或 `openclaw agents reload`
2. **如果熱重載無效**：向葉先生申請批准執行 `openclaw gateway restart`

---

## 回答问题前先回报
- Agent ID
- 当前状态
- 使用的 Chrome Profile（如果是浏览器操作）
