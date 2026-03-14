---
title: "Slack"
description: "Connect Nachos to Slack via Socket Mode or HTTP webhooks."
---

# Slack

The Slack channel adapter uses `@slack/bolt` to connect Nachos to your Slack workspace. It supports both Socket Mode (for development) and HTTP Mode (for production behind a reverse proxy).

## Prerequisites

Create a Slack app at [api.slack.com/apps](https://api.slack.com/apps).

Required bot token scopes:
- `chat:write`
- `app_mentions:read`
- `channels:history`
- `im:history`
- `im:write`

## Step 1: Add the channel

```bash
nachos add channel slack
```

## Step 2: Set environment variables

Add to `.env`:

```bash
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-token
```

For HTTP mode (instead of socket mode):

```bash
SLACK_SIGNING_SECRET=your-signing-secret
```

## Step 3: Configure nachos.toml

```toml
[channels.slack]
enabled = true
mode = "socket"                            # "socket" | "http"
app_token = "${SLACK_APP_TOKEN}"
bot_token = "${SLACK_BOT_TOKEN}"
typing_indicators = true                   # Subscribe to status events

[channels.slack.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["U123456"]

[channels.slack.dm]
user_allowlist = ["U111", "U222"]
pairing = false

[[channels.slack.servers]]
ids = ["T123456"]
channel_ids = ["C111", "C222"]
user_allowlist = ["U123", "U456"]
mention_gating = true
```

## Connection modes

| Mode | Use when | Requires |
|------|----------|----------|
| `socket` | Development, no public URL needed | `SLACK_APP_TOKEN` |
| `http` | Production, behind a reverse proxy | `SLACK_SIGNING_SECRET`, public URL |

For HTTP mode:

```toml
[channels.slack]
mode = "http"
signing_secret = "${SLACK_SIGNING_SECRET}"
webhook_path = "/slack/events"
```

## Supported features

| Feature | Status | Notes |
|---------|--------|-------|
| Text messages (send/receive) | Supported | |
| File attachments (inbound) | Supported | Via `url_private` or `url_private_download` |
| File attachments (outbound) | Supported | Base64 upload or URL link |
| Threads | Supported | Via `thread_ts` |
| Slash commands (`/nachos`) | Supported | status, help, config show, session reset, context |
| DM policy enforcement | Supported | Allowlist + pairing |
| Group policy enforcement | Supported | Mention gating, channel/user allowlists |
| Bot message filtering | Supported | Drops all bot subtypes and bot_id messages |

## Step 4: Apply

```bash
nachos restart
nachos status
```

## Gotchas

- **Socket mode requires an app-level token** (`xapp-...`), not the bot token. Generate it under your app's **Basic Information > App-Level Tokens**.
- **Channel IDs, not names**: Use Slack channel IDs (`C...`) in config, not `#channel-name`.
- **User IDs, not usernames**: Use Slack member IDs (`U...`), not display names.
- **Typing indicators**: Slack API does not allow bots to send typing indicators in regular channels. Status events are subscribed for future support.
- **No bot-to-bot messaging**: Bot messages are always filtered out (unlike Discord which supports `allow_bots`).
