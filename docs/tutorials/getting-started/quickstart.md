---
title: "快速开始"
sidebarTitle: "快速开始"
description: "OpenClaw 快速入门：快速开始（3 分钟）。用安装脚本、onboard 向导和 Web 控制 UI 跑通第一个 OpenClaw。"
---

# 快速开始（3 分钟）

> 这是最精简的安装路径，只有 3 个步骤。如果需要详细说明，请看[完整入门指南](/tutorials/getting-started/getting-started)。

---

## 第一步：安装 OpenClaw

**macOS / Linux（在终端里运行）：**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows（在 PowerShell 里运行）：**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

::: info 前提条件
推荐使用 Node.js 24；如果你已经是 Node.js 22.19+，也可以继续用。
安装脚本通常会帮你处理 Node.js。想手动检查，可以运行 `node -v`，详见[安装 Node.js](/tutorials/installation/node)。
:::

---

## 第二步：运行配置向导

```bash
openclaw onboard --install-daemon
```

跟着向导走，它会问你：
1. 选择 AI 服务商（OpenAI、Anthropic、本地模型等都可以）
2. 填入 API 密钥
3. 选择聊天软件（推荐先选 **Telegram**）

整个过程 2~3 分钟，向导会帮你完成所有配置。

---

## 第三步：打开控制 UI，开始对话

向导完成后，Gateway 通常已经作为后台服务运行。打开浏览器控制台：

```bash
openclaw dashboard
```

也可以直接访问 [http://127.0.0.1:18789](http://127.0.0.1:18789)。

在控制 UI 的输入框里发消息，AI 能回复，就说明第一步跑通了。

---

## 下一步

- **在 Telegram / WhatsApp 里聊天** → [频道接入教程](/tutorials/channels/)
- **了解详细设置** → [完整入门指南](/tutorials/getting-started/getting-started)
- **遇到问题？** → [故障排查](/tutorials/gateway/troubleshooting)
