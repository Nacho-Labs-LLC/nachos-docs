---
title: "Nachos"
description: "Docker-native, modular AI assistant framework."
---

# Nachos

Nachos is a modular AI assistant framework that runs entirely in Docker. Connect messaging channels, enable tools, and enforce security policies — all from a single `nachos.toml` file.

## How it works

Every component runs as an isolated container. You compose your stack by choosing which channels (Slack, Discord, Telegram), tools (browser, filesystem, code runner), and security policies to enable.

```text
┌─────────────────────────────────────────────────┐
│              Docker Compose                      │
│  ┌───────────────────────────────────────────┐  │
│  │            Bus (NATS/Redis)               │  │
│  └───────────────────────────────────────────┘  │
│  ┌────────┐ ┌─────────┐ ┌────────┐ ┌───────┐  │
│  │Gateway │ │LLM Proxy│ │Channels│ │ Tools │  │
│  └────────┘ └─────────┘ └────────┘ └───────┘  │
│  ┌───────────────────────────────────────────┐  │
│  │       Internal Network (isolated)         │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

## Quick start

```bash
pnpm add -g @nachos/cli
nachos init
nachos up
```

Edit `nachos.toml` to add channels and tools, then `nachos restart` to apply.

## Where to go next

- [Getting started](/start/getting-started) — full walkthrough from zero to running stack
- [Quickstart](/start/quickstart) — minimal steps for experienced users
- [Install](/install/index) — CLI, Docker, and update instructions
- [Configuration](/configuration/index) — `nachos.toml` and environment variables
- [Security](/security/index) — policies, DLP, and audit logging
- [Architecture](/architecture/index) — how gateway, bus, and llm-proxy fit together
