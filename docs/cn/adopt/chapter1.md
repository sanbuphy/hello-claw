# 第一章 十分钟上手 OpenClaw

这一章会带你完成 OpenClaw 的安装，从环境准备到运行配置向导，整个过程不超过 10 分钟。

## 1. 系统要求

- **操作系统**：macOS、Linux、或 Windows（推荐 WSL2）
- **Node.js**：22 或更高版本
- **内存**：至少 1GB，推荐 4GB
- **端口**：18789 需要可用

检查 Node.js 版本：

```bash
node --version
```

如果版本低于 22，需要先升级 Node.js。

## 2. 安装 Node.js

### Windows 用户

**方案一：使用一键安装脚本（推荐）**

打开 PowerShell（管理员模式），运行：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

这个脚本会自动安装 Node.js 和 OpenClaw。如果遇到权限错误，先运行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**方案二：手动安装**

1. 下载 nvm-windows：访问 [github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases) 下载 `nvm-setup.exe`
2. 安装后重新打开 PowerShell（管理员模式）
3. 安装 Node.js 22：

```powershell
nvm install 22
nvm use 22
```

### macOS 用户

使用 Homebrew 安装：

```bash
brew install node@22
```

### Linux 用户

使用 nvm（推荐）：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
```

## 3. 安装 OpenClaw

全局安装 OpenClaw CLI：

```bash
npm install -g openclaw@latest
```

验证安装：

```bash
openclaw --version
```

看到版本号说明安装成功。

## 4. 运行配置向导

安装完成后，运行配置向导：

```bash
openclaw onboard --install-daemon
```

这个命令会启动交互式配置向导，并安装后台守护进程。

### 4.1 选择模型提供商

向导首先会询问你使用哪个 AI 模型：

```
◇  Model/auth provider
│  ● Anthropic (Claude)
│  ○ OpenAI (GPT)
│  ○ DeepSeek（国内推荐）
│  ○ OpenRouter（聚合多家模型）
```

**国内用户推荐**：
- **DeepSeek**：国内服务商，支持支付宝充值，注册即用
- **OpenRouter**：一个 Key 访问多家模型，支持多种支付方式

**国际用户**：选择 Anthropic 或 OpenAI，需要国际信用卡。

### 4.2 输入 API Key

根据选择的提供商输入 API Key：

```
◇  DeepSeek API key
│  （粘贴你的 API Key）
```

获取 API Key：

- DeepSeek: https://platform.deepseek.com
- OpenRouter: https://openrouter.ai
- Anthropic: https://console.anthropic.com
- OpenAI: https://platform.openai.com

### 4.3 配置聊天渠道（可选）

向导会询问是否配置 Slack、Telegram 等聊天渠道。如果暂时不需要，可以选择跳过，后续通过 `openclaw configure` 添加。

### 4.4 配置技能（可选）

向导会显示可用的技能列表，询问是否安装。建议先跳过，等熟悉基本操作后再安装。

配置完成后，向导会自动启动 Gateway 守护进程。

## 5. 验证安装

检查 Gateway 状态：

```bash
openclaw status
```

看到 `Gateway service: running` 说明安装成功。

打开 Web 控制面板：

```bash
openclaw dashboard
```

浏览器会自动打开 `http://localhost:18789`，你可以在这里和 OpenClaw 对话。

## 6. 第一次对话

在控制面板输入：

```
帮我列出当前目录的文件
```

如果 OpenClaw 能正确执行命令并返回结果，说明一切正常。

## 7. 常用命令

### 基础命令

```bash
# 查看整体状态
openclaw status

# 深度健康检查
openclaw status --deep

# 系统诊断和修复
openclaw doctor

# 查看版本
openclaw --version

# 查看帮助
openclaw --help
```

### Gateway 管理

```bash
# 查看 Gateway 状态
openclaw gateway status

# 启动 Gateway
openclaw gateway start

# 停止 Gateway
openclaw gateway stop

# 重启 Gateway（修改配置后必须执行）
openclaw gateway restart

# 前台运行并显示日志（调试用）
openclaw gateway run --verbose
```

### 模型管理

```bash
# 列出可用模型
openclaw models list

# 查看模型状态
openclaw models status

# 设置默认模型
openclaw models set <provider/model>

# 检查 API 认证
openclaw models auth
```

### 渠道管理

```bash
# 列出所有渠道
openclaw channels list

# 查看渠道连接状态
openclaw channels status

# 添加新渠道
openclaw channels add

# 登录渠道（如扫码登录 WhatsApp）
openclaw channels login

# 登出渠道
openclaw channels logout
```

### 日志和调试

```bash
# 实时查看日志
openclaw logs --follow

# 查看最近日志
openclaw logs
```

## 8. 常见问题

**Q: OpenClaw 只会聊天不干活，让它执行命令却只给建议？**

A: 这是 2026.3.2 版本后最常见的问题，原因是 Tools Profile 被设置成了 messaging。有两种修复方法：

方法一：命令行修复（推荐）

```bash
# 查看当前 profile
openclaw config get tools

# 如果不是 full，切换成 full
openclaw config set tools.profile full

# 重启 gateway
openclaw gateway restart
```

方法二：Web 界面修复

1. 访问 http://localhost:18789（本地模式默认端口）
2. 点击左侧"配置"或"Agents"
3. 找到对应的 Agent，点击旁边的"tools"
4. 选择"Full"选项
5. 保存并重启

或者直接编辑配置文件，找到 tools 部分改成：

```json
"tools": {
  "profile": "full"
}
```

**Q: 提示"API key not found"怎么办？**

A: 编辑配置文件 `~/.openclaw/config.yaml`，在对应的提供商下添加 API 密钥。例如：

```yaml
llm:
  provider: claude
  api_key: sk-ant-xxxxx
```

**Q: Web 面板无法访问？**

A: 检查防火墙设置，确保端口 18789 未被占用。可以在配置文件中修改端口号。

**Q: 命令执行失败？**

A: 确认 OpenClaw 有足够的文件系统权限。某些操作可能需要明确授权。

---

**下一步**：[第二章 移动端接入](/cn/adopt/chapter2)
