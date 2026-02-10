---
title: "Telegram"
description: "Telegram setup and configuration."
---

# Telegram

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

[[channels.telegram.servers]]
id = "9876543210"
channel_ids = ["333", "444"]
user_allowlist = ["user_c", "user_d"]
```

## Step 4: Apply

```bash
nachos restart
nachos status
```

## How it connects

The Telegram channel uses long polling by default — no public URL or webhook setup required. The container polls the Telegram Bot API for updates.

## Gotchas

- **Bot token format**: Telegram tokens look like `123456789:ABCdef...`. The part before the colon is the bot's numeric ID.
- **User allowlist**: Telegram user IDs are numeric. Find yours by messaging @userinfobot.
- **Group chats**: To use the bot in group chats, add the chat ID to `channel_ids`. Group chat IDs are negative numbers (e.g., `-1001234567890`).
- **Privacy mode**: By default, bots in groups only see messages that mention them or start with `/`. Disable privacy mode via BotFather (`/setprivacy`) if you want the bot to see all messages.
