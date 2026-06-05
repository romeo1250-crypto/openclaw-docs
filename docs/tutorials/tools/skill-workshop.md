---
title: "Skill Workshop"
sidebarTitle: "Skill Workshop"
description: "OpenClaw 工具系统：通过受控提案流程创建和更新 workspace skills。"
---

# Skill Workshop

Skill Workshop 是 OpenClaw 创建和更新 workspace skills 的受控流程。

它的核心原则是：**先生成提案，再由人或受信策略应用**。Agent 不会直接把生成内容写成生效的 `SKILL.md`；它会先创建 `PROPOSAL.md`，经过扫描和审批后才变成真实 skill。

---

## 工作方式

- 先提案：生成内容存为 `PROPOSAL.md`，不是 `SKILL.md`。
- 只有 apply 会写入生效 skill。
- 只写 workspace skills，不修改 bundled、plugin、ClawHub、extra-root、managed 或系统 skill。
- create 不覆盖已有 skill。
- update 绑定当前目标 hash；目标变了，提案会变 stale。
- apply 前重新扫描。
- 写入前保存 rollback 元数据，方便恢复。

生命周期：

```text
create/update -> pending
revise        -> pending
apply         -> applied
reject        -> rejected
quarantine    -> quarantined
target change -> stale
```

---

## 聊天里怎么用

你可以直接让 Agent 创建或更新 skill：

```text
创建一个 morning-catchup skill，用来处理我周一早上的 inbox 例行流程。
```

更新已有 workspace skill：

```text
更新 trip-planning，让它订票前也检查座位图。
```

然后继续审阅和应用：

```text
显示 morning-catchup 提案。
修改它，让它也标记 urgent 邮件。
应用 morning-catchup 提案。
```

默认情况下，Agent 发起 `apply`、`reject`、`quarantine` 会触发审批提示。可信环境可以把 `skills.workshop.approvalPolicy` 设成 `"auto"`。

---

## CLI

创建新 skill 提案：

```bash
openclaw skills workshop propose-create \
  --name morning-catchup \
  --description "Daily inbox catch-up: triage, archive, surface, draft, plan" \
  --proposal ./PROPOSAL.md
```

更新已有 workspace skill：

```bash
openclaw skills workshop propose-update trip-planning --proposal ./PROPOSAL.md
```

查看、修订、收尾：

```bash
openclaw skills workshop list
openclaw skills workshop inspect <proposal-id>
openclaw skills workshop revise <proposal-id> --proposal ./PROPOSAL.md
openclaw skills workshop apply <proposal-id>
openclaw skills workshop reject <proposal-id> --reason "Duplicate"
openclaw skills workshop quarantine <proposal-id> --reason "Needs security review"
```

---

## 带支持文件的提案

如果 skill 需要脚本、模板或示例文件，用 `--proposal-dir`：

```bash
openclaw skills workshop propose-create \
  --name weekly-update \
  --description "Friday wrap-up: stats, highlights, next week's top three" \
  --proposal-dir ./weekly-update-proposal
```

目录必须包含 `PROPOSAL.md`。支持文件只能放在：

- `assets/`
- `examples/`
- `references/`
- `scripts/`
- `templates/`

绝对路径、隐藏路径段、路径穿越、可执行文件、非 UTF-8 文本和超出标准目录的文件都会被拒绝。

---

## 配置

```json5
{
  skills: {
    workshop: {
      autonomous: {
        enabled: false,
      },
      approvalPolicy: "pending",
      maxPending: 50,
      maxSkillBytes: 40000,
    },
  },
}
```

- `autonomous.enabled`：允许 OpenClaw 从长期对话信号自动创建 pending 提案。默认 `false`。
- `approvalPolicy: "pending"`：Agent 发起 apply/reject/quarantine 前需要审批。
- `approvalPolicy: "auto"`：跳过该审批提示。
- `maxPending`：限制每个 workspace 的 pending 和 quarantined 提案数量。
- `maxSkillBytes`：限制提案正文大小，默认 40000。

---

## 故障排查

| 问题 | 处理 |
|------|------|
| `Skill proposal description is too large` | 缩短 description 到 160 字节以内 |
| `Skill proposal content is too large` | 缩短正文或调大 `skills.workshop.maxSkillBytes` |
| `Target skill changed after proposal creation` | 基于当前目标重新修订或新建提案 |
| `Proposal scan failed` | 查看扫描结果，修订或隔离提案 |
| 提案列表里看不到 | 检查当前 `--agent` workspace 和 `OPENCLAW_STATE_DIR` |
