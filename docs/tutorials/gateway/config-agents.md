---
title: "Agent 配置"
sidebarTitle: "Agent 配置"
---

# Agent 配置：决定 AI 助手怎么工作

`agents.*` 相关配置决定 Agent 的默认工作区、模型、技能、上下文文件、会话和消息行为。

你可以把它想成给助手准备工位：桌子在哪里、能看哪些说明书、能用哪些技能、一次能带多少资料。

---

## 最常见的配置

| 配置 | 作用 |
|------|------|
| `agents.defaults.workspace` | 默认工作区 |
| `agents.defaults.repoRoot` | 项目根目录提示 |
| `agents.defaults.skills` | 默认允许的技能 |
| `agents.defaults.contextInjection` | 是否注入工作区说明文件 |
| `agents.list` | 配多个 Agent |
| `multiAgent` | 多 Agent 路由 |
| `session` | 会话生命周期和绑定 |
| `messages` | 消息投递和格式 |
| `talk` | 语音/对话模式相关配置 |

---

## 工作区

默认工作区通常是：

```text
~/.openclaw/workspace
```

如果环境变量里设置了 `OPENCLAW_WORKSPACE_DIR`，默认工作区会先用它。
只有你在 `agents.defaults.workspace` 里显式写了路径，才会覆盖这个环境变量。

示意配置：

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace"
    }
  }
}
```

工作区里常见 `AGENTS.md`、`SOUL.md`、`USER.md` 等文件。它们会影响 Agent 的规则、风格和记忆。

---

## 技能 allowlist

如果你想限制 Agent 能用哪些技能，可以配置：

```json5
{
  agents: {
    defaults: {
      skills: ["github", "weather"]
    },
    list: [
      { id: "writer" },
      { id: "locked-down", skills: [] }
    ]
  }
}
```

意思是：

- `writer` 继承默认技能。
- `locked-down` 不允许任何技能。
- 单个 Agent 设置了自己的 skills，就以它自己的为准。

---

## 上下文注入

`contextInjection` 决定工作区说明文件什么时候进入系统提示词。

常见值：

| 值 | 说明 |
|----|------|
| `always` | 每次都注入 |
| `continuation-skip` | 安全续写时跳过，省上下文 |
| `never` | 完全不注入，适合自定义 runtime |

新手保持默认即可。

单个 Agent 也可以覆盖上下文注入和 bootstrap 大小限制：

```json5
{
  agents: {
    defaults: {
      contextInjection: "continuation-skip",
      bootstrapMaxChars: 12000,
      bootstrapTotalMaxChars: 60000,
      bootstrapPromptTruncationWarning: "always"
    },
    list: [
      {
        id: "docs",
        contextInjection: "always",
        bootstrapMaxChars: 50000,
        bootstrapTotalMaxChars: 300000
      }
    ]
  }
}
```

这里最容易误会的是 `bootstrapPromptTruncationWarning`。
新版默认是 `"always"`：只要工作区说明文件被截断，就每次在系统提示词里放一个简短提醒。

::: tip 给奶奶看的解释
如果 AGENTS.md 太长，OpenClaw 不会假装自己全看完了。
它会告诉 Agent：“这份说明被截断了，必要时自己去读原文件。”
这比静悄悄漏掉后半截规则安全。
:::

工具结果也有自己的上限。`toolResultMaxChars` 如果不写，OpenClaw 会按模型上下文自动计算：

- 100K token 以下：约 16000 字符
- 100K+ token：约 32000 字符
- 200K+ token：约 64000 字符

普通用户不要急着手动调这个值。需要确认实际生效值时运行：

```bash
openclaw doctor --deep
```

---

## 图片质量

`agents.defaults.imageQuality` 控制文件图片、URL 图片、媒体引用进入模型前的压缩/细节策略。

```json5
{
  agents: {
    defaults: {
      imageQuality: "auto"
    }
  }
}
```

可选值：

| 值 | 适合场景 |
|----|----------|
| `auto` | 默认，让 OpenClaw 按模型和图片数量自动判断 |
| `efficient` | 省 token、低延迟 |
| `balanced` | 平衡清晰度和成本 |
| `high` | 截图、文档、图表需要更多细节 |

新手保持 `auto`。

---

## 配多个 Agent

你可以让不同 Agent 有不同工作区、工具和风格。例如：

- `main` 负责日常聊天。
- `docs` 负责写文档。
- `ops` 负责运维检查。

多 Agent 很强，但配置也更复杂。基础通道和模型没跑稳前，不建议一开始就拆很多 Agent。

---

## Runtime 策略放在哪里

新版规则很重要：

```text
Runtime 策略属于 provider 或 model，不属于整个 agents.defaults。
```

推荐写法：

```json5
{
  models: {
    providers: {
      openai: {
        agentRuntime: { id: "codex" }
      }
    }
  },
  agents: {
    defaults: {
      model: "openai/gpt-5.5",
      models: {
        "vllm/*": {
          agentRuntime: { id: "openclaw" }
        }
      }
    }
  }
}
```

旧写法不要再依赖：

```json5
{
  agents: {
    defaults: {
      agentRuntime: { id: "codex" }
    }
  }
}
```

`agents.defaults.agentRuntime`、`agents.list[].agentRuntime`、会话里的 runtime pin、
`OPENCLAW_AGENT_RUNTIME` 都属于旧路线。新版 runtime 选择会忽略这些 whole-agent key。

如果你以前写过这些字段，运行：

```bash
openclaw doctor --fix
```

它会尽量清掉旧值，避免你以为配置生效了，实际上没有生效。

常见 runtime id：

| id | 意思 |
|----|------|
| `auto` | 让已注册插件自己认领能处理的模型，否则回到 OpenClaw |
| `codex` | 使用 Codex app-server harness |
| `openclaw` | 使用 OpenClaw 内置运行时 |

`pi` 只是旧版兼容别名。新配置请写 `openclaw`。

OpenAI Agent 模型现在默认会选择 Codex harness，所以普通 `openai/gpt-5.5` 配置不需要手动写 runtime。

---

## provider 通配模型和本地服务

`agents.defaults.models` 可以写 provider 通配项：

```json5
{
  agents: {
    defaults: {
      models: {
        "vllm/*": {},
        "openai/gpt-5.5": { alias: "gpt" }
      }
    }
  }
}
```

这表示“允许 vLLM provider 动态发现出来的模型”，不用一个个列。

如果你跑本地模型服务，还可以在 `models.providers.<provider>.localService` 配启动命令。
当选中的模型属于这个 provider，OpenClaw 会先探测健康地址；服务没起来才启动命令。

完整说明看 [Local model services](/tutorials/gateway/local-model-services)。

---

## Talk 配置

`talk` 控制语音对话。
它分两层：

- `talk.provider` + `talk.providers.<provider>`：传统语音播放，也就是 `talk.speak` 和 STT/TTS 模式里的 TTS。
- `talk.realtime.*`：浏览器或服务端实时语音会话。

示例：

```json5
{
  talk: {
    provider: "elevenlabs",
    speechLocale: "zh-CN",
    silenceTimeoutMs: 1500,
    interruptOnSpeech: true,
    realtime: {
      provider: "openai",
      providers: {
        openai: {
          model: "gpt-realtime",
          voice: "alloy"
        }
      },
      mode: "realtime",
      transport: "webrtc",
      brain: "agent-consult"
    }
  }
}
```

如果配置里还写着旧字段：

- `talk.mode`
- `talk.transport`
- `talk.brain`
- `talk.model`
- `talk.voice`

运行：

```bash
openclaw doctor --fix
```

新版 doctor 会把这些旧的顶层 realtime selector 迁到 `talk.realtime`。

---

## 继续阅读

- [Agent 是什么](/tutorials/concepts/agent)
- [多智能体路由](/tutorials/concepts/multi-agent)
- [SOUL.md](/tutorials/concepts/soul)
