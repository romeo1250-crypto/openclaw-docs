---
title: "Workboard CLI"
sidebarTitle: "Workboard"
description: "OpenClaw CLI：使用 openclaw workboard 管理 Workboard 卡片、查看状态并触发 dispatch。"
---

# Workboard CLI

`openclaw workboard` 是 [Workboard 插件](/tutorials/plugins/workboard) 的命令行入口。你可以在终端里列出卡片、创建卡片、查看卡片详情，并让 Gateway 把 ready 卡片 dispatch 给子 Agent 执行。

使用前先启用插件：

```bash
openclaw plugins enable workboard
openclaw gateway restart
```

---

## 命令总览

```bash
openclaw workboard list [--board <id>] [--status <status>] [--json]
openclaw workboard create <title...> [--notes <text>] [--status <status>] [--priority <priority>] [--agent <id>] [--board <id>] [--labels <items>] [--json]
openclaw workboard show <id> [--json]
openclaw workboard dispatch [--url <url>] [--token <token>] [--timeout <ms>] [--json]
```

这些命令读写的就是 Dashboard 和 Workboard Agent 工具使用的同一份插件 SQLite 状态。卡片 ID 可以用完整 ID，也可以用不歧义的前缀。

---

## 列出卡片

```bash
openclaw workboard list
openclaw workboard list --board default --status ready
openclaw workboard list --json
```

文本输出会显示 ID 前缀、状态、优先级、board、可选 agent 和标题：

```text
7f4a2c10  ready     high    default agent-a  Fix stale worker heartbeat
```

---

## 创建卡片

```bash
openclaw workboard create "Fix stale worker heartbeat" --priority high --labels bug,workboard
openclaw workboard create "Write Workboard docs" --status ready --agent docs-agent --board docs --notes "Cover CLI, slash command, dispatch, and SQLite state."
```

常用参数：

| 参数 | 用途 |
|------|------|
| `--notes <text>` | 初始备注 |
| `--status <status>` | 初始状态，默认 `todo` |
| `--priority <priority>` | 优先级，默认 `normal` |
| `--agent <id>` | 指派 agent 或 owner |
| `--board <id>` | 指定 board |
| `--labels <items>` | 逗号分隔标签 |
| `--json` | 输出机器可读 JSON |

---

## 查看详情

```bash
openclaw workboard show 7f4a2c10
openclaw workboard show 7f4a2c10 --json
```

JSON 输出会包含执行元数据、attempt、评论、链接、proof、artifact、worker log、诊断和自动化元数据等完整字段。

---

## Dispatch

```bash
openclaw workboard dispatch
openclaw workboard dispatch --json
openclaw workboard dispatch --url http://127.0.0.1:18789 --token "$OPENCLAW_GATEWAY_TOKEN"
```

`dispatch` 会优先调用运行中的 Gateway RPC `workboard.cards.dispatch`。这条路径使用和 Dashboard dispatch 相同的子 Agent 运行机制，把 ready 卡片变成带 task/run/session 关联的 worker run。

一次 dispatch 会保守地选择任务：

1. 把依赖已完成的子卡片提升到 `ready`。
2. 阻塞过期 claim 或超时 worker run。
3. 给 ready 卡片记录 dispatch 元数据。
4. 选择少量未 claim 的 ready 卡片。
5. 为 dispatcher 或指定 agent claim 卡片。
6. 启动带卡片上下文和 claim token 的子 Agent worker run。

默认一次最多启动 3 个 worker。已经归档、已有 active claim、或状态不是 `ready` 的卡片不会被选中。

如果没有显式 `--url` / `--token`，且本地 Gateway 不可用，CLI 会退回到 data-only dispatch：可以做依赖提升、清理 stale claim、阻塞超时 run，但不会启动 worker。

---

## 斜杠命令

支持命令的聊天通道也可以使用：

```text
/workboard list
/workboard show 7f4a2c10
/workboard create Fix stale worker heartbeat
/workboard dispatch
```

`/workboard create` 和 `/workboard dispatch` 会修改 board 状态，需要对应通道发送者有足够权限。

---

## 故障排查

### 看不到卡片

确认插件在同一个 profile 和 state root 下启用：

```bash
openclaw plugins inspect workboard --runtime --json
```

如果 Dashboard 有卡片但 CLI 没有，检查是否使用了不同的 `--dev` 或 `--profile`。

### Dispatch 只做 data-only

启动或重启 Gateway：

```bash
openclaw gateway restart
openclaw gateway status --deep
```

worker run 需要 live Gateway。

### Dispatch 没启动任何任务

确认至少有一个没有 active claim 的 `ready` 卡片：

```bash
openclaw workboard list --status ready
```

如果同一个 owner 已有 running 或 review 工作，也会被跳过。
