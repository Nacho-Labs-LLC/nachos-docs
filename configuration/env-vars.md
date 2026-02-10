---
title: "Environment Variables"
description: "Secrets and runtime configuration."
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

### LLM providers

| Variable           | Required when              | Description              |
|--------------------|----------------------------|--------------------------|
| `ANTHROPIC_API_KEY`| `llm.provider = "anthropic"` | Anthropic API key       |
| `OPENAI_API_KEY`   | `llm.provider = "openai"`    | OpenAI API key          |

### Channel tokens

| Variable              | Required when                 | Description                     |
|-----------------------|-------------------------------|---------------------------------|
| `SLACK_BOT_TOKEN`     | `channels.slack.enabled`      | Slack bot token (`xoxb-...`)   |
| `SLACK_APP_TOKEN`     | Slack socket mode             | Slack app token (`xapp-...`)   |
| `SLACK_SIGNING_SECRET`| Slack HTTP mode               | For verifying webhook requests |
| `DISCORD_BOT_TOKEN`   | `channels.discord.enabled`    | Discord bot token              |
| `TELEGRAM_BOT_TOKEN`  | `channels.telegram.enabled`   | Telegram BotFather token       |

### Runtime

| Variable              | Default | Description                          |
|-----------------------|---------|--------------------------------------|
| `NACHOS_PAIRING_TOKEN`| —       | Required token for DM pairing        |
| `REDIS_URL`           | —       | Override the Redis URL from `nachos.toml` |

## Using env vars in nachos.toml

Reference environment variables with `${VAR_NAME}` syntax:

```toml
[channels.slack]
bot_token = "${SLACK_BOT_TOKEN}"
app_token = "${SLACK_APP_TOKEN}"
```

At startup, Nachos resolves these from `.env` or the host environment. Host environment variables take precedence over `.env`.

## Gotchas

- **Missing required keys**: `nachos up` will fail with a clear error if a required env var is unset. Run `nachos config validate` to check ahead of time.
- **Quotes in `.env`**: Values are read as-is. Don't wrap values in quotes unless the quotes are part of the value.
- **`.env` is not hot-reloaded**: Changes require `nachos restart`.
