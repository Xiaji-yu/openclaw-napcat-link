---
tags:
  - openclaw
  - NapCat
  - websocket
date: 2026-02-10T20:29:00
---

# OpenClaw 部署及连接 NapCat 教程

> 本教程基于 Linux 环境，需提前安装 Node.js 22+ 版本。OpenClaw 部署完成后，接入 QQ 或其他聊天工具可以让你随时随地管理服务器。

## 目录

- [一、部署 OpenClaw](#一部署-openclaw)
- [二、配置 OpenClaw](#二配置-openclaw)
- [三、网页端设备配对（可选）](#三网页端设备配对可选)
- [四、安装 OpenClaw QQ 插件](#四安装-openclaw-qq-插件)
- [五、部署 NapCat](#五部署-napcat)
- [六、配置 OpenClaw 连接 NapCat](#六配置-openclaw-连接-napcat)

---

## 一、部署 OpenClaw

### 系统要求

- **Node.js >= 22**（建议定期检查官网更新）
- 操作系统：Linux/macOS/Windows（本教程以 Linux 为主）

### 部署方式

#### 1. 快速安装（推荐）

**Linux：**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows（PowerShell）：**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**后续步骤：**
```bash
openclaw onboard --install-daemon  # 运行新手引导
```

#### 2. 从源代码安装（适合开发/贡献者）

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build  # 首次会自动安装 UI 依赖
pnpm build
openclaw onboard --install-daemon
```

#### 3. 其他安装方式

- **Docker**：参考 [Docker 安装文档](https://docs.openclaw.ai/install/docker)
- **Nix**：参考 [Nix 安装文档](https://docs.openclaw.ai/install/nix)
- **Ansible**：参考 [Ansible 安装文档](https://docs.openclaw.ai/install/ansible)
- **Bun**（仅 CLI）：参考 [Bun 安装文档](https://docs.openclaw.ai/install/bun)

> **官方完整文档**：[https://docs.openclaw.ai/zh-CN/install](https://docs.openclaw.ai/zh-CN/install)

---

## 二、配置 OpenClaw

### 1. 运行新手引导

执行快速安装后，按照新手引导逐步配置：

```
I understand this is powerful and inherently risky. Continue?
│  ○ Yes / ● No
```

选择 **Yes** 进入配置流程（看不懂英文可借助翻译）。

**配置选项建议：**

| 选项 | 推荐值 | 说明 |
|------|--------|------|
| Onboarding mode | Manual | 手动配置端口、网络、Tailscale 等 |
| What do you want to set up? | Local gateway | 设置成本地网关 |
| Workspace directory | 默认路径 | 除非有特殊需求，否则保持默认 |
| Model/auth provider | MiniMax | 根据已有模型供应商选择并配置密钥 |
| Gateway port | 18789 | 默认端口，可自定义 |
| Gateway bind | LAN (0.0.0.0) | 局域网内所有设备可访问 |
| Gateway auth | Token | 推荐，后续访问 Web UI 需要 |
| Tailscale exposure | off | 如无内网穿透需求则关闭 |
| Gateway token | 自定义 | 务必记住，相当于网页访问密码 |
| Configure chat channels now? | yes | 现在配置频道 |
| Select a channel | Finished (Skip) | 稍后配置 |
| Configure skills now? | no | 技能稍后配置 |
| Enable hooks? | Skip | 跳过 |
| Install Gateway service | yes | 必须安装 |
| Gateway service runtime | node | Node 运行 |
| Gateway service already installed | Restart | 重启生效 |

### 2. 修改 `openclaw.json` 配置

关闭已运行的 OpenClaw 界面（Ctrl+C），回到主目录，找到 `.openclaw` 文件夹：

```bash
cd ~
ls -a  # 查看隐藏文件
```

进入 `.openclaw` 目录，编辑 `openclaw.json`：

```bash
cd .openclaw
vim openclaw.json
```

在 `gateway` 字段中，`"mode"` 和 `"auth"` 之间添加以下配置：

```json
"controlUi": {
  "allowInsecureAuth": true,
  "dangerouslyAllowHostHeaderOriginFallback": true
},
```

> **注意**：`dangerouslyAllowHostHeaderOriginFallback: true` 用于解决局域网访问时的安全检查问题。

**如需暴露到公网，还需添加：**
```json
"allowedOrigins": ["你的域名:端口"]
```

修改完成后保存并退出：
- 按 `Esc` 退出插入模式
- 输入 `:wq` 回车保存

### 3. 重启 OpenClaw

```bash
openclaw gateway restart
```

---

## 三、网页端设备配对（可选）

> 首次通过浏览器访问网关时需要进行设备配对，这是安全机制。

访问地址：`http://服务器IP:18789`

### 情况一：出现 "control ui requires device identity"

**方法 1：Chrome 浏览器（推荐）**

1. 地址栏输入：`chrome://flags/#unsafely-treat-insecure-origin-as-secure`
2. 找到 **"Insecure origins treated as secure"**
3. 在文本框中添加：`http://服务器IP:18789`（多个用逗号分隔）
4. 将下拉菜单改为 **Enabled**
5. 点击 **Relaunch** 重启浏览器

**方法 2：Edge 浏览器**

1. 地址栏输入：`edge://flags/#unsafely-treat-insecure-origin-as-secure`
2. 同样设置并重启

### 情况二：出现 "disconnected (1008): pairing required"

在服务器端执行：

```bash
openclaw devices list          # 查看连接请求列表
openclaw devices approve <requestId>  # 批准指定请求ID
```

批准后刷新浏览器即可。

---

## 四、安装 OpenClaw QQ 插件

### 1. 下载并安装插件

```bash
# 进入 OpenClaw 目录
cd ~/.openclaw

# 创建插件目录
mkdir -p extensions

# 进入插件目录
cd extensions

# 克隆 QQ 插件仓库
git clone https://github.com/Xiaji-yu/openclaw_QQ_plugin.git qq

# 进入插件目录
cd qq

# 安装依赖
npm install -g pnpm
pnpm install
```

启用插件：

```bash
openclaw plugins list    # 查看插件列表
openclaw plugins enable qq  # 启用 QQ 插件
```

### 2. 验证插件安装

插件启用后，会在 OpenClaw 日志中显示 QQ 插件已加载。无需其他配置即可开始使用。

---

## 五、部署 NapCat

NapCat 是 QQ 机器人平台，OpenClaw 通过它连接 QQ。

### 1. 创建部署目录

```bash
cd ~/opt
mkdir napcat
cd napcat
```

### 2. 创建 Docker Compose 配置

```bash
vim docker-compose.yaml
```

粘贴以下内容：

```yaml
version: '3.8'

services:
  napcat:
    image: mlikiowa/napcat-docker
    container_name: napcat
    restart: always
    network_mode: bridge
    ports:
      - "3001:3001"   # WebSocket API（OpenClaw 连接）
      - "6099:6099"   # WebUI 管理界面
    environment:
      - TZ=Asia/Shanghai
      - VNC_PASSWD=vncpasswd  # WebUI 密码，可自定义
```

**端口说明：**
- **3001**：WebSocket API，供 OpenClaw 连接
- **6099**：WebUI 管理界面

保存并退出（`:wq`）。

### 3. 启动容器

```bash
docker compose up -d
```

### 4. 登录 QQ 并获取 Token

查看容器日志，扫码登录 QQ 小号：

```bash
docker logs -f napcat
```

登录成功后，访问 `http://服务器IP:6099` 进入 NapCat WebUI，使用日志中的 Token 登录管理后台。

### 5. 配置 NapCat WebSocket 服务器

在 NapCat WebUI 中：

1. 点击「网络配置」→「新建」
2. 选择「新建 WebSocket 服务器」
3. 按图示配置：

| 配置项 | 值 |
|--------|-----|
| 服务器地址 | `127.0.0.1` 或服务器 IP |
| 端口 | `3001` |
| Token | 自定义（与 `openclaw.json` 中保持一致） |
| 协议版本 | OneBot V11 |
| 反向域 | 留空 |

保存配置。

---

## 六、配置 OpenClaw 连接 NapCat

编辑 OpenClaw 配置文件：

```bash
cd ~/.openclaw
vim openclaw.json
```

### 完整 QQ 频道配置

```json
{
  "channels": {
    "qq": {
      "wsUrl": "ws://127.0.0.1:3001",
      "accessToken": "your_token_here",
      "admins": [12345678],
      "allowedGroups": [10001, 10002],
      "blockedUsers": [],
      "systemPrompt": "你是一位精通 Linux 系统管理、Docker 容器化架构、以及 Python 脚本开发的专家级运维助理。",
      "historyLimit": 5,
      "keywordTriggers": ["小助手", "帮助"],
      "autoApproveRequests": false,
      "enableGuilds": true,
      "enableTTS": false,
      "rateLimitMs": 2500,
      "formatMarkdown": true,
      "antiRiskMode": false,
      "maxMessageLength": 4000,
      "requireMention": true
    }
  },
  "plugins": {
    "entries": {
      "qq": { "enabled": true }
    }
  }
}
```

### 配置项说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `wsUrl` | string | ✅ | NapCat WebSocket 地址 |
| `accessToken` | string | ✅ | 连接验证 Token（与 NapCat 配置一致） |
| `admins` | number[] | ❌ | 管理员 QQ 号列表，可执行 `/status` 等指令 |
| `requireMention` | boolean | ❌ | 是否需要 @ 触发，默认 `true` |
| `allowedGroups` | number[] | ❌ | 群组白名单，为空则响应所有群 |
| `blockedUsers` | number[] | ❌ | 用户黑名单 |
| `systemPrompt` | string | ❌ | AI 人设提示词 |
| `historyLimit` | number | ❌ | 历史消息条数（0 关闭），默认 5 |
| `keywordTriggers` | string[] | ❌ | 关键词触发（无需 @） |
| `autoApproveRequests` | boolean | ❌ | 自动通过好友/群邀请，默认 `false` |
| `enableGuilds` | boolean | ❌ | 开启 QQ 频道支持，默认 `true` |
| `enableTTS` | boolean | ❌ | 启用语音回复（需服务端支持），默认 `false` |
| `rateLimitMs` | number | ❌ | 发送间隔（防风控），默认 2500ms |
| `formatMarkdown` | boolean | ❌ | 将 Markdown 转换为易读文本，默认 `true` |
| `antiRiskMode` | boolean | ❌ | 开启风控规避（如 URL 加空格），默认 `false` |
| `maxMessageLength` | number | ❌ | 单条消息最大长度，默认 4000 |

### 保存并重启

```bash
openclaw gateway restart
```

---

## 常见问题

### Q1: `openclaw: command not found`

**原因**：npm 全局路径未加入 `PATH` 环境变量。

**解决：**

1. 查看 npm 全局路径：
   ```bash
   npm prefix -g
   ```

2. 查看当前 `PATH`：
   ```bash
   echo $PATH
   ```

3. 若缺少 npm 路径，编辑 `~/.bashrc`：
   ```bash
   vim ~/.bashrc
   ```

   在文件末尾添加：
   ```bash
   export PATH=$PATH:/你的/npm/目录/路径
   ```

4. 生效配置：
   ```bash
   source ~/.bashrc
   ```

5. 验证：
   ```bash
   which openclaw
   ```

### Q2: 网页端一直提示配对

确保：
1. 浏览器已添加安全例外（`chrome://flags` 中设置）
2. 已执行 `openclaw devices approve <id>`
3. 使用 `http://` 而非 `https://`（局域网下）

### Q3: QQ 机器人无响应

检查清单：
- [ ] NapCat 容器运行正常：`docker ps | grep napcat`
- [ ] WebSocket 端口 3001 可访问：`netstat -tlnp | grep 3001`
- [ ] `openclaw.json` 中 `wsUrl` 和 `accessToken` 正确
- [ ] QQ 插件已启用：`openclaw plugins list`
- [ ] OpenClaw 服务状态正常：`openclaw gateway status`
- [ ] 检查日志：`openclaw logs -f` 或 `docker logs -f napcat`

### Q4: 消息发送被风控

适当调整 `rateLimitMs`（建议 3000-5000），或开启 `antiRiskMode`。

---

## 参考资料

- OpenClaw 官方文档：[https://docs.openclaw.ai](https://docs.openclaw.ai)
- NapCat 官网：[https://napcat.napneko.icu](https://napcat.napneko.icu)
- NapCat Docker 项目：[https://github.com/NapNeko/NapCat-Docker](https://github.com/NapNeko/NapCat-Docker)
- OpenClaw QQ 插件：[https://github.com/Xiaji-yu/openclaw_QQ_plugin](https://github.com/Xiaji-yu/openclaw_QQ_plugin)

---

**祝你部署顺利！** 🎉

如有问题，欢迎在 OpenClaw Discord 社区交流：https://discord.com/invite/clawd
