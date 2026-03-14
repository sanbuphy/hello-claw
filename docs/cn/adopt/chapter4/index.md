---
prev:
  text: '第3章 初始配置向导'
  link: '/cn/adopt/chapter3'
next:
  text: '第5章 模型管理'
  link: '/cn/adopt/chapter5'
---

# 第四章 聊天平台接入

> 本章介绍如何将 OpenClaw 接入聊天平台，让你的龙虾拥有"联系方式"。完成后你可以在飞书中随时和它对话。

> **AutoClaw 用户提示**：[AutoClaw](/cn/adopt/chapter1/) 已内置飞书一键接入（扫码即完成），可跳过本章。

## 0. 支持的聊天平台

OpenClaw 可以连接你日常使用的几乎所有聊天软件。每个渠道通过 Gateway 接入，文字消息全渠道支持，媒体和互动能力因平台而异。

| 渠道 | 说明 | 安装方式 |
|------|------|---------|
| **飞书 / Lark** | WebSocket 长连接；企业协作首选 | 内置 |
| **WhatsApp** | 全球最流行；Baileys 库，需 QR 配对 | 内置 |
| **Telegram** | Bot API（grammY）；支持群聊，API 最开放 | 内置 |
| **Discord** | Bot API + Gateway；服务器、频道、私信 | 内置 |
| **Slack** | Bolt SDK；工作区应用 | 内置 |
| **Signal** | signal-cli；注重隐私 | 内置 |
| **Google Chat** | Google Chat API；HTTP webhook | 内置 |
| **iMessage** | BlueBubbles（推荐）或旧版 imsg CLI | 内置 |
| **IRC** | 经典 IRC 服务器；频道 + 私信 | 内置 |
| **WebChat** | Gateway 内置 Web 聊天界面 | 内置 |
| **QQ** | QQ 开放平台 Bot API | 插件 |
| **LINE** | LINE Messaging API | 插件 |
| **Matrix** | Matrix 开放协议 | 插件 |
| **Mattermost** | Bot API + WebSocket | 插件 |
| **Microsoft Teams** | Bot Framework；企业支持 | 插件 |
| **Nostr** | 去中心化协议 NIP-04 | 插件 |
| **Twitch** | IRC 连接 | 插件 |
| **Zalo** | Zalo Bot API（越南） | 插件 |

> **多渠道同时接入**：你可以同时连接多个平台，OpenClaw 会自动按来源路由消息。比如在飞书处理工作、在 Telegram 做个人助理——它们共享同一个 AI 大脑。

> **最快上手**：Telegram 配置最简单（只需一个 Bot Token），但在国内需要代理。本章以**飞书**为例——它与国内办公场景高度契合，深度集成文档、日历、多维表格等能力，能真正成为你的"数字分身"。

## 1. 前置准备

在开始之前，确保：

- 已完成 [第二章](/cn/adopt/chapter2/) 的安装，OpenClaw 正在运行（`openclaw status` 显示正常）
- 拥有飞书账号

<details>
<summary>个人用户也能用</summary>

即使你不是企业管理员，飞书也允许创建"企业自建应用"用于个人测试和使用，不需要企业认证。

</details>

## 2. 创建飞书应用

### 第一步：登录飞书开放平台

访问 [飞书开放平台](https://open.feishu.cn/app)，使用飞书账号登录。

![飞书开放平台首页](/lark.png)

<details>
<summary>国际版 Lark 用户</summary>

请访问 [Lark Open Platform](https://open.larksuite.com/app)，后续配置中需设置 `domain: "lark"`。

</details>

### 第二步：创建企业自建应用

1. 点击"创建企业自建应用"
2. 填写应用名称（如"OpenClaw 助理"）、描述、选择图标
3. 点击"创建"进入应用详情页

### 第三步：获取应用凭证

进入"凭证与基础信息"页面，复制 **App ID**（格式 `cli_xxx`）和 **App Secret**。

![飞书应用凭证页面](/lark-credential.png)

> **重要**：App Secret 务必立即复制保存，不要泄露给他人。

### 第四步：启用机器人能力

进入"添加应用能力" → "机器人"页面：

![添加应用能力页面](/lark-app.png)

点击"机器人"卡片下方的"添加"按钮，开启机器人能力。添加成功后页面会显示机器人配置面板，点击"如何开始使用"可赋予或修改机器人显示名称，填写后点击"确定"即可。

> **注意**：必须先启用机器人能力，否则下一步批量导入权限时，消息相关权限（如发送系统消息、接收群聊@机器人消息、读取单聊消息）将无法开通。

### 第五步：配置权限

进入"权限管理"页面，点击"批量导入"按钮，粘贴以下 JSON 一键导入所需权限：

![飞书权限管理页面](/lark-authentication.png)

```json
{
  "scopes": {
    "tenant": [
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:contact.base:readonly",
      "contact:user.employee_id:readonly",
      "docs:document.content:read",
      "docx:document:readonly",
      "event:ip_list",
      "im:chat",
      "im:chat.members:bot_access",
      "im:chat:read",
      "im:chat:update",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.group_msg",
      "im:message.p2p_msg:readonly",
      "im:message.pins:read",
      "im:message.pins:write_only",
      "im:message.reactions:read",
      "im:message.reactions:write_only",
      "im:message:readonly",
      "im:message:recall",
      "im:message:send_as_bot",
      "im:message:send_multi_users",
      "im:message:send_sys_msg",
      "im:message:update",
      "im:resource",
      "sheets:spreadsheet",
      "wiki:wiki:readonly"
    ],
    "user": [
      "base:app:copy",
      "base:app:create",
      "base:app:read",
      "base:app:update",
      "base:field:create",
      "base:field:delete",
      "base:field:read",
      "base:field:update",
      "base:record:create",
      "base:record:delete",
      "base:record:retrieve",
      "base:record:update",
      "base:table:create",
      "base:table:delete",
      "base:table:read",
      "base:table:update",
      "base:view:read",
      "base:view:write_only",
      "board:whiteboard:node:create",
      "board:whiteboard:node:read",
      "calendar:calendar.event:create",
      "calendar:calendar.event:delete",
      "calendar:calendar.event:read",
      "calendar:calendar.event:reply",
      "calendar:calendar.event:update",
      "calendar:calendar.free_busy:read",
      "calendar:calendar:read",
      "contact:contact.base:readonly",
      "contact:user.base:readonly",
      "contact:user.employee_id:readonly",
      "contact:user:search",
      "docs:document.comment:create",
      "docs:document.comment:read",
      "docs:document.comment:update",
      "docs:document.media:download",
      "docs:document:copy",
      "docx:document:create",
      "docx:document:readonly",
      "docx:document:write_only",
      "drive:drive.metadata:readonly",
      "drive:file:download",
      "drive:file:upload",
      "im:chat.members:read",
      "im:chat:read",
      "im:message",
      "im:message.group_msg:get_as_user",
      "im:message.p2p_msg:get_as_user",
      "im:message:readonly",
      "offline_access",
      "search:docs:read",
      "search:message",
      "space:document:delete",
      "space:document:move",
      "space:document:retrieve",
      "task:comment:read",
      "task:comment:write",
      "task:task:read",
      "task:task:write",
      "task:task:writeonly",
      "task:tasklist:read",
      "task:tasklist:write",
      "wiki:node:copy",
      "wiki:node:create",
      "wiki:node:move",
      "wiki:node:read",
      "wiki:node:retrieve",
      "wiki:space:read",
      "wiki:space:retrieve",
      "wiki:space:write_only"
    ]
  }
}
```

> 批量导入会自动开通消息收发、云文档、多维表格、日历、任务等完整能力。

<details>
<summary>这些权限分别做什么？</summary>

| 权限类别 | 代表权限 | 用途 |
|---------|---------|------|
| **应用管理（application:）** | `application:application:self_manage`、`application:bot.menu:write` | 应用自管理、机器人菜单配置 |
| **消息卡片（cardkit:）** | `cardkit:card:read`、`cardkit:card:write` | 读写消息卡片 |
| **消息（im:）** | `im:message`、`im:message:send_as_bot`、`im:resource` | 收发消息、图片、文件 |
| **群聊（im:chat）** | `im:chat`、`im:chat.members:bot_access` | 群聊管理、成员访问 |
| **联系人（contact:）** | `contact:user.base:readonly`、`contact:user.employee_id:readonly` | 获取用户基础信息 |
| **云文档（docx:/docs:）** | `docx:document:create`、`docs:document.content:read` | 创建和读取飞书文档 |
| **电子表格（sheets:）** | `sheets:spreadsheet` | 操作飞书电子表格 |
| **多维表格（base:）** | `base:record:create`、`base:table:read` | 操作多维表格数据 |
| **日历（calendar:）** | `calendar:calendar.event:create`、`calendar:calendar.event:read` | 管理日程 |
| **任务（task:）** | `task:task:read`、`task:task:write` | 创建和管理飞书任务 |
| **知识库（wiki:）** | `wiki:node:read`、`wiki:wiki:readonly` | 读写飞书知识库 |
| **云空间（drive:/space:）** | `drive:file:upload`、`drive:file:download` | 上传下载文件 |
| **白板（board:）** | `board:whiteboard:node:create`、`board:whiteboard:node:read` | 读写飞书白板 |
| **搜索（search:）** | `search:docs:read`、`search:message` | 搜索文档和消息 |
| **事件（event:）** | `event:ip_list` | 事件推送 IP 白名单 |

如果你只需要基础聊天功能，最少只需 `im:message`、`im:message.p2p_msg:readonly`、`im:message.group_at_msg:readonly`、`im:message:send_as_bot`、`im:resource` 这几个消息相关权限即可。但建议导入完整权限以获得最佳体验。

</details>

导入权限后，点击"申请开通"，在弹出的数据范围确认框中点击"确认"。如果你是企业管理员可直接通过；否则需联系管理员审核。

### 第六步：配置事件订阅

> **重要**：在配置事件订阅之前，请先完成下面的[第 3 步](#_3-在-openclaw-中添加飞书渠道)（添加飞书渠道并确保网关在运行）。否则长连接设置可能保存失败。

进入"事件与回调" → "事件配置"：

![事件与回调页面](/lark-event.png)

1. 选择"**使用长连接接收事件**"（WebSocket 模式）
2. 添加事件：`im.message.receive_v1`（接收消息事件）

<details>
<summary>什么是"长连接"？为什么推荐？</summary>

传统的 webhook 方式需要你有一个公网可访问的地址，飞书把消息推送过来。而长连接（WebSocket）是反过来的——OpenClaw 主动连接飞书服务器并保持连接，消息实时送达。

好处：不需要公网 IP、不需要域名、不需要端口映射，家用网络就能用。这也是 OpenClaw 的默认推荐方式。

</details>

### 第七步：发布应用

![版本管理与发布页面](/lark-version.png)

1. 进入"版本管理与发布"
2. 点击"创建版本"，填写版本号和更新说明
3. 点击"申请线上发布"，在确认弹窗中提交发布申请
4. 等待企业管理员审批（如果你本人就是管理员，可直接通过）

## 3. 在 OpenClaw 中添加飞书渠道

飞书应用创建完成后，回到终端，将飞书渠道添加到 OpenClaw。

**方式一：命令行向导（推荐）**

```bash
openclaw channels add
```

按交互式提示操作：
1. 选择 "Feishu/Lark (飞书)"
2. 输入 App ID 和 App Secret
3. 其他选项保持默认即可

**方式二：通过配置向导添加**

如果你刚安装完 OpenClaw、还没完成过初始配置，可以运行：

```bash
openclaw onboard
```

向导会引导你完成模型配置和渠道添加。

添加完成后，重启网关使配置生效：

```bash
openclaw gateway restart
```

验证网关状态：

```bash
openclaw gateway status
```

<details>
<summary>手动编辑配置文件（高级）</summary>

编辑 `~/.openclaw/openclaw.json`（Windows：`C:\Users\你的用户名\.openclaw\openclaw.json`）：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "connectionMode": "websocket",
      "dmPolicy": "pairing",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "你的App Secret"
        }
      }
    }
  }
}
```

修改后运行 `openclaw gateway restart` 生效。

</details>

> **注意顺序**：先完成此步骤（添加渠道 + 启动网关），再回到飞书开放平台配置事件订阅（[第六步](#第六步-配置事件订阅)）。网关未运行时，长连接设置会保存失败。

## 4. 配对与首次对话

### 第一步：发送测试消息

在飞书中找到你创建的机器人，发送一条消息（如"你好"）。

### 第二步：获取配对码

机器人会回复一个**配对码**（8 位大写字母数字），表示有新用户请求对话。

![飞书配对码提示界面](/lark-pairing.jpg)

<details>
<summary>什么是"配对"（Pairing）？</summary>

配对是 OpenClaw 的安全机制——不是任何人给你的机器人发消息都能得到回复。默认情况下，新用户发消息时会收到一个配对码，只有你在终端批准后，该用户才能正常对话。

这防止了陌生人滥用你的 AI 助手（毕竟每次对话都消耗你的模型 API 额度）。

配对码特点：
- 8 位字符，大写字母和数字，不含易混淆字符（如 0/O、1/I）
- 1 小时后过期
- 每个渠道最多 3 个待批准请求

</details>

### 第三步：批准配对

在终端执行：

```bash
openclaw pairing approve feishu <配对码>
```

> 把 `<配对码>` 替换成你实际收到的码，例如 `openclaw pairing approve feishu 6KKG7C7K`。

你也可以在 OpenClaw Web 控制面板（`openclaw dashboard`）中点击批准按钮。

### 第四步：开始对话

配对成功后，回到飞书再给机器人发一条消息，它就能正常回复了！试试：

```
你好，请介绍一下你自己
```

恭喜！你的飞书 AI 助手已经上线。日常使用直接在飞书中和机器人对话即可，无需额外操作。

## 5. 群聊中使用

除了私聊，你还可以把机器人拉进飞书群聊，让团队成员共同使用。

**基本规则**：在群聊中需要 **@机器人** 才会触发回复，避免在群里刷屏。

<details>
<summary>群聊访问控制</summary>

OpenClaw 通过 `groupPolicy` 控制群聊行为：

| 策略 | 行为 |
|------|------|
| `"open"` | 允许所有群聊，仍需 @提及才回复 |
| `"allowlist"` | 仅允许白名单中的群（默认） |
| `"disabled"` | 禁用所有群聊消息 |

配置示例：

```bash
# 允许所有群聊
openclaw config set channels.feishu.groupPolicy "open"

# 设置某个群不需要@就回复
openclaw config set channels.feishu.groups.<群ID>.requireMention false
```

群聊中每个群拥有独立的会话——群里的对话不会影响你和机器人的私聊记录。

</details>

<details>
<summary>私聊访问策略（dmPolicy）</summary>

`dmPolicy` 控制谁能通过私聊使用你的机器人：

| 策略 | 行为 |
|------|------|
| `"pairing"` | 默认。新用户需配对码批准 |
| `"allowlist"` | 仅允许 `allowFrom` 列表中的用户 |
| `"open"` | 允许所有人（需在 `allowFrom` 中设置 `"*"`） |
| `"disabled"` | 禁用私聊 |

</details>

## 6. 常见问题

**Q: 事件订阅保存失败？**

A: 请确保已先运行 `openclaw channels add` 添加飞书渠道，且网关处于运行状态（`openclaw gateway status` 显示正常）。长连接模式要求网关在线才能注册。

**Q: 机器人没有回复？**

A: 逐步排查：
1. 网关是否运行：`openclaw status`
2. 飞书渠道是否已添加：检查 `openclaw.json` 中的 `channels.feishu` 配置
3. 是否已完成配对：`openclaw pairing list feishu`
4. 查看实时日志定位错误：`openclaw logs --follow`
5. 重启网关重试：`openclaw gateway restart`

**Q: 权限审核不通过？**

A: 如果你本人就是管理员，进入飞书管理后台（Lark Admin） → "工作台" → "应用审核"，找到对应应用点击"审核"，然后点击"通过"即可。否则联系企业管理员完成审批。

![应用审核页面](/lark-review.png)

**Q: 配置修改后不生效？**

A: 修改配置后必须重启网关：`openclaw gateway restart`。更多配置项说明见[附录 G：配置文件详解](/cn/appendix/appendix-g)。

**Q: 群聊中 @了机器人但没反应？**

A: 检查 `groupPolicy` 是否为 `"disabled"`；如果是 `"allowlist"`，确认该群的 ID 已加入白名单。
