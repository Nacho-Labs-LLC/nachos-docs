---
title: "Channels"
description: "Chat integrations and setup."
---

# Channels

Channels connect Nachos to messaging platforms. Each channel runs as an isolated Docker container that communicates with the gateway over the internal bus.

## Available channels

| Channel    | Protocol     | Config section         |
|------------|-------------|------------------------|
| Webchat    | HTTP/WS     | `[channels.webchat]`   |
| Slack      | Socket / HTTP | `[channels.slack]`   |
| Discord    | Gateway WS  | `[channels.discord]`   |
| Telegram   | Long poll   | `[channels.telegram]`  |

## Adding a channel

```bash
nachos add channel slack
```

This scaffolds the channel config in `nachos.toml`. You still need to:

1. Create a bot on the platform (Slack app, Discord bot, Telegram BotFather)
2. Add the bot token to `.env`
3. Configure server/channel allowlists in `nachos.toml`
4. Run `nachos restart`

## Webchat (default)

Webchat is enabled by default on port 8080 and requires no external credentials:

```toml
[channels.webchat]
enabled = true
port = 8080
```

## Common configuration

Every channel supports server-level allowlists:

```toml
[[channels.slack.servers]]
id = "T123456"
channel_ids = ["C111", "C222"]
user_allowlist = ["U123", "U456"]
```

- `channel_ids` — restrict the bot to specific channels
- `user_allowlist` — restrict which users can interact with the bot

## Channel guides

- [Slack](/channels/slack)
- [Discord](/channels/discord)
- [Telegram](/channels/telegram)
