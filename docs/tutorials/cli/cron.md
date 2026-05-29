---
title: "openclaw cron"
sidebarTitle: "cron"
---

# `openclaw cron`

管理定时任务。

适合“每天早上发简报”“每周跑报告”这类固定时间任务。

```bash
openclaw cron list
openclaw cron show <jobId>
openclaw cron create "0 7 * * *" "Summarize overnight updates." --name "Morning brief"
openclaw cron run <jobId>
```

## 什么时候用

- 固定时间自动执行任务。
- 想查看定时任务是否还在。
- 某个定时任务失败，需要看最近运行记录。

## 新手提醒

定时任务依赖 Gateway 在线、时间设置正确、模型和通道可用。
如果没触发，先看：

```bash
openclaw cron list
openclaw tasks list
openclaw logs --follow
```

继续阅读：[Cron 定时任务](/tutorials/automation/cron-jobs)。

## 创建任务

`openclaw cron create` 是 `openclaw cron add` 的别名。
新任务建议用 `create`，把时间放第一位，把 Agent 要做的事放第二位：

```bash
openclaw cron create "0 7 * * *" \
  "Summarize overnight updates." \
  --name "Morning brief" \
  --agent ops
```

时间可以是：

- cron 表达式：`"0 9 * * 1"`
- 自然间隔：`"every 1h"`
- 简短间隔：`"20m"`
- ISO 时间：`"2026-02-01T16:00:00Z"`

## Webhook 输出

如果任务结果要给外部系统，不发到聊天，用 `--webhook`：

```bash
openclaw cron create "0 18 * * 1-5" \
  "Summarize today's deploys as JSON." \
  --name "Deploy digest" \
  --webhook "https://example.invalid/openclaw/cron"
```

也可以给已有任务设置：

```bash
openclaw cron edit <jobId> --webhook "https://example.invalid/openclaw/cron"
```

`--webhook` 不要和聊天投递参数混用，例如 `--announce`、`--channel`、`--to`、`--thread-id`、`--account`。

## 会话和投递

`--session` 常用值：

| 值 | 说明 |
|----|------|
| `main` | 复用主会话 |
| `isolated` | 每次任务新建会话 |
| `current` | 从当前聊天创建提醒时使用当前会话 |
| `session:<id>` | 指定固定会话 |

聊天投递常用：

```bash
openclaw cron create "0 7 * * *" \
  "Summarize overnight updates." \
  --name "Morning brief" \
  --session isolated \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## 常见误会

Cron 只负责“到点触发”。触发后能不能成功，还取决于模型、工具、通道和权限。
所以 cron 失败时，也要同时看 `tasks` 和日志。
