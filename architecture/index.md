---
title: "Architecture"
description: "High-level system overview."
---

# Architecture

Nachos is a set of Docker containers connected by a message bus. The three core services — gateway, bus, and llm-proxy — are always running. Channels and tools are added as optional containers.

## System diagram

```text
User Message → Channel Container → Bus → Gateway → LLM Proxy → LLM
                                                        ↓
                                               Tool Container (if needed)
                                                        ↓
                                          Response → Bus → Channel → User
```

## Core services

| Service    | Role                                          | Always running |
|------------|-----------------------------------------------|----------------|
| Gateway    | Policy enforcement, session management, routing | Yes          |
| Bus        | NATS message broker for inter-service communication | Yes      |
| LLM Proxy  | Provider abstraction and failover             | Yes            |

## Network isolation

Nachos creates two Docker networks:

- **`nachos-internal`** — no external access. Used for inter-container communication (gateway ↔ bus ↔ tools).
- **`nachos-egress`** — controlled external access. Used by containers that need to reach external APIs (llm-proxy, channels, browser).

Containers only join the networks they need, based on their manifest.

## Message flow

1. A user sends a message through a **channel** (Slack, Discord, Telegram, webchat)
2. The channel container publishes the message to the **bus**
3. The **gateway** receives it, checks **policies**, and manages the **session**
4. The gateway forwards the message to the **llm-proxy**
5. The llm-proxy calls the configured **LLM provider** and returns the response
6. If the LLM requests a **tool call**, the gateway evaluates the policy and routes the call to the tool container
7. The final response flows back through the bus to the channel

## Container defaults

All containers run with:

- Non-root user
- Read-only root filesystem
- Dropped Linux capabilities
- Memory and CPU limits (configurable in `[runtime.resources]`)
- PID limits

## Deep dives

- [Gateway](/architecture/gateway) — the control plane
- [Bus](/architecture/bus) — message routing
- [LLM Proxy](/architecture/llm-proxy) — provider abstraction
