---
title: "Getting Started"
description: "Install the CLI, initialize a stack, and bring it up."
---

# Getting Started

This guide walks you from zero to a running Nachos stack with a working webchat channel.

## Prerequisites

- **Docker Desktop** or **Docker Engine** — the daemon must be running
- **Node.js 22+** and **pnpm** — for the CLI
- An **LLM API key** — Anthropic, OpenAI, or a local Ollama instance

## Step 1: Install the CLI

```bash
pnpm add -g @nachos/cli
nachos --version
```

## Step 2: Initialize your project

```bash
nachos init
```

This creates:
- `nachos.toml` — your stack configuration
- `.env` — placeholder for API keys and secrets
- `policies/` — default security policy files

## Step 3: Configure your LLM provider

Open `.env` and add your API key:

```bash
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

Or for OpenAI:

```bash
OPENAI_API_KEY=sk-xxxxx
```

The default `nachos.toml` points to Anthropic. Change `[llm].provider` and `[llm].model` if you want a different provider.

## Step 4: Start the stack

```bash
nachos up
```

Nachos pulls container images, creates the internal Docker network, and starts all enabled services. On first run this takes a minute or two.

## Step 5: Verify

```bash
nachos status
```

All services should show `healthy`. Open the webchat at `http://localhost:8080` (enabled by default).

## What's running

| Container  | Purpose                              |
|------------|--------------------------------------|
| gateway    | Session management, policy enforcement, routing |
| bus        | NATS message broker for inter-service communication |
| llm-proxy  | Normalizes LLM provider APIs         |
| webchat    | Browser-based chat interface          |

## Next steps

- **Add a channel**: `nachos add channel slack` — see [Channels](/channels/index)
- **Enable a tool**: set `tools.browser.enabled = true` in `nachos.toml` — see [Tools](/tools/index)
- **Tighten security**: adjust `security.mode` — see [Security](/security/index)
- **Validate config**: `nachos config validate`
