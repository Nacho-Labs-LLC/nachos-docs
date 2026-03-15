---
title: "WhatsApp"
description: "Connect Nachos to WhatsApp via the Cloud API with webhook verification."
---

# WhatsApp

The WhatsApp channel adapter connects to the WhatsApp Cloud API using native HTTP (no SDK). It uses webhooks to receive messages and supports text, images, documents, video, audio, and stickers.

## Prerequisites

Set up a WhatsApp Business account through the [Meta Developer Portal](https://developers.facebook.com/):

1. Create a Meta app with WhatsApp product enabled
2. Set up a WhatsApp Business phone number
3. Generate a permanent access token
4. Configure a webhook URL pointing to your Nachos instance

## Step 1: Add the channel

```bash
nachos add channel whatsapp
```

## Step 2: Set environment variables

Add to `.env`:

```bash
WHATSAPP_TOKEN=your-permanent-access-token
WHATSAPP_PHONE_NUMBER_ID=your-phone-number-id
WHATSAPP_VERIFY_TOKEN=your-custom-verify-token
WHATSAPP_APP_SECRET=your-app-secret          # Required in strict mode
```

## Step 3: Configure nachos.toml

```toml
[channels.whatsapp]
enabled = true
token = "${WHATSAPP_TOKEN}"
phone_number_id = "${WHATSAPP_PHONE_NUMBER_ID}"
verify_token = "${WHATSAPP_VERIFY_TOKEN}"
app_secret = "${WHATSAPP_APP_SECRET}"
webhook_path = "/whatsapp/webhook"
api_version = "v20.0"

[channels.whatsapp.dm]
user_allowlist = ["15551234567"]
pairing = true
```

## Step 4: Configure the webhook in Meta Developer Portal

1. Go to your app's WhatsApp settings
2. Set the webhook URL to `https://your-domain.com/whatsapp/webhook`
3. Set the verify token to match `WHATSAPP_VERIFY_TOKEN`
4. Subscribe to the `messages` webhook field

## Step 5: Apply

```bash
nachos restart
nachos status
```

## Supported features

| Feature | Status | Notes |
|---------|--------|-------|
| Text messages (send/receive) | Supported | |
| Image messages (inbound) | Supported | Downloaded and converted to data URI |
| Document attachments (inbound) | Supported | With filename |
| Video messages (inbound) | Supported | |
| Audio messages (inbound) | Supported | |
| Sticker messages (inbound) | Supported | |
| Media messages (outbound) | Supported | URL-based or uploaded via media endpoint |
| Read receipts (blue checkmarks) | Supported | Sent on first thinking event |
| Reply context | Supported | Via `context.message_id` |
| Webhook signature verification | Supported | HMAC-SHA256 with timing-safe comparison |
| DM policy enforcement | Supported | Allowlist + pairing |
| Media upload | Supported | Multipart form-data to WhatsApp media endpoint |

## Webhook security

WhatsApp webhook signature verification uses HMAC-SHA256:

- The `WHATSAPP_APP_SECRET` is used to verify that incoming webhooks are genuinely from Meta
- Signature comparison uses timing-safe comparison to prevent timing attacks
- In `strict` security mode, `app_secret` is required -- webhooks without valid signatures are rejected

<Warning>
In non-strict modes, webhook signature verification is disabled if `app_secret` is not set. Always set `WHATSAPP_APP_SECRET` in production.
</Warning>

## How it connects

WhatsApp uses a webhook-based architecture:

1. Meta sends a GET request with a verification challenge (handled automatically)
2. Incoming messages arrive as POST requests to your webhook URL
3. Outbound messages are sent via the WhatsApp Cloud API

You need a publicly accessible URL for the webhook. Use a reverse proxy (nginx, Caddy) or a tunnel (ngrok) during development.

## Gotchas

- **DM-only platform**: WhatsApp Business API is inherently 1:1. There is no group/server policy support.
- **No typing indicators**: WhatsApp Cloud API does not support typing indicators. Read receipts (blue checkmarks) are sent instead when the bot starts processing.
- **Media download limit**: 25 MB per file.
- **Temporary media URLs**: Media URLs from WhatsApp are temporary and require bearer token auth. The adapter downloads and converts them to data URIs automatically.
- **Public URL required**: Unlike Telegram (which uses polling), WhatsApp requires a publicly accessible webhook endpoint.
- **Port configuration**: The webhook server defaults to port 3002. Override with `WHATSAPP_HTTP_PORT`.
