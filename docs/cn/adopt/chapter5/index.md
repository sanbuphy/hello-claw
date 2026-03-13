---
prev:
  text: '第4章 聊天平台接入'
  link: '/cn/adopt/chapter4'
next:
  text: '第6章 智能体管理'
  link: '/cn/adopt/chapter6'
---

# 第五章 模型管理

> 本章带你了解 OpenClaw 的模型系统——如何选择、切换、配置模型提供商，以及当一个模型挂了时如何自动切换到备用方案。

> **前置条件**：已完成[第二章](/cn/adopt/chapter2/)的安装和基础模型配置。本章在此基础上深入讲解模型管理的进阶用法。

---

## 0. 模型是什么？为什么要配置？

OpenClaw 本身不包含 AI 大脑——它需要连接一个**模型提供商**（如 OpenRouter、硅基流动、Anthropic、OpenAI 等）来获得智能。你在[第二章](/cn/adopt/chapter2/#_2-配置-ai-模型)已经配置了第一个模型，现在我们来了解更多玩法。

**模型标识格式**：`provider/model`（提供商/模型名），例如：
- `openrouter/stepfun/step-3.5-flash:free`（OpenRouter 上的免费模型）
- `anthropic/claude-opus-4-6`（Anthropic 的 Claude Opus）
- `openai/gpt-5.1-codex`（OpenAI 的 GPT 5.1）
- `ollama/llama3.3`（本地运行的 Llama 模型）

> **小白提示**：可以把"模型"理解为不同品牌的 AI 大脑，"提供商"就是卖这些大脑的商店。有的商店贵但聪明，有的免费但简单——OpenClaw 让你自由组合。

---

## 1. 快速上手

### 最简配置

编辑 `~/.openclaw/openclaw.json`（Windows：`C:\Users\你的用户名\.openclaw\openclaw.json`）：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6"
      }
    }
  }
}
```

保存后重启网关生效：

```bash
openclaw gateway restart
```

### 用向导配置（推荐新手）

不想手动编辑配置文件？运行向导即可：

```bash
openclaw onboard
```

向导会引导你选择提供商、输入 API Key、设置默认模型——和[第二章](/cn/adopt/chapter2/#_2-配置-ai-模型)的流程一样。

---

## 2. 核心概念

### 模型选择优先级

当你发送一条消息时，OpenClaw 按以下顺序选择模型：

```
1. 主模型（agents.defaults.model.primary）
   ↓ 如果主模型不可用
2. 备用模型列表（agents.defaults.model.fallbacks），按顺序尝试
   ↓ 每个模型内部
3. 认证配置轮换（同一提供商的多个 API Key 之间轮换）
```

### 模型选择策略

- **主模型**：设为你能用到的最强模型（如最新一代的旗舰模型）
- **备用模型**：用于成本/延迟敏感的场景，或主模型挂了时的兜底
- **安全提示**：如果 Agent 需要调用工具（Tool）或处理不可信输入，避免使用老旧/弱小的模型

### 图片模型

当主模型不支持图片输入时，OpenClaw 会自动使用 `agents.defaults.imageModel` 来处理图片：

```json5
{
  agents: {
    defaults: {
      imageModel: {
        primary: "openai/gpt-5.1-codex",
        fallbacks: ["google/gemini-3-pro-preview"]
      }
    }
  }
}
```

<details>
<summary>模型白名单（allowlist）</summary>

如果你设置了 `agents.defaults.models`，它就变成了**白名单**——只有列表中的模型才能使用。用户尝试切换到白名单外的模型时，会收到提示：

> Model "provider/model" is not allowed. Use /model to list available models.

白名单配置示例：

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4-5" },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      }
    }
  }
}
```

设置别名后，用户可以在聊天中用 `/model Sonnet` 快速切换，而不用记住完整的模型 ID。

如果不设置 `agents.defaults.models`，则没有白名单限制，用户可以自由切换任何模型。

</details>

---

## 3. 聊天中切换模型

你可以在聊天过程中随时切换模型，无需重启：

| 命令 | 说明 |
|------|------|
| `/model` | 打开模型选择器（数字列表） |
| `/model list` | 同上，显示可用模型列表 |
| `/model 3` | 选择列表中的第 3 个模型 |
| `/model openai/gpt-5.2` | 直接指定模型 |
| `/model status` | 查看当前模型详情（含认证状态） |

> **注意**：模型 ID 以第一个 `/` 分隔提供商和模型名。如果模型名本身包含 `/`（如 OpenRouter 的模型），必须加上提供商前缀：`/model openrouter/moonshotai/kimi-k2`。

<details>
<summary>Discord 特殊交互</summary>

在 Discord 上，`/model` 和 `/models` 会打开一个交互式选择器，包含提供商和模型的下拉菜单 + Submit 按钮，体验更直观。

</details>

---

## 4. CLI 模型管理

除了在聊天中切换，你还可以用命令行管理模型：

### 查看与设置

```bash
# 查看当前模型状态（主模型 + 备用 + 认证概览）
openclaw models status

# 查看已配置的模型列表
openclaw models list

# 查看所有可用模型（完整目录）
openclaw models list --all

# 只看本地模型
openclaw models list --local

# 按提供商筛选
openclaw models list --provider openai

# 设置主模型
openclaw models set openai/gpt-5.1-codex

# 设置图片模型
openclaw models set-image google/gemini-3-pro-preview
```

### 别名管理

给常用模型起个好记的名字：

```bash
# 添加别名
openclaw models aliases add Sonnet anthropic/claude-sonnet-4-5
openclaw models aliases add Opus anthropic/claude-opus-4-6

# 查看所有别名
openclaw models aliases list

# 删除别名
openclaw models aliases remove Sonnet
```

### 备用模型管理

```bash
# 查看备用模型列表
openclaw models fallbacks list

# 添加备用模型（按添加顺序排列优先级）
openclaw models fallbacks add openai/gpt-5.2
openclaw models fallbacks add google/gemini-3-pro-preview

# 删除某个备用模型
openclaw models fallbacks remove openai/gpt-5.2

# 清空所有备用模型
openclaw models fallbacks clear

# 图片备用模型管理（同理）
openclaw models image-fallbacks list
openclaw models image-fallbacks add google/gemini-3-pro-preview
openclaw models image-fallbacks clear
```

<details>
<summary>models status 的详细输出说明</summary>

`openclaw models status` 输出包含：

- **当前主模型**和**备用模型**
- **认证概览**：每个提供商的认证状态
- **OAuth 到期状态**：24 小时内即将过期的 OAuth Token 会有警告
- **缺失认证**：已配置但缺少凭证的提供商会显示 "Missing auth"

有用的标志：
- `--plain`：只输出当前主模型名（适合脚本使用）
- `--json`：机器可读的 JSON 格式
- `--check`：自动化检查（缺失/过期返回 exit 1，即将过期返回 exit 2）

在远程服务器上，可以用 `--check` 配合监控脚本自动检测认证问题。

</details>

<details>
<summary>扫描 OpenRouter 免费模型</summary>

OpenClaw 提供了一个扫描工具，可以自动发现 OpenRouter 上的免费模型并按质量排序：

```bash
# 扫描免费模型（会实际探测模型能力）
openclaw models scan

# 仅查看元数据，不探测
openclaw models scan --no-probe

# 过滤条件
openclaw models scan --min-params 70    # 最少 700 亿参数
openclaw models scan --max-age-days 30  # 最近 30 天的模型
openclaw models scan --provider meta    # 只看 Meta 的模型

# 直接设为默认模型
openclaw models scan --set-default
```

排名依据（优先级从高到低）：图片支持 → 工具调用延迟 → 上下文窗口大小 → 参数量。

> 扫描需要 OpenRouter API Key（从认证配置或 `OPENROUTER_API_KEY` 环境变量获取）。没有 Key 时用 `--no-probe` 仅查看列表。

</details>

---

## 5. 模型提供商配置

OpenClaw 支持两类提供商：

- **内置提供商**：OpenClaw 自带目录，只需设置认证即可使用
- **自定义提供商**：通过 `models.providers` 配置，适合国产模型、本地模型、代理网关等

### 5.1 内置提供商

以下提供商无需额外配置 `models.providers`，直接设置 API Key 即可：

| 提供商 | provider 标识 | 认证方式 | 示例模型 |
|--------|-------------|---------|---------|
| OpenAI | `openai` | `OPENAI_API_KEY` | `openai/gpt-5.1-codex` |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | `anthropic/claude-opus-4-6` |
| OpenAI Code (Codex) | `openai-codex` | OAuth 登录 | `openai-codex/gpt-5.3-codex` |
| OpenCode Zen | `opencode` | `OPENCODE_API_KEY` | `opencode/claude-opus-4-6` |
| Google Gemini | `google` | `GEMINI_API_KEY` | `google/gemini-3-pro-preview` |
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` | `openrouter/anthropic/claude-sonnet-4-5` |
| Z.AI (GLM) | `zai` | `ZAI_API_KEY` | `zai/glm-5` |
| xAI | `xai` | `XAI_API_KEY` | — |
| Mistral | `mistral` | `MISTRAL_API_KEY` | `mistral/mistral-large-latest` |
| Groq | `groq` | `GROQ_API_KEY` | — |
| Cerebras | `cerebras` | `CEREBRAS_API_KEY` | — |
| GitHub Copilot | `github-copilot` | `GH_TOKEN` | — |
| Hugging Face | `huggingface` | `HF_TOKEN` | `huggingface/deepseek-ai/DeepSeek-R1` |
| Kilo Gateway | `kilocode` | `KILOCODE_API_KEY` | `kilocode/anthropic/claude-opus-4.6` |
| Vercel AI Gateway | `vercel-ai-gateway` | `AI_GATEWAY_API_KEY` | `vercel-ai-gateway/anthropic/claude-opus-4.6` |

> **更多提供商**及获取地址见[附录 E 模型提供商选型指南](/cn/appendix/appendix-e)。

<details>
<summary>OpenAI 配置详情</summary>

```json5
{
  agents: {
    defaults: { model: { primary: "openai/gpt-5.1-codex" } }
  }
}
```

**认证**：设置环境变量 `OPENAI_API_KEY`，或在向导中选择 `openai-api-key`。

**传输协议**：默认 `auto`（WebSocket 优先，SSE 兜底）。可按模型覆盖：

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.1-codex": {
          params: { transport: "sse" }  // 强制使用 SSE
        }
      }
    }
  }
}
```

CLI 快捷设置：`openclaw onboard --auth-choice openai-api-key`

</details>

<details>
<summary>Anthropic 配置详情</summary>

```json5
{
  agents: {
    defaults: { model: { primary: "anthropic/claude-opus-4-6" } }
}
```

**认证方式（二选一）**：

1. **API Key（推荐）**：设置 `ANTHROPIC_API_KEY`
2. **setup-token**：运行 `claude setup-token`，然后 `openclaw models status` 确认

> **注意**：setup-token 是技术兼容方案，Anthropic 曾限制过在 Claude Code 以外使用订阅凭证。建议优先使用 API Key。

CLI 快捷设置：`openclaw onboard --auth-choice token`

</details>

<details>
<summary>OpenAI Code (Codex) 配置详情</summary>

OpenAI Codex 使用 OAuth 登录（ChatGPT 账号），**明确支持**在 OpenClaw 等外部工具中使用。

```json5
{
  agents: {
    defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } }
  }
}
```

CLI 快捷设置：`openclaw onboard --auth-choice openai-codex` 或 `openclaw models auth login --provider openai-codex`

</details>

<details>
<summary>Google Vertex / Antigravity / Gemini CLI</summary>

除了 API Key 方式的 Google Gemini，还有三个 Google 相关提供商：

| 提供商 | 标识 | 认证方式 |
|--------|------|---------|
| Google Vertex | `google-vertex` | gcloud ADC |
| Antigravity | `google-antigravity` | OAuth 插件 |
| Gemini CLI | `google-gemini-cli` | OAuth 插件 |

**Antigravity / Gemini CLI** 是非官方集成，需先启用插件：

```bash
# Antigravity
openclaw plugins enable google-antigravity-auth
openclaw models auth login --provider google-antigravity --set-default

# Gemini CLI
openclaw plugins enable google-gemini-cli-auth
openclaw models auth login --provider google-gemini-cli --set-default
```

> **风险提示**：部分用户反映使用第三方客户端后 Google 账号受限。建议使用非关键账号，并自行评估 Google 服务条款。

</details>

### 5.2 国产模型提供商

以下是常用的国产模型配置方式：

#### 月之暗面 Kimi（Moonshot AI）

Moonshot 使用 OpenAI 兼容接口：

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      }
    }
  }
}
```

可用模型：`kimi-k2.5`、`kimi-k2-0905-preview`、`kimi-k2-turbo-preview`、`kimi-k2-thinking`、`kimi-k2-thinking-turbo`

#### Kimi Coding

Kimi Coding 使用 Anthropic 兼容接口：

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  }
}
```

#### 火山引擎（豆包 Doubao）

```json5
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  }
}
```

CLI：`openclaw onboard --auth-choice volcengine-api-key`

<details>
<summary>火山引擎可用模型列表</summary>

**标准模型**（provider: `volcengine`）：
| 模型 ID | 名称 |
|---------|------|
| `volcengine/doubao-seed-1-8-251228` | 豆包 Seed 1.8 |
| `volcengine/doubao-seed-code-preview-251028` | 豆包 Seed Code |
| `volcengine/kimi-k2-5-260127` | Kimi K2.5 |
| `volcengine/glm-4-7-251222` | GLM 4.7 |
| `volcengine/deepseek-v3-2-251201` | DeepSeek V3.2 128K |

**编码模型**（provider: `volcengine-plan`）：
| 模型 ID | 名称 |
|---------|------|
| `volcengine-plan/ark-code-latest` | ARK Code |
| `volcengine-plan/doubao-seed-code` | 豆包 Seed Code |
| `volcengine-plan/kimi-k2.5` | Kimi K2.5 |
| `volcengine-plan/kimi-k2-thinking` | Kimi K2 Thinking |
| `volcengine-plan/glm-4.7` | GLM 4.7 |

</details>

#### 通义千问 Qwen（免费 OAuth）

Qwen 提供 OAuth 设备码认证，可免费使用 Qwen Coder + Vision 模型：

```bash
# 启用插件并登录
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

可用模型：`qwen-portal/coder-model`、`qwen-portal/vision-model`

#### BytePlus（国际版火山引擎）

面向国际用户，与火山引擎共享模型目录：

```json5
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  }
}
```

CLI：`openclaw onboard --auth-choice byteplus-api-key`

<details>
<summary>更多国产提供商</summary>

OpenClaw 还支持以下国产提供商（配置方式类似，通过 `models.providers` 添加）：

- **硅基流动 SiliconFlow**：`https://api.siliconflow.cn/v1`（OpenAI 兼容）
- **深度求索 DeepSeek**：`https://api.deepseek.com/v1`（OpenAI 兼容）
- **阶跃星辰 StepFun**：OpenAI 兼容
- **稀宇科技 MiniMax**：`openclaw onboard --auth-choice minimax-api`
- **智谱 Z.AI**：内置提供商，`ZAI_API_KEY`
- **混元（腾讯）**：OpenAI 兼容
- **文心一言（百度）**：OpenAI 兼容

配置模式参考[第二章备选方案：硅基流动](/cn/adopt/chapter2/#_2-配置-ai-模型)的格式，将 `baseUrl` 和 `apiKey` 替换为对应提供商即可。

</details>

### 5.3 本地模型

#### Ollama

Ollama 是最流行的本地模型运行时，OpenClaw 自动检测本地运行的 Ollama：

```bash
# 安装 Ollama 并拉取模型
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

> Ollama 在本地 `http://127.0.0.1:11434/v1` 运行时会被自动发现，无需额外配置。

#### vLLM

vLLM 是高性能的本地推理服务器：

```bash
# 设置环境变量启用自动发现
export VLLM_API_KEY="vllm-local"
```

```json5
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } }
  }
}
```

默认连接 `http://127.0.0.1:8000/v1`。

### 5.4 自定义提供商（通用方式）

任何 OpenAI 或 Anthropic 兼容的 API 都可以通过 `models.providers` 接入：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [{
          id: "minimax-m2.5-gs32",
          name: "MiniMax M2.5",
          contextWindow: 200000,
          maxTokens: 8192,
        }]
      }
    }
  }
}
```

<details>
<summary>自定义提供商的可选字段</summary>

在 `models` 数组中，每个模型对象支持以下字段（均可省略）：

| 字段 | 默认值 | 说明 |
|------|--------|------|
| `reasoning` | `false` | 是否支持推理/思考链 |
| `input` | `["text"]` | 支持的输入类型 |
| `cost` | 全为 0 | `{ input, output, cacheRead, cacheWrite }` |
| `contextWindow` | `200000` | 上下文窗口大小 |
| `maxTokens` | `8192` | 最大输出 Token 数 |

建议设置与你的代理/模型实际限制匹配的值。

**API 兼容模式**（`api` 字段）：
- `openai-completions`：OpenAI Chat Completions 兼容
- `anthropic-messages`：Anthropic Messages 兼容

</details>

<details>
<summary>models.json 注册表机制</summary>

自定义提供商配置会写入 `~/.openclaw/agents/<agentId>/models.json`。默认使用 `merge` 模式（与内置目录合并），也可设置 `models.mode: "replace"` 完全替换。

合并优先级规则：
- 已有的非空 `baseUrl` 优先
- 非 SecretRef 管理的 `apiKey` 优先
- SecretRef 管理的 Key 会从源标记（环境变量名等）刷新，不直接存储明文
- 其他字段从配置和标准目录刷新

</details>

---

## 6. API Key 轮换

当你有**多个 API Key** 时，OpenClaw 支持自动轮换，遇到限流（429）时自动切换到下一个 Key：

### 配置方式

通过环境变量配置（以 OpenAI 为例）：

```bash
# 方式一：逗号分隔列表
OPENAI_API_KEYS="sk-key1,sk-key2,sk-key3"

# 方式二：编号列表
OPENAI_API_KEY_1="sk-key1"
OPENAI_API_KEY_2="sk-key2"

# 方式三：单个实时覆盖（最高优先级）
OPENCLAW_LIVE_OPENAI_KEY="sk-hot-key"
```

### 优先级

```
OPENCLAW_LIVE_<PROVIDER>_KEY  ← 最高（单个覆盖）
<PROVIDER>_API_KEYS            ← 逗号列表
<PROVIDER>_API_KEY             ← 主 Key
<PROVIDER>_API_KEY_*           ← 编号列表
```

### 轮换规则

- **只在限流错误**（429、rate_limit、quota、resource exhausted）时轮换
- 非限流错误直接失败，不尝试下一个 Key
- 所有 Key 都失败时，返回最后一个 Key 的错误
- Google 提供商额外支持 `GOOGLE_API_KEY` 作为兜底

---

## 7. 模型故障转移（Failover）

这是 OpenClaw 最强大的可靠性功能之一——当模型或提供商出故障时，自动切换到备用方案，用户完全无感。

### 两阶段故障转移

```
阶段一：认证配置轮换
  同一提供商内的多个 API Key / OAuth 配置之间轮换
  ↓ 所有配置都失败
阶段二：模型备用
  切换到 agents.defaults.model.fallbacks 中的下一个模型
```

### 配置示例

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openai/gpt-5.1-codex",
          "google/gemini-3-pro-preview"
        ]
      }
    }
  }
}
```

当 Anthropic 的所有认证配置都失败后，自动切换到 OpenAI；OpenAI 也失败则切换到 Google。

### 认证配置（Auth Profiles）

OpenClaw 使用**认证配置**管理 API Key 和 OAuth Token：

- 存储位置：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- 支持两种类型：`api_key`（API 密钥）和 `oauth`（OAuth Token）

<details>
<summary>认证配置轮换规则</summary>

**选择顺序**：
1. 显式配置：`auth.order[provider]`（如果设置了）
2. 配置中的 profiles：`auth.profiles` 按提供商筛选
3. 存储的 profiles：`auth-profiles.json` 中的条目

**默认轮换策略**（无显式配置时）：
- 优先使用 OAuth（在 API Key 之前）
- 同类型中，最久未使用的优先
- 冷却中/禁用的配置排到最后

**Profile ID 命名规则**：
- 默认：`provider:default`
- OAuth 带邮箱：`provider:email@example.com`

</details>

<details>
<summary>会话粘性（Session Stickiness）</summary>

为了保持提供商缓存命中率，OpenClaw 会在同一会话内**固定**使用选中的认证配置，不会每次请求都轮换。固定配置在以下情况失效：

- 会话重置（`/new` 或 `/reset`）
- 压缩完成（compaction）
- 当前配置进入冷却/禁用状态

**手动锁定**：在聊天中用 `/model …@<profileId>` 锁定特定配置。锁定后如果该配置失败，OpenClaw 会直接跳到下一个模型（fallback），而不是切换同提供商的其他配置。

**常见困惑**：如果你同时配置了 OAuth 和 API Key，轮换可能在两者之间切换，导致行为看起来不一致。解决方法：
- 用 `auth.order[provider]` 固定顺序
- 或用 `/model …@profileId` 锁定单个配置

</details>

### 冷却机制（Cooldown）

当认证配置因错误失败时，OpenClaw 会将其标记为"冷却中"并自动切换到下一个。冷却时间使用**指数退避**：

| 失败次数 | 冷却时间 |
|---------|---------|
| 第 1 次 | 1 分钟 |
| 第 2 次 | 5 分钟 |
| 第 3 次 | 25 分钟 |
| 第 4 次+ | 1 小时（上限） |

触发冷却的错误类型：认证错误、限流（429）、超时、格式错误（如工具调用 ID 验证失败）、OpenAI 兼容的 stop-reason 错误。

<details>
<summary>计费禁用（Billing Disables）</summary>

余额不足（如 "insufficient credits"、"credit balance too low"）触发的不是短冷却，而是**长期禁用**：

- 起始退避：5 小时
- 每次失败翻倍，上限 24 小时
- 24 小时无失败后自动重置计数器

状态存储在 `auth-profiles.json`：

```json5
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

> **实用建议**：对于 24/7 运行的 Gateway，API Key 认证通常比 OAuth 更稳定可预测。如果使用 OpenRouter，建议开启 Auto Top Up 避免余额用尽导致长时间禁用。

</details>

---

## 8. 常见问题

**Q: 切换模型后没有回复？**

A: 可能触发了模型白名单限制。如果设置了 `agents.defaults.models`，只有白名单中的模型才能使用。解决方法：
1. 将模型添加到白名单
2. 删除 `agents.defaults.models` 取消白名单
3. 用 `/model list` 查看可用模型

**Q: 模型标识中的 `/` 怎么处理？**

A: OpenClaw 以第一个 `/` 分隔提供商和模型名。如果模型名本身包含 `/`（如 OpenRouter 上的 `moonshotai/kimi-k2`），必须加上提供商前缀：`openrouter/moonshotai/kimi-k2`。省略提供商前缀时，OpenClaw 会当作别名或默认提供商的模型处理。

**Q: 如何确认当前用的是哪个模型？**

A: 运行 `openclaw models status` 或在聊天中发送 `/model status`，会显示当前主模型、备用模型、认证状态的完整信息。

**Q: 本地模型（Ollama）和云端模型可以混用吗？**

A: 可以。常见做法是将本地模型作为 fallback——云端模型挂了或限流时自动切换到本地：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["ollama/llama3.3"]
      }
    }
  }
}
```

**Q: 多个 API Key 怎么配置最省钱？**

A: 利用 Key 轮换分散限流压力，再配合 fallback 实现成本梯度：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",      // 日常用便宜的
        fallbacks: ["anthropic/claude-opus-4-6"]      // 需要时用强的
      }
    }
  }
}
```

环境变量中配置多个 Key：`ANTHROPIC_API_KEYS="sk-key1,sk-key2"`
