---
title: "ClawHub CLI"
sidebarTitle: "ClawHub"
description: "OpenClaw 工具系统：通过 OpenClaw CLI 搜索、安装、更新 ClawHub 技能和插件；通过独立 clawhub CLI 完成发布者流程。"
---

# ClawHub CLI

ClawHub 是 OpenClaw 的技能和插件分发中心。现在有两类命令入口：

- `openclaw skills` 和 `openclaw plugins`：在本地 OpenClaw Agent 或 Gateway 里搜索、安装、更新和验证 ClawHub 包。
- 独立的 `clawhub` CLI：给发布者使用，处理登录、发布、转移和同步等维护流程。

如果你只是普通用户，优先使用 `openclaw skills` 和 `openclaw plugins`。只有要发布技能或插件时，才需要安装独立 `clawhub` CLI。

---

## 搜索和安装

安装技能：

```bash
openclaw skills search "calendar"
openclaw skills install <slug>
openclaw skills update <slug>
openclaw skills verify <slug>
```

技能默认安装到当前 workspace 的 `skills/` 目录。需要装到共享 managed skills 目录时，加 `--global`：

```bash
openclaw skills install <slug> --global
```

安装插件：

```bash
openclaw plugins search "calendar"
openclaw plugins install clawhub:<package>
openclaw plugins update <id-or-npm-spec>
```

`clawhub:` 前缀表示通过 ClawHub 解析，而不是走 npm 或其他安装源。

---

## 发布和维护

发布者工作流使用独立 `clawhub` CLI：

```bash
npm i -g clawhub
clawhub login
```

发布插件包：

```bash
clawhub package publish your-org/your-plugin --dry-run
clawhub package publish your-org/your-plugin
clawhub package publish your-org/your-plugin@v1.0.0
```

发布技能目录：

```bash
clawhub skill publish ./skills/review-helper
clawhub skill publish ./skills/review-helper --version 1.0.0
```

维护本地扫描状态或包所有权：

```bash
clawhub sync --all
clawhub package transfer @old-owner/package --to new-owner
```

---

## 常见区别

| 你要做什么 | 用哪个命令 |
|------------|------------|
| 搜索并安装技能 | `openclaw skills search/install` |
| 更新或验证本地技能 | `openclaw skills update/verify` |
| 搜索并安装插件 | `openclaw plugins search/install` |
| 发布技能或插件 | `clawhub skill publish` / `clawhub package publish` |
| 转移包所有权 | `clawhub package transfer` |

---

## 相关页面

- [技能系统](/tutorials/tools/skills)
- [创建技能](/tutorials/tools/creating-skills)
- [管理插件](/tutorials/plugins/manage-plugins)
- [构建插件](/tutorials/plugins/building-plugins)
- [ClawHub 发布](/tutorials/tools/clawhub-publishing)
