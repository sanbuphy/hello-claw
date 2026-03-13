---
prev:
  text: '第6章 智能体管理'
  link: '/cn/adopt/chapter6'
next:
  text: '第8章 网关运维'
  link: '/cn/adopt/chapter8'
---

# 第七章 工具与定时任务

> 本章介绍 OpenClaw 的工具系统（Tools）——让龙虾从"只会聊天"变成"能干实事"的关键。完成后你将了解如何管理工具权限、使用联网搜索、浏览器自动化和定时任务等核心能力。

> **AutoClaw 用户提示**：[AutoClaw](/cn/adopt/chapter1/) 默认开启 `full` 工具配置档，无需手动配置。本章帮助你理解工具系统的工作方式，以及如何按需调整。

## 0. 什么是 Tools？

OpenClaw 的 AI 能力来自模型提供商（第二章配置的"大脑"），而**工具（Tools）就是龙虾的"双手"**——没有工具，它只能跟你聊天；有了工具，它能搜索网页、操作浏览器、读写文件、执行命令、定时跑任务。

先厘清两个容易混淆的概念：

|  | Tools（工具） | Skills（技能） |
|---|---|---|
| **是什么** | OpenClaw 内置的底层能力 | 社区或用户编写的高层指令 |
| **举例** | 执行命令、读写文件、搜索网页 | "每天早上发晨间简报"、"总结网页" |
| **谁提供** | OpenClaw 核心代码 | ClawHub 技能市场 / 用户自己编写 |
| **如何管理** | `openclaw.json` 中的 `tools` 配置 | `clawhub` CLI 安装/卸载 |

简单来说：**Tools 是手脚，Skills 是招式**。技能通过调用工具来完成任务。

## 1. 工具配置档（Tool Profiles）

Tool Profile 控制龙虾"能做什么"。OpenClaw 提供 4 个档位，从最开放到最受限：

| 配置档 | 能力范围 | 适用场景 |
|--------|---------|---------|
| `full` | 无限制，所有工具可用 | **推荐**——个人电脑上的全能助手 |
| `coding` | 文件读写、命令执行、会话管理、记忆、图片分析 | 开发者专用，不含消息和浏览器 |
| `messaging` | 消息收发、会话浏览、状态查看 | 纯聊天机器人，不能操作文件或执行命令 |
| `minimal` | 仅状态查看 | 最小权限，几乎什么都不能做 |

### 查看和修改当前配置档

```bash
# 查看当前配置
openclaw config get tools.profile

# 设置为 full（推荐）
openclaw config set tools.profile full
openclaw gateway restart
```

> **第二章遗留问题**：OpenClaw 3.7 之前的版本默认配置档可能是 `messaging`，导致龙虾"只会说不会做"。如果你发现龙虾只给建议却不执行操作，运行上面的命令切换到 `full`。

::: danger full 模式安全提示
`full` 模式意味着 OpenClaw 可以在你的电脑上执行任意命令、读写任意文件。个人电脑上使用是安全的（OpenClaw 执行每个操作前都会请求确认），但**不要在生产服务器上使用 `full` 模式**。如果只需要聊天功能（如给家人使用），设为 `messaging` 即可。
:::

<details>
<summary>为特定 Agent 单独设置配置档</summary>

如果你运行多个 Agent（如一个全能助手 + 一个客服机器人），可以为每个 Agent 设置不同的配置档：

```json
{
  "tools": { "profile": "full" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

上面的配置让默认 Agent 拥有全部工具，而 `support` Agent 只能收发消息和使用 Slack。

</details>

<details>
<summary>在配置文件中手动设置</summary>

编辑 `~/.openclaw/openclaw.json`（Windows：`C:\Users\你的用户名\.openclaw\openclaw.json`）：

```json
{
  "tools": {
    "profile": "full"
  }
}
```

修改后运行 `openclaw gateway restart` 生效。

</details>

## 2. 内置工具一览

OpenClaw 内置了丰富的工具，按功能分为以下几类：

| 分类 | 工具 | 说明 |
|------|------|------|
| **文件操作** | `read` `write` `edit` `apply_patch` | 读写文件，批量打补丁 |
| **命令执行** | `exec` `process` | 运行 Shell 命令、管理后台进程 |
| **网页** | `web_search` `web_fetch` | 搜索引擎查询、抓取网页内容 |
| **浏览器** | `browser` | 完整的浏览器自动化（点击、输入、截图） |
| **消息** | `message` | 跨渠道收发消息（飞书/QQ/Telegram 等） |
| **定时任务** | `cron` | 创建和管理定时 / 周期性任务 |
| **画布** | `canvas` | 驱动配套 App 的画布功能 |
| **设备** | `nodes` | 发现配套设备、拍照、录屏、获取位置 |
| **图片/PDF** | `image` `pdf` | 分析图片内容、解析 PDF 文档 |
| **网关** | `gateway` | 重启网关、查看和修改配置 |
| **会话** | `sessions_*` `agents_list` | 管理对话会话、生成子 Agent |

> 你不需要逐个记住这些工具——龙虾会根据你的指令自动选择合适的工具。你只需确保对应的工具已启用（配置档为 `full` 时全部可用）。

<details>
<summary>工具组（快捷分组）速查表</summary>

OpenClaw 提供预定义的工具组，方便在配置中批量引用：

| 工具组 | 包含的工具 |
|--------|-----------|
| `group:fs` | read, write, edit, apply_patch |
| `group:runtime` | exec, bash, process |
| `group:web` | web_search, web_fetch |
| `group:ui` | browser, canvas |
| `group:sessions` | sessions_list / history / send / spawn, session_status |
| `group:memory` | memory_search, memory_get |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:nodes` | nodes |
| `group:openclaw` | 所有内置工具（不含插件） |

在后续的权限管理中会用到这些组名。

</details>

## 3. 联网搜索

联网搜索让龙虾可以查询实时信息，而不仅仅依赖训练数据中的旧知识。

### 3.1 支持的搜索引擎

| 搜索引擎 | 说明 |
|---------|------|
| **Brave** | 隐私友好，免费额度充足，推荐入门使用 |
| **Perplexity** | AI 增强搜索，结果质量高 |
| **Kimi** | 国内可用，中文搜索优化 |
| **Gemini** | Google 搜索能力 |
| **Grok** | xAI 提供的搜索 |

### 3.2 配置搜索

最简单的方式是通过配置向导：

```bash
openclaw configure --section web
```

向导会引导你选择搜索引擎并填写 API Key。

<details>
<summary>手动编辑配置文件</summary>

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "brave",
        "maxResults": 5
      },
      "fetch": {
        "enabled": true,
        "maxCharsCap": 50000
      }
    }
  },
  "env": {
    "BRAVE_API_KEY": "你的 Brave API Key"
  }
}
```

修改后运行 `openclaw gateway restart` 生效。

> 搜索结果默认缓存 15 分钟，避免重复请求浪费 API 额度。

</details>

### 3.3 使用示例

配置完成后，直接在对话中提问即可——龙虾会自动判断是否需要联网：

```
今天有什么科技新闻？
```

```
帮我搜索"OpenClaw 最新版本"的更新内容
```

龙虾会自动调用 `web_search` 查询，并将搜索结果整合到回答中。

> **网页抓取**：除了搜索，`web_fetch` 工具可以从指定 URL 抓取网页内容并转为文字。对于 JavaScript 渲染的动态网页（如单页应用），建议使用下一节介绍的浏览器工具。

## 4. 浏览器自动化

浏览器工具让龙虾可以像人一样操作网页——打开页面、点击按钮、填写表单、截取屏幕。

### 4.1 启用浏览器

浏览器工具默认已启用（`browser.enabled: true`）。确认状态：

```bash
openclaw config get browser.enabled
```

如果返回 `false`，手动启用：

```bash
openclaw config set browser.enabled true
openclaw gateway restart
```

> **前提**：系统需要安装 Chrome 或 Chromium 浏览器。大多数电脑已经预装了 Chrome。

### 4.2 使用示例

在对话中直接告诉龙虾你想做什么：

```
帮我打开百度搜索"今日天气"，截图给我看
```

```
打开 https://example.com ，找到登录按钮并点击
```

龙虾会自动完成以下流程：

1. **启动浏览器** → 打开一个受控的 Chrome 实例
2. **打开网页** → 导航到目标 URL
3. **获取页面快照** → 识别页面上所有可操作的元素
4. **执行操作** → 点击、输入、选择等
5. **截图确认** → 把结果截图发给你

<details>
<summary>浏览器支持的操作</summary>

| 操作 | 说明 |
|------|------|
| `status` | 查看浏览器是否运行 |
| `start` / `stop` | 启动 / 关闭浏览器 |
| `open` | 打开指定 URL |
| `tabs` / `focus` / `close` | 标签页管理 |
| `snapshot` | 获取页面快照（识别可操作元素） |
| `screenshot` | 截取当前页面图片 |
| `act` | 交互操作：click / type / press / hover / drag / fill / resize / wait |
| `navigate` | 前进 / 后退 / 刷新 |
| `console` | 查看浏览器控制台日志 |
| `pdf` | 将当前页面导出为 PDF |
| `upload` | 上传文件 |

> `snapshot` 有两种模式：`ai`（默认，使用 Playwright 分析页面结构）和 `aria`（返回无障碍树）。`act` 操作需要引用 `snapshot` 返回的元素编号。

</details>

<details>
<summary>浏览器多实例（Profile）管理</summary>

如果你需要同时登录多个账号（如个人 + 工作），可以创建多个浏览器实例：

```json
{
  "browser": {
    "enabled": true,
    "defaultProfile": "chrome",
    "profiles": {
      "chrome": { "port": 18800 },
      "work": { "port": 18801 }
    }
  }
}
```

每个 Profile 有独立的 Cookie 和登录状态。在对话中指定使用哪个：

```
用 work 浏览器打开飞书文档
```

Profile 命名规则：小写字母 + 数字 + 短横线，最长 64 字符。端口范围 18800–18899（最多约 100 个实例）。

</details>

## 5. 定时任务（Cron）

Cron 让龙虾按计划自动执行任务——比如每天早上发天气预报、每周生成工作汇报、每隔一段时间检查邮件。

### 5.1 创建定时任务

```bash
# 每天早上 8 点发天气预报
openclaw cron add --name "天气预报" --cron "0 8 * * *" \
  --message "查询今天的天气，发送给我" \
  --channel "feishu:chat:你的ChatID"

# 每 30 分钟检查一次
openclaw cron add --name "定期检查" --every 30m \
  --message "检查有没有需要我处理的事情"

# 20 分钟后提醒我（一次性）
openclaw cron add --name "提醒" --at 20m \
  --message "提醒：该休息一下了！"
```

三种调度方式：
- `--cron "表达式"`：标准 Cron 表达式，精确控制执行时间
- `--every 间隔`：固定周期重复（如 `10m`、`1h`、`6h`）
- `--at 延时`：从现在起延迟一次性执行

<details>
<summary>Cron 表达式速查</summary>

Cron 表达式格式：`分 时 日 月 周`

| 表达式 | 含义 |
|--------|------|
| `0 8 * * *` | 每天早上 8:00 |
| `0 9 * * 1-5` | 工作日（周一到周五）上午 9:00 |
| `*/30 * * * *` | 每 30 分钟 |
| `0 20 * * 5` | 每周五晚上 8:00 |
| `0 0 1 * *` | 每月 1 日零点 |

如果不熟悉 Cron 表达式，用 `--every`（周期间隔）或 `--at`（延时一次）更直观。

</details>

### 5.2 管理定时任务

```bash
# 查看所有任务
openclaw cron list

# 手动触发一次
openclaw cron run 天气预报

# 查看执行历史
openclaw cron runs --id <taskID>

# 暂停 / 恢复任务
openclaw cron disable 天气预报
openclaw cron enable 天气预报

# 编辑任务
openclaw cron edit <jobId>

# 删除任务
openclaw cron rm <jobId>
```

> **注意**：Cron 任务依赖 Gateway 持续运行。如果 Gateway 关闭（如电脑关机），任务不会执行。重新启动后任务会自动恢复。

> **`--channel` 参数格式**因平台而异，例如：`feishu:chat:<ChatID>`、`telegram:chat:<ChatID>`、`qqbot:group:<groupid>`。完整格式见[附录 F 命令速查表](/cn/appendix/appendix-f)。

## 6. 命令执行（Exec）

`exec` 工具让龙虾直接在你的电脑上运行 Shell 命令。这是它"干活"最核心的能力——安装软件、处理文件、运行脚本等都靠它。

### 使用示例

```
帮我创建一个文件叫 hello.txt，写上今天的日期和"Hello from OpenClaw!"
```

```
用 Python 写一个猜数字小游戏，保存为 game.py 并运行它
```

龙虾会自动使用 `exec` 执行命令，每次执行前会请求你确认。

<details>
<summary>exec 工具的核心参数</summary>

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `command` | 要执行的命令（必填） | — |
| `timeout` | 超时秒数，超时后终止 | 1800（30 分钟） |
| `background` | 立即后台运行 | false |
| `yieldMs` | 超过该毫秒数后自动切后台 | 10000（10 秒） |
| `pty` | 是否需要真实终端 | false |

后台运行的命令可以通过 `process` 工具管理：

| 操作 | 说明 |
|------|------|
| `list` | 列出所有后台进程 |
| `poll` | 获取新输出和退出状态 |
| `log` | 查看输出日志 |
| `write` | 向进程写入输入 |
| `kill` | 终止进程 |
| `clear` / `remove` | 清理已完成的进程 |

</details>

<details>
<summary>exec 安全策略</summary>

exec 工具有三种安全级别：

| 级别 | 说明 |
|------|------|
| `deny` | 完全禁止执行命令 |
| `allowlist` | 仅允许白名单中的命令 |
| `full` | 允许执行任何命令（需用户确认） |

如果你需要限制龙虾可以执行的命令范围，可以在 `openclaw.json` 中配置 `tools.exec.security` 和对应的白名单。

</details>

## 7. 工具权限管理

有时你需要限制龙虾的能力范围——比如不让它操作浏览器，或者只允许文件操作。

### 7.1 允许 / 禁止特定工具

通过 `tools.allow` 和 `tools.deny` 精确控制。编辑 `~/.openclaw/openclaw.json`：

**禁止浏览器**：

```json
{
  "tools": {
    "deny": ["browser"]
  }
}
```

**只允许文件操作和搜索**：

```json
{
  "tools": {
    "allow": ["group:fs", "group:web"]
  }
}
```

> **规则**：`deny` 优先于 `allow`。匹配不区分大小写，支持 `*` 通配符（`"*"` 表示所有工具）。

修改后运行 `openclaw gateway restart` 生效。

<details>
<summary>按模型提供商限制工具</summary>

如果你使用多个模型提供商，可以为不同提供商设置不同的工具权限。通过 `tools.byProvider` 配置：

```json
{
  "tools": {
    "profile": "coding",
    "byProvider": {
      "google-antigravity": { "profile": "minimal" }
    }
  }
}
```

`byProvider` 只能**收窄**权限，不能超出全局配置档的范围。支持 `provider`（如 `siliconflow`）或 `provider/model`（如 `openai/gpt-5.2`）两种粒度。

</details>

<details>
<summary>工具循环检测（Loop Detection）</summary>

如果龙虾陷入反复调用同一工具的死循环，可以启用循环检测：

```json
{
  "tools": {
    "loopDetection": {
      "enabled": true,
      "warningThreshold": 10,
      "criticalThreshold": 20,
      "globalCircuitBreakerThreshold": 30
    }
  }
}
```

检测器会识别三种循环模式：

- **重复调用**：相同工具 + 相同参数反复调用
- **轮询无进展**：反复检查状态但结果不变
- **乒乓模式**：两个工具交替调用却无进展

默认关闭。如果你发现龙虾偶尔会"卡住"反复做同一件事，可以启用此功能。

</details>

## 8. 插件工具

除了内置工具，OpenClaw 还支持通过插件扩展更多能力：

| 插件 | 说明 |
|------|------|
| **Lobster** | 工作流运行时，支持多步骤任务和可恢复的审批流程 |
| **LLM Task** | JSON 格式的 LLM 调用，用于结构化输出（可选 Schema 校验） |
| **Diffs** | 文件差异查看器，支持 PNG / PDF 渲染 |
| **Voice Call** | 语音通话插件 |
| **Zalo Personal** | Zalo 个人账号插件 |

插件可以注册自己的工具和 CLI 命令。安装和配置详见[附录 G 配置文件详解](/cn/appendix/appendix-g)。

<details>
<summary>工具如何呈现给 AI 模型</summary>

OpenClaw 通过两个通道让模型"看到"工具：

1. **系统提示词**：人类可读的工具列表 + 使用指南
2. **工具 Schema**：结构化的函数定义，发送到模型 API

模型同时看到"有哪些工具"和"如何调用它们"。如果一个工具既不在系统提示词中也不在 Schema 中，模型就无法调用它。这就是 `tools.allow` / `tools.deny` 的工作原理——被禁止的工具不会被发送给模型。

</details>

## 9. 常见问题

**Q: 龙虾只会聊天，不执行任何操作？**

A: 工具配置档可能是 `messaging` 或 `minimal`。修复：

```bash
openclaw config set tools.profile full
openclaw gateway restart
```

**Q: 联网搜索不可用？**

A: 需要配置搜索引擎 API Key。运行 `openclaw configure --section web` 进行设置。没有 API Key 之前，搜索功能无法使用。

**Q: 浏览器工具报错？**

A: 先确认浏览器已启用（`openclaw config get browser.enabled`），再确保系统安装了 Chrome 或 Chromium。Linux 服务器（无图形界面）需要安装 Chromium headless 模式。WSL2 用户可能需要额外配置，详见[附录 G 配置文件详解](/cn/appendix/appendix-g)。

**Q: 定时任务没有执行？**

A: 检查 Gateway 是否在运行（`openclaw status`）。Cron 任务依赖 Gateway 持续运行——电脑关机或 Gateway 停止期间的任务不会补执行。

**Q: 如何知道龙虾调用了哪些工具？**

A: 在 Web 控制面板（`openclaw dashboard`）中查看对话详情可以看到每次工具调用。也可以查看日志：

```bash
openclaw logs --limit 50
```
