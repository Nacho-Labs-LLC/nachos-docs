---
title: "Channels"
description: "Connect Nachos to Slack, Discord, Telegram, WhatsApp, and Matrix."
---

# Channels

Channels connect Nachos to messaging platforms. Each channel runs as an isolated Docker container that normalizes platform messages into a common format and communicates with the gateway over the NATS bus.

## Available channels

| Channel | Protocol | Connection Mode | Config section |
|---------|----------|----------------|----------------|
| Slack | Socket / HTTP | WebSocket or webhook | `[channels.slack]` |
| Discord | Gateway WS | WebSocket | `[channels.discord]` |
| Telegram | Long poll | HTTP polling | `[channels.telegram]` |
| WhatsApp | Webhook | HTTP server | `[channels.whatsapp]` |
| Matrix | Client sync | Incremental sync | `[channels.matrix]` |

## Feature comparison

| Feature | Slack | Discord | Telegram | WhatsApp | Matrix |
|---------|-------|---------|----------|----------|--------|
| Text messages | Yes | Yes | Yes | Yes | Yes |
| Image inbound | Yes | Yes | Yes | Yes | Yes |
| File inbound | Yes | Yes | Yes | Yes | Yes |
| Video/audio inbound | No | No | Yes | Yes | Yes |
| Attachments outbound | Yes | Yes | Yes | Yes | Yes |
| Typing indicators | No\* | Yes | Yes | No\*\* | Yes |
| Status emoji reactions | No | Yes | No | No | No |
| Slash/bot commands | Yes | Yes | Yes | No | No |
| Thread/reply support | Yes | Yes | Yes | Yes | No |
| DM policy | Yes | Yes | Yes | Yes | Yes |
| Group policy | Yes | Yes | Yes | No\*\*\* | Yes |
| Pairing flow | Yes | Yes | Yes | Yes | Yes |
| Bot-to-bot support | No | Yes | No | No | No |
| Read receipts | No | No | No | Yes | No |
| Auto-join on invite | N/A | N/A | N/A | N/A | Yes |

\* Slack API does not allow bots to show typing in regular channels.
\*\* WhatsApp Cloud API does not support typing; read receipts are used instead.
\*\*\* WhatsApp Business API is 1:1 only (DM-based).

## Adding a channel

```bash
nachos add channel slack
```

This scaffolds the channel config in `nachos.toml`. You still need to:

1. Create a bot on the platform (Slack app, Discord bot, Telegram BotFather, etc.)
2. Add the bot token to `.env`
3. Configure server/channel allowlists in `nachos.toml`
4. Run `nachos restart`

## Common configuration

Every channel supports DM and group access control:

```toml
# DM access control
[channels.discord.dm]
user_allowlist = ["user_id_1", "user_id_2"]
pairing = true

# Server/group restrictions
[[channels.discord.servers]]
ids = ["guild_id_1"]
channel_ids = ["channel_1", "channel_2"]
user_allowlist = ["user_a", "user_b"]
mention_gating = true

# Slash command controls
[channels.discord.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["admin_user_id"]
```

## Channel guides

- [Slack](/channels/slack)
- [Discord](/channels/discord)
- [Telegram](/channels/telegram)
- [WhatsApp](/channels/whatsapp)
- [Matrix](/channels/matrix)
