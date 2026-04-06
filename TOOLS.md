# TOOLS.md - 工具配置

---

## 🛠️ Awesome CLI Tools（自動遍歷）
**Skill:** `awesome-cli-tools` | 直接調用合適的 CLI 工具執行任務
**觸發：** CLI 工具推薦、終端應用、macOS 命令

| 領域 | 工具 |
|------|------|
| 文件管理 | nnn, ranger, ncdu, fzf |
| 系統監控 | htop, bottom, bpytop |
| 開發工具 | bat, exa, fd, ripgrep |
| 數據處理 | jq, yq, miller, fx |
| 網絡工具 | httpie, nmap, curl |

---

## 本 Agent 常用工具

### 文件操作
- read - 读取文件
- write - 写入文件
- edit - 编辑文件
- exec - 执行命令

### Pollinations 工具
- opencli - 可能用于 API 调用
- curl - HTTP 请求
- jq - JSON 处理

### 飞书工具
- feishu_bitable_app - 多维表格
- feishu_chat - 群组管理
- feishu_create_doc - 创建文档

### OpenClaw
- sessions_spawn - 创建 subagent
- image_generate - 图像生成

---

## 🚨 每次使用 browser tool 必須指定 `profile="pollinations-agent"`

---

## Gotchas

- 修改 openclaw.json 需要重启 gateway, 需要申请批准重启
- 目录路径使用绝对路径
- 群绑定需要正确的 chat_id
- Pollinations API 需要注意速率限制
- API 密钥需要妥善保管
