---
title: "NovitaAI"
sidebarTitle: "NovitaAI"
description: "OpenClaw 模型接入：通过 NovitaAI 的 OpenAI 兼容 API 使用托管开源和第三方模型。"
---

# NovitaAI

NovitaAI 是一个托管 AI 基础设施 Provider，提供 OpenAI 兼容模型 API。OpenClaw 已内置 `novita` Provider，模型引用形如 `novita/deepseek/deepseek-v3-0324`。

它适合想使用 DeepSeek、Kimi、MiniMax、GLM、Qwen 等托管模型路线，但不想自己维护推理服务器的场景。

---

## 快速设置

在 [NovitaAI Key Management](https://novita.ai/settings/key-management) 创建 API Key 后运行：

```bash
openclaw onboard --auth-choice novita-api-key
```

也可以用环境变量：

```bash
export NOVITA_API_KEY="<your-novita-api-key>"
```

---

## 默认信息

| 项 | 值 |
|----|----|
| Provider | `novita` |
| 别名 | `novita-ai`、`novitaai` |
| Base URL | `https://api.novita.ai/openai/v1` |
| 环境变量 | `NOVITA_API_KEY` |
| 默认模型 | `novita/deepseek/deepseek-v3-0324` |

---

## 什么时候选 Novita

- 你想用托管开源模型，不想部署 vLLM、SGLang、LM Studio 或 Ollama。
- 你想通过一个账号访问 DeepSeek、Kimi、MiniMax、GLM 或 Qwen 系路线。
- 你需要 OpenRouter、GMI、DeepInfra 或直连厂商 API 之外的备用路径。

需要厂商原生参数或支持合同，选直连厂商 Provider；需要模型跑在自己机器或内网，选本地/自托管 Provider。

---

## 模型列表

常见种子模型包括：

- `novita/moonshotai/kimi-k2.5`
- `novita/minimax/minimax-m2.7`
- `novita/zai-org/glm-5`
- `novita/deepseek/deepseek-v3-0324`
- `novita/deepseek/deepseek-r1-0528`
- `novita/qwen/qwen3-235b-a22b-fp8`

实际可用模型以你的账号和 Novita 当前目录为准：

```bash
openclaw models list --provider novita
```

---

## 故障排查

- `401` 或 `403`：确认 Novita Key 仍有效，必要时重新运行 `openclaw onboard --auth-choice novita-api-key`。
- Unknown model：复制 `openclaw models list --provider novita` 返回的完整模型引用。
- 路由慢或失败：换模型路线，或只把 Novita 作为 fallback Provider。
