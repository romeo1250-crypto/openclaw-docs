---
title: "openclaw update"
sidebarTitle: "update"
---

# `openclaw update`

`update` 用来更新 OpenClaw。源码安装、全局包安装、稳定版、beta、dev 通道的细节不完全一样，所以升级前先看清自己是哪种安装方式。

```bash
openclaw update
openclaw update status
openclaw update --dry-run
openclaw update --channel beta
openclaw update --tag main
openclaw update --no-restart
openclaw doctor
openclaw gateway restart
```

## 新手最稳路线

先做预演：

```bash
openclaw update --dry-run
```

确认没问题再更新：

```bash
openclaw update
```

更新完做体检：

```bash
openclaw doctor
openclaw status
```

如果 Gateway 没有自动重启，手动重启：

```bash
openclaw gateway restart
```

## 常用选项

- `--dry-run`：只看看会发生什么，不真正改。
- `--no-restart`：更新后不自动重启 Gateway。
- `--channel stable|beta|dev`：切换更新通道。
- `--tag <dist-tag|version|spec>`：只对这一次更新指定 npm tag、版本号或 GitHub/git 规格。
  如果是 GitHub/git 规格，新版会先打成临时 tarball，再做分阶段全局安装。
- `--yes`：自动确认，适合无人值守脚本。
- `--json`：给脚本读取的 JSON 输出。新版本会把插件同步问题、beta 插件回退、产物校验漂移放在 `postUpdate.plugins` 里。

::: info Nix 模式
如果环境变量里有 `OPENCLAW_NIX_MODE=1`，真正会改文件的 `openclaw update` 会被禁用。
Nix 安装应该更新 flake/input，而不是让 OpenClaw 自己改安装目录。

仍然可以运行：

```bash
openclaw update status
openclaw update --dry-run
```
:::

## 看懂更新结果

如果你运行：

```bash
openclaw update --json
```

看到顶层 `status: "ok"`，通常说明 OpenClaw 主程序已经更新成功。

但还要继续看 `postUpdate.plugins`：

- `status: "ok"`：插件也同步好了。
- `status: "warning"`：主程序更新成功，但某个托管插件损坏、无法加载，或者 npm 插件产物校验不一致。
- `warnings`：告诉你哪个插件需要修。
- `integrityDrifts`：告诉你 npm 插件安装出来的文件和期望校验值不一样。

遇到 `warning` 不要慌，先按顺序做：

```bash
openclaw doctor --fix
openclaw plugins inspect <插件ID> --runtime --json
openclaw plugins doctor
```

如果你不知道 `<插件ID>` 是什么，先运行：

```bash
openclaw plugins list
```

::: tip 给奶奶看的解释
更新像给家里换总电闸。`status: "ok"` 说明总电闸换好了；`postUpdate.plugins.status: "warning"` 说明某个插座接触不好。先修插座，不要重复拆总电闸。
:::

如果看到 `postUpdate.plugins.status: "error"`，情况更严重一些：
OpenClaw 已经发现启用中的插件不完整、`package.json` 无法读取、入口文件缺失，或者配置快照无效。
这时顶层 `status` 会变成 `"error"`，Gateway 不会带着未验证的插件集重启。

先修复插件，再重新更新：

```bash
openclaw doctor --fix
openclaw plugins inspect <插件ID> --runtime --json
openclaw update
```

beta 通道还有一种常见 warning：某个插件没有 beta 版本，OpenClaw 会回退到记录的默认/最新版。
这不会让主程序更新失败，但你应该确认这个插件版本符合预期。

## dev 通道的预检查变化

如果你使用 `dev` 通道，更新前会在临时目录里先跑 TypeScript 构建。
如果最新提交构建失败，它会往前找最多 10 个提交，选择最新一个能构建成功的版本。

默认不会跑 lint，因为很多用户的小机器比 CI 慢。确实需要 lint 时，再这样开：

```bash
OPENCLAW_UPDATE_PREFLIGHT_LINT=1 openclaw update --channel dev
```

普通用户用 `stable` 通道时，不需要管这一段。

dev 通道如果需要 pnpm，新版 updater 会优先走 corepack；还不行时，再临时安装 `pnpm@11`。
如果 pnpm 仍然启动失败，更新会提前停止，而不是在 pnpm 工作区里误跑 `npm run build`。

## 控制面板里点“更新并重启”时发生了什么

控制面板调用的是 Gateway 的 `update.run`。
对源码 checkout，它跑的就是源码更新流程。
对 npm/pnpm 这类全局安装，新版不会让正在运行的 Gateway 直接拆自己的安装目录，
而是做一个 **managed-service handoff**：

```text
Gateway 启动外部 helper → Gateway 退出 → helper 更新包 → helper 重启 Gateway → helper 验证新版本和健康状态
```

看结果时记住这三个值：

| 值 | 意思 |
|----|------|
| `managed-service-handoff-started` | 已经交给 helper，等它更新和重启 |
| `managed-service-handoff-unavailable` | 没找到可托管的服务边界，按返回里的命令手动跑 |
| `managed-service-handoff-failed` | helper 没启动成功，看日志后手动运行 `openclaw update` |

## 升级后先查什么

1. `openclaw status` 看版本和 Gateway 是否在线。
2. `openclaw doctor` 看配置是否需要迁移。
3. `openclaw plugins doctor` 看插件是否还兼容。
4. `openclaw channels status --probe` 看聊天通道是否真的能连。

继续阅读：[更新 OpenClaw](/tutorials/installation/updating)。
