---
title: "GMI Cloud"
sidebarTitle: "GMI Cloud"
description: "OpenClaw 模型接入：通过 GMI Cloud 的 OpenAI 兼容 API 使用多家托管模型路由。"
---

# GMI Cloud

GMI Cloud 是一个托管推理平台，提供 OpenAI 兼容 API。OpenClaw 已内置 `gmi` Provider，可以用普通模型认证流程保存密钥，并使用类似 `gmi/google/gemini-3.1-flash-lite` 的模型引用。

它适合想用一个 API Key 访问多家模型路线的人，比如 Google、Anthropic、OpenAI、DeepSeek、Moonshot、Z.AI 等 GMI 目录里的模型。

---

## 快速设置

在 GMI Cloud 创建 API Key 后运行：

```bash
openclaw onboard --auth-choice gmi-api-key
```

也可以用环境变量：

```bash
export GMI_API_KEY="<your-gmi-api-key>"
```

---

## 默认信息

| 项 | 值 |
|----|----|
| Provider | `gmi` |
| 别名 | `gmi-cloud`、`gmicloud` |
| Base URL | `https://api.gmi-serving.com/v1` |
| 环境变量 | `GMI_API_KEY` |
| 默认模型 | `gmi/google/gemini-3.1-flash-lite` |

---

## 什么时候选 GMI

- 你想用托管 OpenAI 兼容接口，而不是本地模型服务。
- 你想用一个账号试多家商业或开源模型路线。
- 你需要和 OpenRouter、DeepInfra、Together 或直连厂商 API 不同的 fallback 路径。
- GMI 比主 Provider 更早提供你需要的模型。

如果你需要厂商原生能力，优先选对应直连 Provider。如果你更重视数据本地化，优先看 [Ollama](/tutorials/providers/ollama)、vLLM 或 SGLang。

---

## 模型列表

上游内置目录包含这些常见模型引用：

- `gmi/zai-org/GLM-5.1-FP8`
- `gmi/deepseek-ai/DeepSeek-V3.2`
- `gmi/moonshotai/Kimi-K2.5`
- `gmi/google/gemini-3.1-flash-lite`
- `gmi/anthropic/claude-sonnet-4.6`
- `gmi/openai/gpt-5.4`

目录只是种子，不保证每个账号任何时候都能调用。以 CLI 查询结果为准：

```bash
openclaw models list --provider gmi
```

---

## 故障排查

- `401` 或 `403`：确认 `GMI_API_KEY` 对运行 OpenClaw 的进程可见，或重新运行 onboarding。
- Unknown model：用 `openclaw models list --provider gmi` 返回的完整 `gmi/<route-id>`。
- 间歇性 Provider 报错：换一条 GMI route，或把 GMI 配成 fallback，而不是唯一主模型。
