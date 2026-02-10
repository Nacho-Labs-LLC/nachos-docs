---
title: "Quickstart"
description: "Minimal steps to a running stack."
---

# Quickstart

For experienced users who want the shortest path to a running Nachos instance.

## Prerequisites

- Docker running
- Node.js 22+ and pnpm
- An LLM API key

## Steps

```bash
# Install CLI
pnpm add -g @nachos/cli

# Initialize with sensible defaults
nachos init --defaults

# Add your API key
echo "ANTHROPIC_API_KEY=sk-ant-xxxxx" >> .env

# Start the stack
nachos up
```

Open `http://localhost:8080` to chat.

## Verify health

```bash
nachos status
```

## Common next steps

```bash
# Add Slack
nachos add channel slack
# Then set SLACK_BOT_TOKEN and SLACK_APP_TOKEN in .env

# Enable the browser tool
# Set tools.browser.enabled = true in nachos.toml

# Apply changes
nachos restart

# View logs
nachos logs

# Validate configuration
nachos config validate
```

## Gotchas

- `nachos init --defaults` enables webchat on port 8080 and standard security mode. Edit `nachos.toml` to change these.
- If port 8080 is taken, change `channels.webchat.port` in `nachos.toml`.
- Docker must be running before `nachos up`. Check with `docker info`.
