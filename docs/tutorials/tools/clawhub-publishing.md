---
title: "ClawHub 发布"
sidebarTitle: "ClawHub 发布"
description: "OpenClaw 工具系统：ClawHub 发布流程、owner scope、技能和插件包命名规则。"
---

# ClawHub 发布

ClawHub 发布是 owner-scoped 的：每次发布都要选择一个发布者 owner，服务端会判断当前登录用户是否有权限代表这个 owner 发布。

这页主要给技能和插件作者看。普通用户只安装技能或插件时，优先看 [ClawHub CLI](/tutorials/tools/clawhub)。

---

## Owner

Owner 是 ClawHub 发布者标识，例如：

```text
@alice
@openclaw
```

个人 owner 属于单个用户；组织 owner 可以有多个成员。发布时，你要么使用自己的个人 owner，要么选择一个你拥有发布权限的组织 owner。

---

## 发布技能

技能从一个 skill 文件夹发布。公开页面形如：

```text
https://clawhub.ai/<owner>/<slug>
```

示例：

```text
https://clawhub.ai/alice/review-helper
```

发布请求会包含 owner、slug、version、changelog 和文件列表。ClawHub 服务端会先验证当前用户是否能代表该 owner 发布，再创建 release。

常用命令：

```bash
clawhub skill publish ./skills/review-helper
clawhub skill publish ./skills/review-helper --version 1.0.0
```

---

## 发布插件

插件使用 npm 风格包名。带 scope 的包名第一段要和发布 owner 对应：

```text
@owner/package-name
```

例如包名是 `@openclaw/dronzer`，它只能作为 `@openclaw` 发布。如果你想作为 `@vintageayu` 发布，就需要把包名改成：

```text
@vintageayu/dronzer
```

这样可以防止插件包冒用不属于自己的组织 namespace。

常用命令：

```bash
clawhub package publish your-org/your-plugin --dry-run
clawhub package publish your-org/your-plugin
clawhub package publish your-org/your-plugin@v1.0.0
```

---

## Release 流程

1. UI、CLI 或 GitHub Workflow 收集包元数据和文件。
2. 发布请求带着选中的 owner 发到 ClawHub。
3. 服务端验证 owner 权限、包 scope、包名、版本、文件限制和 source metadata。
4. ClawHub 保存 release，并启动自动安全检查。
5. 新 release 在审核和验证完成前，不会出现在普通安装/下载入口。

如果验证失败，release 不会创建。

---

## 常见错误：scope 和 owner 不一致

如果包 scope 和选中的 owner 不一致，ClawHub 会拒绝发布，例如：

```text
Package scope "@openclaw" must match selected owner "@vintageayu".
Publish as "@openclaw" or rename this package to "@vintageayu/dronzer".
```

修复方式：

- 选择和包 scope 一致的 owner 发布。
- 或把包名改成当前 owner 对应的 scope。

如果包名 scope 已经正确，但包归属在错误 publisher 下，可以转移所有权：

```bash
clawhub package transfer @opik/opik-openclaw --to opik
```

只有你同时拥有当前包 owner 和目标 owner 的管理权限时，才能转移。转移命令不能绕过 namespace 权限。
