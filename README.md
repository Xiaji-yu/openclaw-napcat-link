---
tags:
  - openclaw
  - NapCat
  - websocket
date: 2026-02-10T20:29:00
---
本教程基于 Linux 环境，需提前安装 Node.js 工具

**系统要求**
- **Node.js >= 22**（建议定期检查官网更新）

# 一、开始部署 OpenClaw

---

OpenClaw 的部署文档地址：[https://docs.openclaw.ai/zh-CN/install](openclaw)

### 1. 快速安装

Linux：
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Windows（PowerShell）：
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

下一步（如果跳过了新手引导）：
```bash
openclaw onboard --install-daemon  # 运行新手引导
```

### 2. 从源代码（贡献者/开发/GitHub）安装

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖
pnpm build
openclaw onboard --install-daemon # 运行新手引导
```

### 3. 其他安装方式

受于篇幅，其他安装方式请移步至官网文档

- Docker：[Docker](https://docs.openclaw.ai/install/docker)
- Nix：[Nix](https://docs.openclaw.ai/install/nix)
- Ansible：[Ansible](https://docs.openclaw.ai/install/ansible)
- Bun（仅 CLI）：[Bun](https://docs.openclaw.ai/install/bun)

# 二、部署过程选项

---

以下为 Linux 环境下快速安装的详细步骤过程：

## 1. 首先运行 `curl -fsSL https://openclaw.ai/install.sh | bash` 进行快速安装并开启新手引导](./images/快速

![快速安装安装.png)

这一步会等待比较久，具体时间取决于你的网速和处理器解压速度

![1-OpenClaw部署及连接 napcat 教程](./images/1-OpenClaw部署及连接%20napcat%20教程.png)

下载并安装完毕之后开启新手引导：
```
I understand this is powerful and inherently risky. Continue?
│  ○ Yes / ● No
```
选择 `Yes` 开始下一步（看不懂各个选项英文的，可以借助翻译软件）。

以下选项依次：
- **Onboarding mode** → Manual（手动配置端口、网络、Tailscale 和身份验证选项）
- **What do you want to set up?** → Local gateway（设置成本地网关）
- **Workspace directory** → your path（设置工作区目录，建议默认，除非你有把握设置正确的目录并给到足够权限）
- **Model/auth provider** → MiniMax（选择模型/认证提供商，这里我选择 minimax，大家可以根据自己已有的模型供应商进行选择并配置密钥）
- **Gateway port** → 18789（选择网关的监听端口，建议默认）
- **Gateway bind** → LAN（网关绑定，建议选择 lan 0.0.0.0 监听局域网内所有，除非你不想让其他人访问，选择 loopback 仅本地可访问）
- **Gateway auth** → Token（网关认证，推荐使用 token，后续用 Web UI 访问时要用到，一定要记住）
- **Tailscale exposure** → off（内置的内网穿透软件，这里我不使用，我有公网）
- **Gateway token (blank to generate)** → your Token（设置访问 token，相当于网页密码，一定要记住）
- **Configure chat channels now?** → yes（现在配置聊天频道）
- **Select a channel** → Finished (Skip for now)（先跳过）
- **Configure skills now?** → no（立即配置技能）
- **Enable hooks?** → Skip for now（跳过启用挂钩，空格选中再回车）
- **Install Gateway service** → yes（安装网关服务）
- **Gateway service runtime** → node
- **Gateway service already installed** → Restart（网关服务安装成功，选择重启）

## 2. 开始配置 openclaw.json 配置文件

安装完成后，还需要手动修改一些配置文件中的内容，才能把已有的功能全部释放出来。

### (1) 先给 OpenClaw 的界面 Ctrl+C 关掉，回到主目录，找到 .openclaw 文件夹

```bash
# 进入主目录输入
ls -a
# 会显示当前目录中所有目录（包括隐藏目录）
```

![2-OpenClaw部署及连接 napcat 教程](./images/2-OpenClaw部署及连接%20napcat%20教程.png)

### (2) 进入 .openclaw 文件夹内找到 openclaw.json，用 vim 工具修改其内容

```bash
vim openclaw.json
```

找到 gateway 字段，在其子字段 "mode" 和 "auth" 之间插入以下内容，按 **i** 键进入插入模式：

```json
"controlUi": { "allowInsecureAuth": true },
"trustedProxies": [ "127.0.0.1", "192.168.1.0/24" ],  # 第二个地址改成你的局域网网段地址
```

> **注意**：`trustedProxies` 配置用于防止外网访问时丢失真实 IP。

需要暴露在公网的还需要加一段：
```json
"allowedOrigins": ["你的域名+端口号"]
```

![4-OpenClaw部署及连接 napcat 教程](./images/4-OpenClaw部署及连接%20napcat%20教程.png)

gateway 字段内容如上，修改之后按 `Esc` 并输入 `:wq` 保存退出。**冒号一定要是英文冒号！**

## 3. 修改完毕就可以重启 OpenClaw 了

```bash
openclaw gateway restart
```

## PS：如果执行 openclaw gateway 出现 command not found 解决方案

---

首先排查 Node.js 和 npm 版本，再查看 npm 的文件路径，最后看看 npm 文件的路径有没有在环境变量里。

```bash
node -v   # 查看 Node.js 版本
npm -v    # 查看 npm 版本
npm prefix -g   # 查看 npm 的文件路径
echo "$PATH"    # 查看环境变量中有无 npm 的路径
```

![5-OpenClaw部署及连接 napcat 教程](./images/5-OpenClaw部署及连接%20napcat%20教程.png)

如果没有，请将 npm 的路径添加到环境变量里。

### 1. 打开并修改配置文件

```bash
vim ~/.bashrc
```

按 **i** 键进入插入模式，插入一行：
```bash
export PATH=$PATH:/你的/npm目录/路径
```

### 2. 保存并退出

- 按 `Esc` 退出插入模式
- 输入 `:wq` 并回车保存退出

### 3. 应用配置立即生效并验证

执行以下命令，让刚才的修改马上生效，而不需要重启终端：

```bash
source ~/.bashrc
```

输入以下命令查看路径是否已存在：
```bash
echo $PATH
```

如果路径已存在，那么再执行 `openclaw gateway` 就不会再出现 command not found 了。

> **重要**：这时候一定不要把终端关掉！因为还没设置 OpenClaw 后台运行，先不要关掉终端，后续操作请另打开一个终端进行。

---

# 三、网页端配置

---

### 1. 找另一台同局域网的电脑，打开浏览器，输入 Linux 主机地址 + 18789 打开网页

![6-OpenClaw部署及连接 napcat 教程](./images/6-OpenClaw部署及连接%20napcat%20教程.png)

此时，网页会报 `disconnected (1008): device identity required` 错误——别慌！

点击左侧的 **overview** 选项：

![7-OpenClaw部署及连接 napcat 教程](./images/7-OpenClaw部署及连接%20napcat%20教程.png)

![9-OpenClaw部署及连接 napcat 教程](./images/9-OpenClaw部署及连接%20napcat%20教程.png)

出现这个界面，在 Gateway Token 里面填上刚才在设置阶段设置的 token，然后点击 Connect：

![10-OpenClaw部署及连接 napcat 教程](./images/10-OpenClaw部署及连接%20napcat%20教程.png)

连接成功！

![11-OpenClaw部署及连接 napcat 教程](./images/11-OpenClaw部署及连接%20napcat%20教程.png)

显示已连接，还有当前运行状态 OK。

然后返回 Chat 界面，这时候就可以跟设置的 AI 模型对话了（前提是设置的 token 有效）：

![12-OpenClaw部署及连接 napcat 教程](./images/12-OpenClaw部署及连接%20napcat%20教程.png)

初次对话会初始化，它会阅读内置的 BOOTSTRAP.md 等文件来进行环境感知。

### 2. 给出第一条指令

这就像是一个刚出生的宝宝，它要扮演什么角色都要依靠你来告诉它。不过这个与其他的不同，它具有长期记忆，甚至可以访问或者操作局域网内其他设备（外网也可以）。它的人格文件等都在 `.openclaw/workspace` 目录下，你可以自己设置，也可以给它指令让它自己修改。

![13-OpenClaw部署及连接 napcat 教程](./images/13-OpenClaw部署及连接%20napcat%20教程.png)

---

# 四、安装 OpenClaw QQ 插件

---

来到这里，OpenClaw 部署才算基本成功。但总不能干个什么事情都要打开网页，如果它能接入到你的 QQ 小号，或者其他聊天工具，让你随时随地都能对它发号施令，对服务器进行一些操作（比如更新软件包、部署容器、设置定时任务等），那就更方便了。

### 1. 下载并安装插件

插件在我的 GitHub 仓库可以直接下载：

OpenClaw QQ Plugin：[https://github.com/Xiaji-yu/openclaw_QQ_plugin](openclaw_QQ_plugin)

```bash
# 打开 OpenClaw 目录
cd ~/.openclaw

# 新建一个目录用来存放插件
mkdir extensions

# 进入新建目录
cd extensions

# 克隆仓库
git clone https://github.com/Xiaji-yu/openclaw_QQ_plugin.git qq

# 进入仓库的 qq 子文件夹
cd qq

# 安装依赖并构建
npm install -g pnpm
pnpm install

# 查看 OpenClaw 插件列表
openclaw plugins list

# 启用 QQ 插件
openclaw plugins enable qq
```

![14-OpenClaw部署及连接 napcat 教程](./images/14-OpenClaw部署及连接%20napcat%20教程.png)

总体来说还算顺利，只有几个警告不用在意，安装完成。

### 2. 部署 NapCat

我们需要的是一个随身的助理，而不是每次想要做什么都掏出一台笔记本来对着网页一顿输出。这就需要一个 QQ 小号，但小号需要一个代理（平台）来登录——这就是 NapCat 干的活。

项目地址：
- NapCat：[https://napcat.napneko.icu/](napcat)
- NapCat Docker：[https://github.com/NapNeko/NapCat-Docker](napcat-docker)

最方便的方式是使用 Docker Compose 一键部署。

#### (1) 创建目录

```bash
# 打开 opt 目录
cd ~/opt

# 创建一个 napcat 文件夹
mkdir napcat

# 创建 docker compose 文件
vim docker-compose.yaml
```

#### (2) 创建 docker compose 文件

在打开的 docker compose 文件中输入以下内容：

```yaml
version: '3.8'

services:
  napcat:
    image: mlikiowa/napcat-docker
    container_name: napcat
    restart: always
    network_mode: bridge
    ports:
      - "3001:3001"   # WebSocket API
      - "6099:6099"   # WebUI
    environment:
      - TZ=Asia/Shanghai
      - VNC_PASSWD=vncpasswd
```

> **端口说明**：
> - **3001**：WebSocket API（OpenClaw 连接用）
> - **6099**：WebUI 管理界面

#### (3) 保存并退出

- 按 `Esc` 退出插入模式
- 输入 `:wq` 并回车保存退出

#### (4) 一键部署容器

```bash
docker compose up -d
```

#### (5) 登录 QQ 并获取 Token

容器部署完毕之后，打开 NapCat 的日志来进行登录和获取 Token 的操作：

```bash
docker logs -f napcat
```

![15-OpenClaw部署及连接 napcat 教程](./images/15-OpenClaw部署及连接%20napcat%20教程.png)

扫码登录你的小号，然后浏览器访问 Linux 主机地址 + 6099 访问 WebUI 界面，登陆密码就是图中的 Token，然后进入管理界面：

![16-OpenClaw部署及连接 napcat 教程](./images/16-OpenClaw部署及连接%20napcat%20教程.png)

#### (6) 配置 NapCat 的网络信息

点击「网络配置」→「新建」→「新建 WebSocket 服务器」：

![17-OpenClaw部署及连接 napcat 教程](./images/17-OpenClaw部署及连接%20napcat%20教程.png)

如图配置，尽量让 NapCat 部署在同一台物理机上，没有也没关系。注意记住上面的 Token 和端口号，下面配置还要用到。

### 3. 手动配置详解 (`openclaw.json`)

---

依旧打开 OpenClaw 的配置文件：

```bash
vim ~/.openclaw/openclaw.json
```

以下是完整配置清单：

```json
{
  "channels": {
    "qq": {
      "wsUrl": "ws://127.0.0.1:3001",
      "accessToken": "123456",
      "admins": [12345678],
      "allowedGroups": [10001, 10002],
      "blockedUsers": [999999],
      "systemPrompt": "你是一位精通 Linux 系统管理、Docker 容器化架构、以及 Python 脚本开发 的专家级运维助理。你擅长处理复杂的网络调试（如 ECONNREFUSED）、权限管理以及系统服务优化",
      "historyLimit": 5,
      "keywordTriggers": ["小助手", "帮助"],
      "autoApproveRequests": true,
      "enableGuilds": true,
      "enableTTS": false,
      "rateLimitMs": 2500,
      "formatMarkdown": true,
      "antiRiskMode": false,
      "maxMessageLength": 4000
    }
  },
  "plugins": {
    "entries": {
      "qq": { "enabled": true }
    }
  }
}
```

| 配置项                   | 类型       | 默认值     | 说明                                             |
| :-------------------- | :------- | :------ | :--------------------------------------------- |
| `wsUrl`               | string   | **必填**  | OneBot v11 WebSocket 地址                        |
| `accessToken`         | string   | -       | 连接鉴权 Token                                     |
| `admins`              | number[] | `[]`    | **管理员 QQ 号列表**。拥有执行 `/status`, `/kick` 等指令的权限。 |
| `requireMention`      | boolean  | `true`  | **是否需要 @ 触发**。设为 `true` 仅在被 @ 或回复机器人时响应。       |
| `allowedGroups`       | number[] | `[]`    | **群组白名单**。若设置，Bot 仅在这些群组响应；若为空，则响应所有群组。        |
| `blockedUsers`        | number[] | `[]`    | **用户黑名单**。Bot 将忽略这些用户的消息。                      |
| `systemPrompt`        | string   | -       | **人设设定**。注入到 AI 上下文的系统提示词。                     |
| `historyLimit`        | number   | `5`     | **历史消息条数**。群聊时携带最近 N 条消息给 AI，设为 0 关闭。          |
| `keywordTriggers`     | string[] | `[]`    | **关键词触发**。群聊中无需 @，包含这些词也会触发回复。                 |
| `autoApproveRequests` | boolean  | `false` | 是否自动通过好友申请和群邀请。                                |
| `enableGuilds`        | boolean  | `true`  | 是否开启 QQ 频道 (Guild) 支持。                         |
| `enableTTS`           | boolean  | `false` | (实验性) 是否将 AI 回复转为语音发送 (需服务端支持 TTS)。            |
| `rateLimitMs`         | number   | `2500`  | **发送限速**。多条消息间的延迟(毫秒)，建议设为 2000~3000 以防风控。     |
| `formatMarkdown`      | boolean  | `false` | 是否将 Markdown 表格/列表转换为易读的纯文本排版。                 |
| `antiRiskMode`        | boolean  | `false` | 是否开启风控规避（如给 URL 加空格）。                          |
| `maxMessageLength`    | number   | `4000`  | 单条消息最大长度，超过将自动分片发送。                            |

---

# 五、连接方式详解

---

## 5.1 WebSocket 连接（推荐）

这是默认且推荐的连接方式，适合实时消息收发。

```json
{
  "channels": {
    "qq": {
      "wsUrl": "ws://<NapCat服务器IP>:3001",
      "accessToken": "你的Token"
    }
  }
}
```

**适用场景**：群聊消息、私聊消息、QQ 频道消息

## 5.2 HTTP API 连接（可选）

如果只需要接收消息，不需要主动发送，可以只用 HTTP API。

```json
{
  "channels": {
    "qq": {
      "httpUrl": "http://<NapCat服务器IP>:3000",
      "accessToken": "你的Token"
    }
  }
}
```

> **注意**：HTTP 方式需要 NapCat 开启 HTTP API 功能（部分 NapCat 版本支持）。

## 5.3 同时启用双通道

```json
{
  "channels": {
    "qq": {
      "wsUrl": "ws://<NapCat服务器IP>:3001",
      "httpUrl": "http://<NapCat服务器IP>:3000",
      "accessToken": "你的Token"
    }
  }
}
```

---

# 六、NapCat 端口说明

---

| 端口  | 协议   | 用途                    |
| :--- | :----- | :--------------------- |
| 3000 | HTTP   | HTTP API               |
| 3001 | WebSocket | WebSocket API（OpenClaw 连接用） |
| 6099 | HTTP   | WebUI 管理界面          |

> 如果部署在不同机器上，确保防火墙开放相应端口：
> ```bash
> firewall-cmd --add-port=3000/tcp --permanent
> firewall-cmd --add-port=3001/tcp --permanent
> firewall-cmd --reload
> ```

---

# 七、常见问题排查

---

### Q1: 连接显示 `ECONNREFUSED`

1. 检查 NapCat 容器是否运行：`docker ps`
2. 检查端口是否正确：`netstat -tlnp | grep 3001`
3. 检查防火墙：`firewall-cmd --list-ports`
4. 如果 NapCat 在另一台机器，确认内网互通

### Q2: 消息收不到

1. 确认 NapCat 已成功登录 QQ（查看 `docker logs -f napcat`）
2. 检查 `admins` 配置是否包含你的 QQ 号
3. 检查 `allowedGroups` 是否为空（为空则响应所有群）
4. 确认 OpenClaw 已启用 QQ 插件：`openclaw plugins list`

### Q3: WebSocket 断连频繁

1. 检查网络稳定性
2. 适当增加 `rateLimitMs` 到 3000-5000
3. 查看 NapCat 日志是否有错误

### Q4: 机器人不回复

1. 检查是否需要 @ 触发（`requireMention` 默认 true）
2. 检查 `blockedUsers` 是否误填了自己的 QQ
3. 查看 OpenClaw 日志：`openclaw gateway logs`

### Q5: 如何通过外网连接

如果 NapCat 需要通过公网访问：
1. 使用 Tailscale / frp / Cloudflare Tunnel 等内网穿透
2. 在 OpenClaw 配置中使用公网地址（需 HTTPS 或配置 `insecure`）
3. 建议使用 WebSocket over TLS

---
