---
title: "接入 SMS（Twilio）"
sidebarTitle: "接入 SMS"
description: "OpenClaw 通道接入：通过 Twilio 手机号或 Messaging Service 接收和发送 SMS 短信。"
---

# 接入 SMS（Twilio）

OpenClaw 现在可以通过 Twilio 手机号或 Messaging Service 收发短信。Gateway 会注册入站 Webhook，默认校验 Twilio 请求签名，并通过 Twilio Messages API 发回回复。

短信通道适合“没有聊天 App、只想用手机号发消息”的场景。它也更容易被误用，所以建议默认使用 `pairing` 或 `allowlist`，不要把公网短信入口直接设成开放访问。

---

## 准备事项

你需要：

- 一个 Twilio 账号
- 一个支持 SMS 的 Twilio 手机号，或一个 Twilio Messaging Service
- Twilio Account SID 和 Auth Token
- 一个能访问 Gateway 的公网 HTTPS 地址
- 一个发送者策略：`pairing`、`allowlist` 或 `open`

如果同一个 Twilio 号码同时用于 SMS 和语音通话，请在 Twilio 里分别配置 SMS Webhook 和 Voice Webhook。本文只讲 SMS。

---

## 快速配置

先保存一个 `sms.patch.json5`：

```json5
{
  channels: {
    sms: {
      enabled: true,
      accountSid: "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      authToken: "twilio-auth-token",
      fromNumber: "+15551234567",
      publicWebhookUrl: "https://gateway.example.com/webhooks/sms",
      dmPolicy: "pairing",
    },
  },
}
```

应用配置：

```bash
openclaw config patch --file ./sms.patch.json5 --dry-run
openclaw config patch --file ./sms.patch.json5
```

然后在 Twilio 控制台打开 **Phone Numbers > Manage > Active numbers**，找到这个号码的 **Messaging** 设置，把 **A message comes in** 配成：

```text
https://gateway.example.com/webhooks/sms
```

请求方法选择 `POST`。默认路径是 `/webhooks/sms`，如果你改了 `channels.sms.webhookPath`，Twilio 里也要保持一致。

---

## 本地测试和公网暴露

Gateway 必须能被 Twilio 从公网访问。如果只是本地测试，可以用 Tailscale Funnel 暴露准确的 SMS 路径：

```bash
tailscale funnel --bg --set-path /webhooks/sms http://127.0.0.1:<gateway-port>/webhooks/sms
tailscale funnel status
```

只暴露需要的路径，不要把整个 Gateway 裸露到公网。

---

## 首次配对

启动或重启 Gateway：

```bash
openclaw gateway restart
```

给 Twilio 号码发一条短信。第一次发信会生成配对请求：

```bash
openclaw pairing list sms
openclaw pairing approve sms <CODE>
```

配对码 1 小时后过期。批准后，再发一条短信就会进入正常对话。

---

## 环境变量方式

如果你不想把密钥写进配置文件，可以让宿主环境提供密钥：

```bash
export TWILIO_ACCOUNT_SID="ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export TWILIO_AUTH_TOKEN="<twilio-auth-token>"
export TWILIO_PHONE_NUMBER="+15551234567"
export SMS_PUBLIC_WEBHOOK_URL="https://gateway.example.com/webhooks/sms"
```

配置里只启用通道：

```json5
{
  channels: {
    sms: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

`TWILIO_SMS_FROM` 可以作为 `TWILIO_PHONE_NUMBER` 的别名。如果使用 Messaging Service，用 `TWILIO_MESSAGING_SERVICE_SID` 代替手机号发送者。

---

## allowlist 私有号码

如果只允许固定手机号使用：

```json5
{
  channels: {
    sms: {
      enabled: true,
      accountSid: "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      authToken: "twilio-auth-token",
      fromNumber: "+15551234567",
      publicWebhookUrl: "https://gateway.example.com/webhooks/sms",
      dmPolicy: "allowlist",
      allowFrom: ["+15557654321"],
    },
  },
}
```

---

## 故障排查

- 收不到短信：确认 Twilio 的 Webhook URL 是 HTTPS、路径是 `/webhooks/sms`，方法是 `POST`。
- Gateway 日志里签名校验失败：确认公网 URL 与 `publicWebhookUrl` 完全一致，代理层不要改 Host 或路径。
- 能收到但不回复：检查 `openclaw pairing list sms`，可能还没批准配对。
- 回复发送失败：确认 Account SID、Auth Token、`fromNumber` 或 `messagingServiceSid` 有效。

继续排查可以看 [通道故障排查](/tutorials/channels/troubleshooting) 和 [配对说明](/tutorials/channels/pairing)。
