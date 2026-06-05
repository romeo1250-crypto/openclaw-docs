---
title: "Qwen OAuth / Portal"
sidebarTitle: "Qwen OAuth"
description: "OpenClaw 模型接入：使用 qwen-oauth Provider 连接 Qwen Portal 端点，主要面向旧版 Portal/OAuth 凭证迁移。"
---

# Qwen OAuth / Portal

`qwen-oauth` 是 Qwen Portal Provider ID，目标端点是：

```text
https://portal.qwen.ai/v1
```

它主要用于已有 Qwen Portal Token、旧版 Qwen OAuth 或 Qwen CLI 工作流迁移。新用户通常应该优先使用 [通义千问 Qwen](/tutorials/providers/qwen)，也就是标准 ModelStudio / DashScope 路线。

---

## 快速设置

通过 onboarding 提供 Portal Token：

```bash
openclaw onboard --auth-choice qwen-oauth
```

也可以用环境变量：

```bash
export QWEN_API_KEY="<your-qwen-portal-token>"
```

---

## 默认信息

| 项 | 值 |
|----|----|
| Provider | `qwen-oauth` |
| 别名 | `qwen-portal`、`qwen-cli` |
| Base URL | `https://portal.qwen.ai/v1` |
| 环境变量 | `QWEN_API_KEY` |
| API 风格 | OpenAI 兼容 |
| 默认模型 | `qwen-oauth/qwen3.5-plus` |

---

## 和 `qwen` 的区别

| Provider | 端点家族 | 适合场景 |
|----------|----------|----------|
| `qwen` | Qwen Cloud / Alibaba DashScope / Coding Plan | 新 API Key、标准按量付费、多模态 DashScope 能力 |
| `qwen-oauth` | Qwen Portal `portal.qwen.ai/v1` | 已有 Portal Token、旧版 OAuth / CLI 迁移 |

两者虽然都走 OpenAI 兼容请求形状，但认证面不同。`qwen-oauth` 的 Token 不应该当成 DashScope 或 ModelStudio Key 使用。

---

## 迁移建议

如果旧 Portal OAuth Profile 不再可刷新，优先重新认证，或者迁移到标准 Qwen Provider：

```bash
openclaw onboard --auth-choice qwen-standard-api-key
openclaw models set qwen/qwen3-coder-plus
```

标准全球 ModelStudio 端点是：

```text
https://dashscope-intl.aliyuncs.com/compatible-mode/v1
```

---

## 故障排查

- Portal OAuth 刷新失败：旧 OAuth Profile 可能已经不可刷新，重新用当前 Token 做 onboarding。
- 端点错误：Portal Token 使用 `qwen-oauth/` 模型引用；标准 Qwen Key 使用 `qwen/`。
- `QWEN_API_KEY` 混淆：两个 Qwen 页面都会提到这个环境变量。若同一台机器同时保留两套凭证，更推荐用 onboarding 按 Provider ID 存储。
