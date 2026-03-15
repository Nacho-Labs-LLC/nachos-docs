---
title: "Nachos"
description: "Docker-native, modular AI assistant framework."
---

# Nachos

Nachos is a modular AI assistant framework that runs entirely in Docker. Connect messaging channels, enable tools, and enforce security policies — all from a single `nachos.toml` file.

```bash
pnpm add -g @nachos/cli
nachos init
nachos up
```

## How it works

Every component runs as an isolated container. You compose your stack by choosing which channels, tools, and security policies to enable. The gateway connects everything: it receives messages from channels, enforces policy, calls the LLM, and routes responses back.

```text
┌─────────────────────────────────────────────────┐
│              Docker Compose                      │
│  ┌───────────────────────────────────────────┐  │
│  │              Bus (NATS)                   │  │
│  └───────────────────────────────────────────┘  │
│  ┌────────┐ ┌─────────┐ ┌────────┐ ┌───────┐  │
│  │Gateway │ │LLM Proxy│ │Channels│ │ Tools │  │
│  └────────┘ └─────────┘ └────────┘ └───────┘  │
│  ┌───────────────────────────────────────────┐  │
│  │       Internal Network (isolated)         │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

## Where to start

<CardGroup cols={2}>
  <Card title="Getting started" icon="rocket" href="/start/getting-started">
    Full walkthrough — install the CLI, initialize a stack, and bring it up.
  </Card>
  <Card title="Quickstart" icon="bolt" href="/start/quickstart">
    Already know Docker? Get to a running stack in under 5 minutes.
  </Card>
  <Card title="Configuration" icon="sliders" href="/configuration/index">
    Everything you can configure in `nachos.toml` and `.env`.
  </Card>
  <Card title="Security" icon="shield" href="/security/index">
    Policies, DLP, audit logging, and security modes.
  </Card>
</CardGroup>

## Add channels

Connect Nachos to your messaging platform of choice.

<CardGroup cols={3}>
  <Card title="Slack" icon="slack" href="/channels/slack">
    Bot with slash commands and socket mode.
  </Card>
  <Card title="Discord" icon="discord" href="/channels/discord">
    Bot with slash commands and status reactions.
  </Card>
  <Card title="Telegram" icon="telegram" href="/channels/telegram">
    Personal bot via BotFather.
  </Card>
  <Card title="WhatsApp" icon="whatsapp" href="/channels/whatsapp">
    Business API with webhook verification.
  </Card>
  <Card title="Matrix" icon="hashtag" href="/channels/matrix">
    Decentralized chat with auto-join.
  </Card>
</CardGroup>

## Add capabilities

Extend the assistant with tools and skills.

<CardGroup cols={2}>
  <Card title="Tools" icon="wrench" href="/tools/index">
    Container-based capabilities: browser, filesystem, and code runner.
  </Card>
  <Card title="Skills" icon="wand-magic-sparkles" href="/skills/index">
    Lightweight CLI tools built into the gateway: Places, GIFs, Summarize, Google Workspace.
  </Card>
</CardGroup>

## Go deeper

- [Architecture](/architecture/index) — how gateway, bus, and LLM proxy fit together
- [CLI reference](/cli/commands) — all `nachos` commands
- [Troubleshooting](/reference/troubleshooting) — common issues and fixes
