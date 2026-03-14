---
title: "Discord"
description: "Connect Nachos to Discord with status reactions and slash commands."
---

# Discord

The Discord channel adapter uses `discord.js` to connect via the Discord Gateway WebSocket API. It supports real-time status emoji reactions, typing indicators, slash commands, and bot-to-bot messaging.

## Prerequisites

Create a Discord application and bot at [discord.com/developers/applications](https://discord.com/developers/applications).

Required bot permissions:
- Send Messages
- Read Message History
- Use Slash Commands
- Add Reactions (for status emojis)

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
allow_bots = false                         # Allow messages from other bots
bot_allowlist = ["bot_id_1"]               # If allow_bots=true, restrict to these IDs
typing_indicators = true                   # Show typing while processing

[channels.discord.status_emojis]
enabled = true                             # Real-time emoji reactions

[channels.discord.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["1234567890"]

[channels.discord.dm]
user_allowlist = ["user_id_1", "user_id_2"]
pairing = true

[[channels.discord.servers]]
ids = ["guild_id_1"]
channel_ids = ["channel_1", "channel_2"]
user_allowlist = ["user_a", "user_b"]
mention_gating = true
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

## Status emoji reactions

The Discord adapter provides real-time visual feedback via emoji reactions on user messages:

| Emoji | Meaning | Timing |
|-------|---------|--------|
| Brain | Thinking/reasoning | Debounced (700ms) |
| Hammer and Wrench | Generic tool execution | Debounced |
| Computer | Code-related tools (exec, read, write) | Debounced |
| Globe | Web tools (search, fetch, browser) | Debounced |
| Check Mark | Completed successfully | Held 1.5s then removed |
| Cross Mark | Error occurred | Held 2.5s then removed |
| Hourglass | Soft stall warning | After 10s inactivity |
| Warning | Hard stall warning | After 30s inactivity |

Reactions are debounced to avoid hitting Discord rate limits.

To disable:

```toml
[channels.discord.status_emojis]
enabled = false
```

## Slash commands

Discord supports slash commands registered globally via application commands:

- `/nachos status` -- Show bot status
- `/nachos help` -- Show available commands
- `/nachos config show` -- Show current configuration (admin only)
- `/nachos session reset` -- Reset conversation session
- `/nachos context` -- Show current context
- `/nachos approve <id>` -- Approve a restricted tool call
- `/nachos deny <id>` -- Deny a restricted tool call

Admin commands require the user to be in `admin_allowlist` or have Discord Administrator/ManageGuild permissions.

## Supported features

| Feature | Status | Notes |
|---------|--------|-------|
| Text messages (send/receive) | Supported | |
| File attachments (inbound) | Supported | Images and files with content type detection |
| File attachments (outbound) | Supported | Base64 buffer upload |
| Reply threading | Supported | Via `messageReference` |
| Slash commands | Supported | Registered globally |
| Status emoji reactions | Supported | Real-time processing feedback |
| Typing indicators | Supported | Refreshed every 8 seconds |
| Stall detection | Supported | Soft (10s) and hard (30s) warnings |
| Bot-to-bot messaging | Supported | Via `allow_bots` and `bot_allowlist` |
| Tool approval/deny | Supported | `/nachos approve`, `/nachos deny` |

## Gotchas

- **Server ID**: Right-click the server name in Discord > **Copy Server ID** (enable Developer Mode in Discord settings first).
- **Channel IDs**: Right-click a channel > **Copy Channel ID**.
- **Message Content intent**: Without this, the bot cannot read message text. Enable it in the developer portal.
- **One bot per token**: Each Nachos instance needs its own Discord bot application.
- **Self-reply prevention**: The bot always ignores its own messages even when `allow_bots` is true.
