---
title: "Copilot SDK Harness"
sidebarTitle: "Copilot Harness"
description: "OpenClaw 插件：通过 @openclaw/copilot 使用 GitHub Copilot SDK Harness 运行嵌入式 Agent 回合。"
---

# Copilot SDK Harness

`@openclaw/copilot` 是一个外部插件，让 OpenClaw 可以把嵌入式 Agent 回合交给 GitHub Copilot CLI / `@github/copilot-sdk` Harness 运行，而不是使用内置 PI Harness。

它适合已经有 GitHub Copilot 订阅，并且希望 Copilot CLI 接管底层 Agent Loop、原生压缩和 CLI 会话状态的场景。OpenClaw 仍然负责聊天通道、会话文件、模型选择、动态工具桥接、审批、媒体交付、可见 transcript mirror、`/btw` 旁路问题和 `openclaw doctor`。

---

## 使用前提

- 安装 `@openclaw/copilot` 插件。
- 有可驱动 Copilot CLI 的 GitHub Copilot 订阅，或为 headless / cron 运行提供 `gitHubToken` 环境变量或 auth profile。
- 有可写的 `copilotHome` 目录。默认会为每个 agent 隔离到 `~/.openclaw/agents/<agentId>/copilot`。
- 如果配置了 `plugins.allow`，要加入 manifest ID `copilot`，不是 npm 包名 `@openclaw/copilot`。

运行 `openclaw doctor` 可以检查插件 doctor contract。启用 agent runtime 前，先让 doctor 通过。

---

## 安装

```bash
openclaw plugins install @openclaw/copilot
```

这个插件不会随核心包强制安装，因为 `@github/copilot-sdk` 和平台相关 CLI 二进制体积较大。只有确实要把 agent runtime 选到 Copilot Harness 时再安装。

如果缺少 SDK，OpenClaw 会报 `COPILOT_SDK_MISSING`，按提示重新安装插件。

---

## 快速配置

把某个模型固定到 Copilot Harness：

```json5
{
  agents: {
    defaults: {
      model: "github-copilot/gpt-5.5",
      models: {
        "github-copilot/gpt-5.5": {
          agentRuntime: { id: "copilot" },
        },
      },
    },
  },
}
```

也可以在 Provider 层设置 `agentRuntime.id`，让该 Provider 下所有模型都走 Copilot Harness。

支持的 Provider ID：

- `github-copilot`

其他 Provider 会回退到 OpenClaw 默认 Harness。

---

## 认证优先级

Copilot Harness 会按顺序选择认证信号：

1. 显式 `useLoggedInUser: true`：使用 `copilotHome` 下 Copilot CLI 已登录用户。
2. 显式 `gitHubToken`：适合直接 CLI 调用或测试。
3. OpenClaw 解析出的 `resolvedApiKey` + `authProfileId`：生产主路径，适合 headless、cron、多 profile。
4. 环境变量回退：
   - `OPENCLAW_GITHUB_TOKEN`
   - `COPILOT_GITHUB_TOKEN`
   - `GH_TOKEN`
   - `GITHUB_TOKEN`
5. 默认使用已登录用户。

每个 agent 默认有独立的 `copilotHome`，避免不同 agent 共享 Copilot CLI Token、session 和配置。

---

## 配置面

Harness 主要读取 per-attempt input 和少量环境默认值：

- `copilotHome`：每个 agent 的 CLI 状态目录。
- `model`：模型字符串或 `{ provider, id, api? }`。
- `reasoningEffort`：`"low" | "medium" | "high" | "xhigh"`。
- `infiniteSessionConfig`：SDK `infiniteSessions` 覆盖项。
- `hooksConfig`：把 OpenClaw before/after-message-write hooks 桥接给 SDK Loop。
- `permissionPolicy`：SDK 内置工具权限处理；OpenClaw 桥接工具仍走 OpenClaw 自己的审批包装。
- `enableSessionTelemetry`：可选 OpenTelemetry 路由。

---

## 注意事项

- `/btw` 不是 Copilot Harness 原生能力，会回退到 OpenClaw 内置 PI 旁路问题路径。
- transcript mirror 是 best-effort，写入失败不会让本次 attempt 失败。
- 如果 `plugins.allow` 写成 `@openclaw/copilot`，插件可能被 allowlist 挡住；应写 manifest ID `copilot`。

---

## 故障排查

- `COPILOT_SDK_MISSING`：重新运行 `openclaw plugins install @openclaw/copilot`。
- 选择了 `github-copilot/*` 但没有走 Harness：确认模型或 Provider 配了 `agentRuntime: { id: "copilot" }`。
- Headless / cron 认证失败：优先配置 auth profile，或设置 `OPENCLAW_GITHUB_TOKEN`。
- 插件未加载：检查 `openclaw plugins list --verbose`、`plugins.allow` 和 `openclaw doctor`。
