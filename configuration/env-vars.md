---
title: "Environment Variables"
description: "All secrets and runtime configuration variables."
---

# Environment Variables

Secrets and API keys live in a `.env` file in your project root. Nachos passes these into containers at startup. Never commit `.env` to source control.

## Creating the file

`nachos init` creates a `.env` with placeholders. You can also create one manually:

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

## Reference

### Infrastructure (required)

| Variable | Required | Description |
|----------|----------|-------------|
| `NATS_TOKEN` | Yes | NATS bus authentication token |
| `REDIS_PASSWORD` | Yes | Redis authentication password |

<Warning>
Set both `NATS_TOKEN` and `REDIS_PASSWORD` to strong random values. Docker Compose will fail to start without them. Generate with: `openssl rand -hex 32`
</Warning>

### LLM providers

| Variable | Required when | Description |
|----------|--------------|-------------|
| `ANTHROPIC_API_KEY` | `llm.provider = "anthropic"` | Anthropic API key |
| `ANTHROPIC_SETUP_TOKEN` | Alternative to API key | Anthropic subscription token |
| `OPENAI_API_KEY` | `llm.provider = "openai"` | OpenAI API key |

### Channel tokens

| Variable | Required when | Description |
|----------|--------------|-------------|
| `SLACK_BOT_TOKEN` | Slack enabled | Slack bot token (`xoxb-...`) |
| `SLACK_APP_TOKEN` | Slack socket mode | Slack app token (`xapp-...`) |
| `SLACK_SIGNING_SECRET` | Slack HTTP mode | Webhook request verification |
| `DISCORD_BOT_TOKEN` | Discord enabled | Discord bot token |
| `TELEGRAM_BOT_TOKEN` | Telegram enabled | BotFather token |
| `WHATSAPP_TOKEN` | WhatsApp enabled | Meta access token |
| `WHATSAPP_PHONE_NUMBER_ID` | WhatsApp enabled | Business phone number ID |
| `WHATSAPP_VERIFY_TOKEN` | WhatsApp enabled | Webhook verification token |
| `WHATSAPP_APP_SECRET` | WhatsApp strict mode | Webhook signature verification |
| `MATRIX_HOMESERVER_URL` | Matrix enabled | Homeserver URL |
| `MATRIX_ACCESS_TOKEN` | Matrix enabled | Bot account access token |
| `MATRIX_USER_ID` | Matrix enabled | Bot user ID |

### Tool integrations

| Variable | Required when | Description |
|----------|--------------|-------------|
| `GH_TOKEN` / `GITHUB_TOKEN` | GitHub tool enabled | GitHub personal access token |
| `BITBUCKET_USERNAME` | Bitbucket app password auth | Bitbucket username |
| `BITBUCKET_APP_PASSWORD` | Bitbucket app password auth | Bitbucket app password |
| `BITBUCKET_TOKEN` | Bitbucket OAuth auth | Bitbucket OAuth token |
| `BRAVE_SEARCH_API_KEY` | Web search enabled | Brave Search API key |
| `COMPOSIO_API_KEY` | Composio enabled | Composio SDK API key |

### Security and admin

| Variable | Default | Description |
|----------|---------|-------------|
| `NACHOS_ADMIN_TOKEN` | Auto-generated | Admin UI authentication token |
| `NACHOS_AUDIT_HMAC_SECRET` | (none) | HMAC key for audit log integrity |
| `NACHOS_PAIRING_TOKEN` | (none) | Token for DM pairing flow |

### Runtime overrides

| Variable | Default | Description |
|----------|---------|-------------|
| `NACHOS_CONFIG_PATH` | `./nachos.toml` | Override config file path |
| `REDIS_URL` | (from config) | Override Redis URL |

## Using env vars in nachos.toml

Reference environment variables with `${VAR_NAME}` syntax:

```toml
[channels.slack]
bot_token = "${SLACK_BOT_TOKEN}"
app_token = "${SLACK_APP_TOKEN}"
```

At startup, Nachos resolves these from `.env` or the host environment. Host environment variables take precedence over `.env`.

Docker Compose enforces required variables with the `${VAR:?Required}` syntax, preventing startup without critical secrets.

## Gotchas

- **Missing required keys**: `nachos up` fails with a clear error if a required env var is unset. Run `nachos config validate` to check ahead of time.
- **Quotes in `.env`**: Values are read as-is. Do not wrap values in quotes unless the quotes are part of the value.
- **`.env` is not hot-reloaded**: Changes require `nachos restart`.
- **Never commit `.env`**: Ensure it is in both `.gitignore` and `.dockerignore`.
