---
title: "Matrix"
description: "Connect Nachos to the Matrix decentralized network."
---

# Matrix

The Matrix channel adapter uses `matrix-js-sdk` to connect to any Matrix homeserver. It supports text, images, files, audio, video, typing indicators, and auto-join on room invite.

## Prerequisites

You need a Matrix bot account on a homeserver:

1. Register a bot account on your homeserver (e.g., `matrix.org`, or your self-hosted instance)
2. Generate an access token for the bot account
3. Note the bot's full user ID (e.g., `@bot:matrix.org`)

<Tip>
To get an access token, you can log in with the bot account using Element or another Matrix client, then find the token in Settings > Help & About > Access Token.
</Tip>

## Step 1: Add the channel

```bash
nachos add channel matrix
```

## Step 2: Set environment variables

Add to `.env`:

```bash
MATRIX_HOMESERVER_URL=https://matrix.org
MATRIX_ACCESS_TOKEN=your-access-token
MATRIX_USER_ID=@bot:matrix.org
```

## Step 3: Configure nachos.toml

```toml
[channels.matrix]
enabled = true
homeserver_url = "https://matrix.org"
access_token = "${MATRIX_ACCESS_TOKEN}"
user_id = "@bot:matrix.org"
device_id = "NACHOS_BOT"

[channels.matrix.dm]
user_allowlist = ["@alice:matrix.org"]
pairing = true

[[channels.matrix.servers]]
ids = ["!roomId123:matrix.org"]
channel_ids = ["!roomId123:matrix.org"]
user_allowlist = ["@alice:matrix.org"]
mention_gating = true
```

## Step 4: Apply

```bash
nachos restart
nachos status
```

## Step 5: Invite the bot

Invite your bot user to the rooms where you want it to operate. The bot automatically accepts room invites and starts listening for messages.

## Supported features

| Feature | Status | Notes |
|---------|--------|-------|
| Text messages (send/receive) | Supported | m.text, m.notice, m.emote |
| Markdown/HTML formatting | Supported | Basic markdown-to-HTML conversion |
| Image messages (inbound) | Supported | mxc:// URLs converted to HTTP |
| File attachments (inbound) | Supported | m.file events with MXC URLs |
| Audio/video messages (inbound) | Supported | m.audio, m.video events |
| File attachments (outbound) | Supported | Uploaded to homeserver, sent as m.file/m.image |
| Typing indicators | Supported | Refreshed every 25 seconds |
| Auto-join on invite | Supported | Bot automatically joins rooms |
| DM detection | Supported | Rooms with exactly 2 joined members |
| DM policy enforcement | Supported | Allowlist + pairing |
| Group policy enforcement | Supported | Mention gating, room/user allowlists |
| User display names | Supported | Room-level and global profile resolution |

## How it connects

Matrix uses the Client-Server API with incremental sync:

1. The bot starts a sync loop with the homeserver
2. New messages arrive as `m.room.message` events in the sync response
3. Outbound messages are sent via the Matrix Client-Server API
4. Media is uploaded to the homeserver and referenced via `mxc://` URLs

No public URL or webhook is required -- the bot polls the homeserver.

## DM detection

The adapter detects DMs by checking if a room has exactly 2 joined members. This is a heuristic that works for most cases.

## Gotchas

- **No E2E encryption**: End-to-end encryption (Olm/Megolm) is not yet supported. Messages in encrypted rooms will not be readable.
- **No reaction support**: `m.reaction` events are not processed (planned for future).
- **No message editing**: `m.replace` events are not processed (planned for future).
- **No thread support**: MSC3440 threads are not supported (planned for future).
- **DM heuristic**: Rooms with exactly 2 members are treated as DMs. Small group rooms may be misclassified.
- **Markdown conversion**: Outbound markdown-to-HTML is basic (bold, italic, code, strikethrough, line breaks). Complex markdown may not render as expected.
- **Health check**: Based on sync state -- the bot reports healthy when sync is in `SYNCING` or `PREPARED` state.
