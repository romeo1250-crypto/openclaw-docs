---
title: "Cron 定时任务"
sidebarTitle: "Cron 定时任务"
description: "OpenClaw 自动化：Cron 定时任务。用 Gateway 内置调度器在指定时间唤醒 Agent，也可以把结果发到聊天或 Webhook。"
---

# Cron 定时任务

Cron 是 OpenClaw Gateway 内置的定时器。
它负责“到点叫醒 Agent”，适合日报、巡检、提醒、定期汇总。

第一次使用记住这个格式：

```bash
openclaw cron create "<时间>" "<要 Agent 做什么>" --name "<任务名>"
```

---

## 先做一个最小任务

创建一个一次性提醒：

```bash
openclaw cron create "2026-02-01T16:00:00Z" \
  "Reminder: check the cron docs draft" \
  --name "Reminder" \
  --session main \
  --wake now
```

看任务是否创建成功：

```bash
openclaw cron list
```

手动跑一次，不用等到时间：

```bash
openclaw cron run <jobId> --wait
```

`<jobId>` 从 `openclaw cron list` 里复制。

---

## 每天、每周、每隔多久

`cron create` 的第一个位置参数是时间。常用写法：

| 写法 | 意思 |
|------|------|
| `"0 7 * * *"` | 每天 7 点 |
| `"0 9 * * 1-5"` | 工作日 9 点 |
| `"every 1h"` | 每小时 |
| `"20m"` | 每 20 分钟 |
| `"2026-02-01T16:00:00Z"` | 指定 UTC 时间执行一次 |

第二个位置参数是给 Agent 的任务说明：

```bash
openclaw cron create "0 7 * * *" \
  "Summarize overnight updates." \
  --name "Morning brief" \
  --tz "America/Los_Angeles" \
  --session isolated
```

::: tip 时区别靠猜
服务器不一定在你的城市。需要固定本地时间时，显式写 `--tz`。
中国大陆常用 `--tz "Asia/Shanghai"`。
:::

---

## 会话怎么选

`--session` 决定任务醒来时用哪个上下文。

| 值 | 适合场景 |
|----|----------|
| `main` | 复用主会话，适合持续跟踪同一件事 |
| `isolated` | 每次新会话，适合日报、巡检、批处理 |
| `current` | 从当前聊天创建提醒时使用当前会话 |
| `session:<id>` | 指定某个固定会话 |

不确定时，用 `isolated`，更干净。

---

## 把结果发到聊天

让任务结束后把最终结果发到 Slack：

```bash
openclaw cron create "0 7 * * *" \
  "Summarize overnight updates." \
  --name "Morning brief" \
  --session isolated \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

常见投递参数：

| 参数 | 作用 |
|------|------|
| `--announce` | 让 runner 把最终回复发出去 |
| `--channel <name>` | 投递通道，例如 `slack`、`telegram` |
| `--to <target>` | 投递目标，例如 Slack channel ID |
| `--no-deliver` | 不做 runner fallback 投递 |

如果任务本身能用 `message` 工具发消息，`--announce` 是兜底投递，不是唯一投递路径。

---

## 把结果发到 Webhook

如果你要把 cron 结果交给外部系统，用 `--webhook`：

```bash
openclaw cron create "0 18 * * 1-5" \
  "Summarize today's deploys as JSON." \
  --name "Deploy digest" \
  --webhook "https://example.invalid/openclaw/cron"
```

`--webhook` 会在任务完成后 POST 运行结果。

不要把 `--webhook` 和聊天投递参数混用：

- 不要同时用 `--announce`
- 不要同时用 `--channel`
- 不要同时用 `--to`
- 不要同时用 `--thread-id`
- 不要同时用 `--account`

Webhook 是给系统收结果；聊天投递是给人看结果。两条路选一条。

---

## 多 Agent 怎么指定

如果你有多个 Agent，用 `--agent`：

```bash
openclaw cron create "0 6 * * *" \
  "Check ops queue" \
  --name "Ops sweep" \
  --session isolated \
  --agent ops
```

清掉任务上的 Agent 绑定：

```bash
openclaw cron edit <jobId> --clear-agent
```

---

## 常用命令

```bash
openclaw cron list
openclaw cron show <jobId>
openclaw cron runs --id <jobId>
openclaw cron run <jobId> --wait
openclaw cron edit <jobId> --name "New name"
openclaw cron remove <jobId>
```

`openclaw cron create` 是 `openclaw cron add` 的别名。
新任务建议用 `create`，因为它更像一句自然语言命令：先写时间，再写任务。

---

## 没按时执行怎么办

按顺序查：

```bash
openclaw gateway status
openclaw cron list
openclaw cron runs --id <jobId>
openclaw logs --follow
```

常见原因：

- Gateway 没在运行。
- 时区不是你以为的时区。
- 模型认证失败。
- 目标聊天通道没有权限。
- Webhook 地址不可达或返回错误。

---

## 继续阅读

- [openclaw cron](/tutorials/cli/cron)
- [Cron vs Heartbeat](/tutorials/automation/cron-vs-heartbeat)
- [自动化故障排查](/tutorials/automation/troubleshooting)
