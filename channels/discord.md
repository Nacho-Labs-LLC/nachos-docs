---
title: "Discord"
description: "Discord setup and configuration."
---

# Discord

## Prerequisites

Create a Discord application and bot at [discord.com/developers/applications](https://discord.com/developers/applications).

Required bot permissions:
- Send Messages
- Read Message History
- Use Slash Commands

Enable the **Message Content** intent under **Bot > Privileged Gateway Intents**.

## Step 1: Add the channel

```bash
nachos add channel discord
```

## Step 2: Set environment variables

Add to `.env`:

```bash
DISCORD_BOT_TOKEN=your-bot-token
```

## Step 3: Configure nachos.toml

```toml
[channels.discord]
enabled = true
token = "${DISCORD_BOT_TOKEN}"

[channels.discord.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["1234567890"]

[[channels.discord.servers]]
id = "1234567890"
channel_ids = ["111", "222"]
user_allowlist = ["user_a", "user_b"]
```

## Step 4: Invite the bot

Use the OAuth2 URL generator in your Discord app settings:
1. Go to **OAuth2 > URL Generator**
2. Select scopes: `bot`, `applications.commands`
3. Select bot permissions listed above
4. Copy the generated URL and open it to invite the bot to your server

## Step 5: Apply

```bash
nachos restart
nachos status
```

## Status reactions

Nachos adds emoji reactions to messages in real time to show what the assistant is doing.

| Emoji | Meaning |
| ----- | ------- |
| 🧠 | Thinking |
| 🛠️ | Using a tool |
| 💻 | Running code or editing files |
| 🌐 | Fetching from the web |
| ✅ | Done |
| ❌ | Error |
| ⏳ | Still working (10 s) |
| ⚠️ | Long wait (30 s) |

Reactions are debounced to avoid hitting Discord rate limits. The ✅ and ❌ reactions auto-remove after a short hold so the message stays clean.

Status reactions are enabled by default. To disable them, add to `nachos.toml`:

```toml
[channels.discord.status_reactions]
enabled = false
```

> **Permissions**: The bot needs **Add Reactions** permission in your server for status emojis to appear. If reactions are missing, check bot permissions in the Discord server settings.

## Gotchas

- **Server ID**: Right-click the server name in Discord > **Copy Server ID** (enable Developer Mode in Discord settings first).
- **Channel IDs**: Right-click a channel > **Copy Channel ID**.
- **Message Content intent**: Without this, the bot cannot read message text. Enable it in the developer portal.
- **One bot per token**: Each Nachos instance needs its own Discord bot application.
