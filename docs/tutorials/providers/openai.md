---
title: "OpenAI"
sidebarTitle: "OpenAI"
description: "OpenClaw 模型接入：OpenAI。讲清 OpenAI API Key、Codex 订阅登录、openai/* 模型路由和实时语音账单的区别。"
---

# OpenAI

OpenAI 这一页最容易让人混乱，因为名字很像：

- `openai`：模型提供商前缀，也就是模型写成 `openai/gpt-5.5`。
- `openai-codex`：旧配置和 Codex OAuth 登录资料的名字。
- `codex`：OpenClaw 里负责原生 Codex app-server 运行时的插件/运行时。

先记住一句话：

```text
新配置里，OpenAI Agent 模型优先写 openai/gpt-5.5。
就算你用的是 ChatGPT/Codex 订阅登录，模型名也不要写成 openai-codex/gpt-*。
```

旧的 `openai-codex/*` 不是推荐的新模型路线。升级后请运行：

```bash
openclaw doctor --fix
openclaw config validate
```

Doctor 会尽量把旧模型引用修成 `openai/*`，并保留能用的 Codex 登录资料。

---

## 先选你是哪一种用法

| 你想做什么 | 推荐模型写法 | 登录/付费方式 |
|------------|--------------|---------------|
| 用 ChatGPT/Codex 订阅跑 Agent | `openai/gpt-5.5` | `openai-codex` OAuth 登录 |
| 用 OpenAI Platform API Key 跑 Agent | `openai/gpt-5.5` | 配好 OpenAI API Key，并按需要设置 auth order |
| 用 OpenAI 图片、语音、Embedding | `openai/gpt-image-2` 等 | OpenAI API Key 或支持的 OAuth 路线 |
| 试 ChatGPT Instant 最新别名 | `openai/chat-latest` | 只建议 API Key 实验，不建议生产默认 |

::: tip 给奶奶看的解释
模型名像“要点哪道菜”，登录方式像“用哪张卡付钱”，运行时像“哪个厨房来做菜”。
不要把三件事写成一个名字。现在点 OpenAI Agent 这道菜，菜名通常就是 `openai/gpt-5.5`。
:::

---

## 方式 A：ChatGPT/Codex 订阅登录

适合：你有 ChatGPT/Codex 订阅，想让 OpenClaw 走原生 Codex 运行时。

### 第一步：登录 Codex OAuth

```bash
openclaw onboard --auth-choice openai-codex
```

或者只登录模型认证：

```bash
openclaw models auth login --provider openai-codex
```

如果服务器没有方便打开浏览器，用设备码：

```bash
openclaw models auth login --provider openai-codex --device-code
```

### 第二步：模型仍然写 `openai/*`

```bash
openclaw config set agents.defaults.model.primary openai/gpt-5.5
```

配置文件里等价写法：

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai/gpt-5.5" }
    }
  }
}
```

新版本里，OpenAI Agent turn 默认会选择原生 Codex app-server 运行时。
普通用户不需要再写 `agents.defaults.agentRuntime`。

### 第三步：检查是否真的登录成功

```bash
openclaw models auth list --provider openai-codex
openclaw models status
```

Gateway 已经运行后，也可以在聊天里发：

```text
/codex status
```

---

## 方式 B：OpenAI API Key

适合：你想走 OpenAI Platform 按量计费，或者需要图片、Embedding、Realtime 等平台能力。

### CLI 设置

```bash
openclaw onboard --auth-choice openai-api-key
```

也可以非交互式传入：

```bash
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### 配置示例

```json5
{
  env: {
    OPENAI_API_KEY: "sk-..."
  },
  agents: {
    defaults: {
      model: { primary: "openai/gpt-5.5" }
    }
  }
}
```

如果你同时有 Codex 订阅和 API Key，并希望订阅优先、API Key 备用，可以把认证顺序放在 `auth.order.openai` 下。
这样模型名仍然保持 `openai/gpt-5.5`：

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai/gpt-5.5" }
    }
  },
  auth: {
    order: {
      openai: [
        "openai-codex:user@example.com",
        "openai:api-key-backup"
      ]
    }
  }
}
```

---

## `openai-codex/*` 旧写法怎么办？

如果你的配置里还有：

```text
openai-codex/gpt-*
codex-cli/gpt-*
```

先不要手动乱改一堆文件。运行：

```bash
openclaw doctor --fix
openclaw config validate
```

新版 Doctor 会尽量做这些事：

1. 把旧模型名修成 `openai/*`。
2. 保留已有的 `openai-codex` OAuth 登录资料。
3. 清理旧的 runtime pin，避免会话继续走过期路线。

::: warning 不要把 `openai-codex` 当成新模型前缀
`openai-codex` 现在更多是旧配置和登录资料命名。新 Agent 模型引用优先使用 `openai/gpt-5.5`。
只有账号目录明确暴露的特殊 Codex 模型，才可能需要保留特殊旧路线；普通用户不要从这里开始。
:::

---

## Realtime 语音账单要特别注意

OpenAI Realtime 语音不是消耗 ChatGPT/Codex 订阅额度。
它走的是 **OpenAI Platform Realtime API**，需要 OpenAI Platform 组织有可用额度或账单。

所以可能出现这种情况：

```text
文字聊天能用，因为 Codex OAuth 正常。
Realtime 语音失败，因为 OpenAI Platform 没有充值或没有账单。
```

如果看到 `insufficient_quota` 或 “You exceeded your current quota”，请到
[OpenAI Platform Billing](https://platform.openai.com/account/billing) 检查对应组织的额度。

---

## `openai/chat-latest` 是什么？

`openai/chat-latest` 是 OpenAI API 的移动别名，适合实验 ChatGPT 当前 Instant 模型。

不建议把它当生产默认模型，因为它会随 OpenAI 调整而变化。
生产配置优先使用明确模型，例如：

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai/gpt-5.5" }
    }
  }
}
```

---

## 继续阅读

- [模型提供商](/tutorials/concepts/model-providers)
- [模型选择](/tutorials/concepts/models)
- [OAuth 认证](/tutorials/concepts/oauth)
- [Agent 运行时](/tutorials/concepts/agent-runtimes)
