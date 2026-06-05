---
title: "Goal 会话目标"
sidebarTitle: "Goal"
description: "OpenClaw 工具系统：用 /goal 为当前会话设置一个持久目标，并跟踪完成、阻塞和 token 预算状态。"
---

# Goal 会话目标

Goal 是绑定在当前 OpenClaw 会话上的一个持久目标。它适合长任务：让用户、Agent 和 TUI 都能看到“这一轮到底要完成什么”。

Goal 不是后台任务、提醒、Cron 或 Standing Order。它不会自动调度工作，只是把当前会话的目标固定下来，防止长对话中跑偏。

---

## 快速使用

设置目标：

```text
/goal start 修复 PR 里的 CI 失败，验证后推送
```

查看当前目标：

```text
/goal
```

暂停：

```text
/goal pause 等待 CI 结果
```

恢复：

```text
/goal resume
```

完成：

```text
/goal complete 已修复并验证
```

清除：

```text
/goal clear
```

---

## 适合什么场景

- PR 收尾：修复、验证、复审、推送、更新 PR。
- Bug 定位：复现、找根因、修补、证明修复有效。
- 文档更新：读相关资料、写页面、补交叉链接、构建验证。
- 维护任务：检查当前状态、做有限改动、跑对应检查、报告结果。

如果工作需要脱离当前会话运行、定时重复、拆成多个托管子任务，请看 [Task Flow](/tutorials/automation/taskflow)、[任务](/tutorials/automation/tasks)、[Cron 定时任务](/tutorials/automation/cron-jobs) 或 [Standing Orders](/tutorials/automation/standing-orders)。

---

## 命令参考

- `/goal` 或 `/goal status`：查看当前目标。
- `/goal start <objective>`：创建新目标。
- `/goal set <objective>`、`/goal create <objective>`：`start` 的别名。
- `/goal pause [note]`：暂停目标。
- `/goal resume [note]`：恢复暂停、阻塞或预算受限的目标。
- `/goal complete [note]`：标记完成。
- `/goal done [note]`：`complete` 的别名。
- `/goal block [note]`：标记阻塞。
- `/goal blocked [note]`：`block` 的别名。
- `/goal clear`：从会话清除目标。

一个会话同一时间只能有一个 Goal。要换目标，先完成或清除旧目标。

---

## 状态

- `active`：正在追这个目标。
- `paused`：用户暂停了目标。
- `blocked`：目标被真实阻塞。
- `budget_limited`：达到 token 预算。
- `usage_limited`：触发使用限制。
- `complete`：目标完成。

`/new` 和 `/reset` 会清除当前会话 Goal，因为它们表示开始新的会话上下文。

---

## Token 预算

Goal 可以带 token 预算。达到预算后，状态会变成 `budget_limited`，目标不会被删除，但 Agent 不会继续主动追这个目标，直到你恢复或清除。

Token 预算只是会话目标护栏，不是账单上限。模型额度、费用统计和上下文窗口仍由 OpenClaw 正常的使用量控制负责。

---

## Agent 工具

OpenClaw 暴露了三个目标工具给 Agent Harness：

- `get_goal`：读取当前会话目标和预算状态。
- `create_goal`：只有用户、系统或开发者明确要求时才创建目标。
- `update_goal`：把目标标记为 `complete` 或 `blocked`。

模型不能悄悄暂停、恢复、清除或替换目标。这些动作需要通过 `/goal` 这类会话控制完成。
