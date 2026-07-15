# SakuraBot 🌸 Release

> SakuraBot 的Release 发布与全平台构建仓库。
> 这里负责定时检查源码仓库的 tag，使用 GitHub Actions 构建 Release 产物 ✨

SakuraBot 是一个面向 QQ 官方机器人的管理工具，由 **Rust 后端服务** 和 **Tauri 桌面管理客户端** 组成。
后端负责连接 QQBot WebSocket / Webhook、保存消息历史、代理官方 API、转发实时事件；客户端负责管理 Bot、查看会话、发送消息、维护配置和观察运行状态。

由于源码仓库不承担全平台构建任务，Release 产物统一由本仓库生成。
因此本仓库作为发布构建仓库使用：源码仓库推送 tag 后，本仓库通过 GitHub Actions cron 定时检查源码仓库最新 tag，并在公开环境中完成构建和发布。

## ✨ 这里有什么

- 🚀 全平台 Release 构建工作流
- 🛠️ 后端可执行文件构建
- 🖥️ 桌面客户端构建
- 📦 Release 产物上传
- 🌐 面向用户的公开下载入口

## 📦 下载

请前往本仓库的 **Releases** 页面下载最新版本：

```text
https://github.com/Lovely-02/SakuraBot/releases
```

Release 产物会按平台区分，常见命名类似：

```text
SakuraBot-backend-x86_64-pc-windows-msvc.tar.gz
SakuraBot-backend-x86_64-apple-darwin.tar.gz
SakuraBot-backend-x86_64-unknown-linux-gnu.tar.gz
SakuraBot-frontend-desktop-...
SakuraBot-frontend-ios-unsigned.ipa
```

不同版本的实际文件名以 Releases 页面为准。

## 🧁 项目能力

- 🤖 多 Bot 管理：新增、编辑、删除、启停 Bot，支持正式环境和沙箱环境
- 💬 实时聊天：群聊、好友、频道会话列表，支持会话置顶、备注、头像和消息历史
- 📨 消息发送：文本、Markdown、引用回复、图片、视频、文件、语音等富媒体能力
- 🌐 官方事件接入：支持 QQBot WebSocket 和 Webhook 两种模式
- 🧩 API 代理：统一代理群、单聊、频道、子频道、频道私信、成员、权限、表态、日程、帖子和音频相关能力
- 🗄️ 多数据库：支持 SQLite / MySQL / PostgreSQL
- 🔐 管理鉴权：HTTP 管理接口和管理事件 WebSocket 使用 `Authorization: Bearer <token>`
- 🖥️ 桌面客户端：基于 Tauri，面向 Windows / macOS / Linux 等平台

## 🚀 快速使用

### 1. 下载后端

从 Releases 下载与你系统匹配的后端压缩包，解压后运行：

```bash
./SakuraBot
```

Windows PowerShell：

```powershell
.\SakuraBot.exe
```

首次启动会在运行目录生成：

```text
config.toml
SakuraBot.db
```

默认监听地址：

```text
http://127.0.0.1:3000
```

### 2. 下载桌面客户端

从 Releases 下载与你系统匹配的桌面客户端安装包或可执行文件。

登录时填写：

```text
后端地址: ws://127.0.0.1:3000
管理 Token: config.toml 中的 webui.token
```

也可以填写 `http://127.0.0.1:3000`，客户端会自动转换 HTTP 和 WebSocket 地址 🌷

## ⚙️ 配置文件

主要配置文件是 `config.toml`。文件不存在时，后端会自动生成默认配置。

```toml
[server]
host = "0.0.0.0"
port = 3000
max_connections = 1000

[websocket]
heartbeat_interval = 30
connection_timeout = 90

[webui]
enabled = true
title = "SakuraBot"
token = "admin123"
session_duration_hours = 24

[push]
mode = "broadcast"

[database]
type = "sqlite"
url = "sqlite://SakuraBot.db?mode=rwc"
pool_size = 10

[message_history]
retention_days = 7

[framework]
log_level = "info"
auto_load_bots = true
graceful_shutdown_timeout = 30
```

上线前请把 `webui.token` 改成随机高强度字符串 🔐

## 🌸 接入方式

### 桌面管理客户端

桌面客户端连接后端管理 API，并使用请求头鉴权：

```http
Authorization: Bearer <token>
```

客户端登录时只需要填写后端根地址，例如：

```text
ws://127.0.0.1:3000
```

不要填写 `/websocket/{app_id}/{app_secret}`，那个是业务程序使用的入口。

### 业务 WebSocket

外部业务程序如需接收 QQBot 原始事件或调用机器人 API，使用：

```text
ws://127.0.0.1:3000/websocket/{app_id}/{app_secret}
```

基础请求格式：

```json
{
	"id": "request-1",
	"api": "send_message",
	"target_type": "group",
	"target_id": "GROUP_OPENID",
	"params": {
		"msg_type": 0,
		"content": "你好呀 🌸"
	}
}
```

### Webhook

Webhook 模式订阅地址：

```text
http://127.0.0.1:3000/webhook/{app_id}/{app_secret}
```

在 QQBot 官方后台填写公网可访问的 Webhook 地址即可。后端会校验签名、保存消息并广播事件。

## 🛠️ 构建流程

本仓库的 `.github/workflows/release.yml` 负责发版构建：

1. 源码仓库推送 Release tag。
2. 本仓库通过 cron 定时检查源码仓库最新 tag。
3. 如果发布仓库对应 Release 没有 `SakuraBot-build-complete.txt` 完成标记，则开始构建。
4. GitHub Actions 使用 `GH_TOKEN` 检出源码仓库对应 tag 的源码。
5. 分别构建后端、桌面端和无签名 iOS IPA 产物。
6. 将构建产物和完成标记上传到本仓库 Release。

也可以在 Actions 页面手动运行 `🚀 发版构建`，输入需要构建的 tag；留空则检查源码仓库最新 tag。

需要在发布仓库 Secrets 中配置 `GH_TOKEN`，该 token 需要拥有读取源码仓库 `Lovely-02/SakuraBot` 和写入本仓库 Release 的权限。

## ❓ 常见问题

### 这是主开发仓库吗？

不是。这里是发布和构建仓库，主要用于承载 Release、构建产物和发布说明。

### 为什么要用发布仓库构建？

源码仓库不承担全平台构建任务。本仓库用于统一生成并发布构建产物。

### 客户端登录地址填什么？

填写后端根地址，例如：

```text
ws://127.0.0.1:3000
```

或：

```text
http://127.0.0.1:3000
```

### 管理 Token 在哪里？

管理 Token 是 `config.toml` 中的 `webui.token`。

## 🌙 小尾巴

SakuraBot 希望做一只安静可靠的机器人小管家：
平时乖乖待在后台，消息来了就认真记录，事件到了就及时转发，需要发消息时也会好好把话送出去。

祝你的 Bot 每天在线，日志干净，消息都能顺利送达 🌸
