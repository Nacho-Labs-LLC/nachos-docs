---
title: "Telegram"
description: "Connect Nachos to Telegram via BotFather with rich media support."
---

# Telegram

The Telegram channel adapter uses the `telegraf` library to connect via long polling. It supports text, photos, documents, video, audio, voice messages, and stickers.

## Prerequisites

Create a bot through [BotFather](https://t.me/botfather) on Telegram:

1. Open a chat with @BotFather
2. Send `/newbot` and follow the prompts
3. Copy the bot token

## Step 1: Add the channel

```bash
nachos add channel telegram
```

## Step 2: Set environment variables

Add to `.env`:

```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

## Step 3: Configure nachos.toml

```toml
[channels.telegram]
enabled = true
token = "${TELEGRAM_BOT_TOKEN}"

[channels.telegram.dm]
user_allowlist = ["123456789"]
pairing = true

[[channels.telegram.servers]]
ids = ["group_chat_id"]
channel_ids = ["channel_id"]
user_allowlist = ["user_id"]
mention_gating = true
```

## Step 4: Apply

```bash
nachos restart
nachos status
```

## Supported features

| Feature | Status | Notes |
|---------|--------|-------|
| Text messages (send/receive) | Supported | |
| Photo messages (inbound) | Supported | Picks largest resolution |
| Document attachments (inbound) | Supported | With filename preservation |
| Video messages (inbound) | Supported | |
| Audio/voice messages (inbound) | Supported | |
| Sticker messages (inbound) | Supported | Treated as images |
| File attachments (outbound) | Supported | URL or base64 as photo/document |
| Reply threading | Supported | Via `reply_to_message_id` |
| Typing indicators | Supported | Refreshed every 4 seconds |
| Bot commands | Supported | `/reset`, `/context`, `/help` |
| DM policy enforcement | Supported | Allowlist + pairing |
| Group policy enforcement | Supported | Mention gating via @username |
| Media captions | Supported | Used as message text |

## Bot commands

Telegram bot commands are registered via `setMyCommands`:

- `/reset` -- Reset conversation session
- `/context` -- Show current context
- `/help` -- Show available commands

<Note>
Telegram uses bot commands (not slash commands like Discord/Slack). These are set through the BotFather API and appear in the bot's command menu.
</Note>

## How it connects

The Telegram channel uses long polling by default -- no public URL or webhook setup required. The container polls the Telegram Bot API for updates.

## Gotchas

- **Bot token format**: Telegram tokens look like `123456789:ABCdef...`. The part before the colon is the bot's numeric ID.
- **User allowlist**: Telegram user IDs are numeric. Find yours by messaging @userinfobot.
- **Group chats**: To use the bot in group chats, add the chat ID to `channel_ids`. Group chat IDs are negative numbers (e.g., `-1001234567890`).
- **Privacy mode**: By default, bots in groups only see messages that mention them or start with `/`. Disable privacy mode via BotFather (`/setprivacy`) if you want the bot to see all messages.
- **No inline keyboard support**: Callback queries and inline keyboards are not currently supported.
