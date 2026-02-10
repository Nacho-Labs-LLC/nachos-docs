---
title: "Slack"
description: "Slack setup and configuration."
---

# Slack

## Prerequisites

You need a Slack app with bot permissions. Create one at [api.slack.com/apps](https://api.slack.com/apps).

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
mode = "socket"
app_token = "${SLACK_APP_TOKEN}"
bot_token = "${SLACK_BOT_TOKEN}"

[channels.slack.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["U123456"]

[[channels.slack.servers]]
id = "T123456"
channel_ids = ["C111", "C222"]
user_allowlist = ["U123", "U456"]
```

## Connection modes

| Mode     | Use when                                    | Requires                    |
|----------|---------------------------------------------|-----------------------------|
| `socket` | Development, no public URL needed           | `SLACK_APP_TOKEN`           |
| `http`   | Production, behind a reverse proxy          | `SLACK_SIGNING_SECRET`, public URL |

For HTTP mode:

```toml
[channels.slack]
mode = "http"
signing_secret = "${SLACK_SIGNING_SECRET}"
webhook_path = "/slack/events"
```

## Slash commands

Nachos registers slash commands in Slack. Control which are available:

```toml
[channels.slack.commands]
enabled = ["status", "help", "config.show", "session.reset", "context"]
admin_allowlist = ["U123456"]
```

Only users in `admin_allowlist` can run admin commands like `config.show`.

## Step 4: Apply

```bash
nachos restart
nachos status
```

## Gotchas

- **Socket mode requires an app-level token** (`xapp-...`), not the bot token. Generate it under your app's **Basic Information > App-Level Tokens**.
- **Channel IDs, not names**: Use Slack channel IDs (`C...`) in config, not `#channel-name`.
- **User IDs, not usernames**: Use Slack member IDs (`U...`), not display names.
