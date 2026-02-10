---
title: "Message Bus"
description: "NATS-based message routing."
---

# Message Bus

The bus is a NATS server that provides request/reply messaging between all Nachos containers. It's the communication backbone — no container talks directly to another.

## Why NATS

- Lightweight (single binary, minimal memory footprint)
- Built-in request/reply semantics
- Pub/sub for broadcast events (e.g., policy reload notifications)
- No external dependencies

## How it's used

| Pattern        | Use case                                    |
|----------------|---------------------------------------------|
| Request/reply  | Channel → gateway, gateway → tool calls     |
| Pub/sub        | Policy reload events, health broadcasts     |

## Topic structure

Messages are routed via NATS subjects (topics). The topic definitions live in `core/bus/src/topics.ts`.

Common topic patterns:

```text
nachos.channel.{channel_name}.inbound    # Messages from users
nachos.channel.{channel_name}.outbound   # Messages to users
nachos.gateway.request                    # Requests to the gateway
nachos.tool.{tool_name}.call             # Tool call dispatch
nachos.tool.{tool_name}.result           # Tool call results
nachos.system.policy.reload              # Policy file change notification
nachos.system.health                     # Health check broadcasts
```

## Configuration

The bus is configured automatically by `nachos up`. No manual configuration is needed. It runs on `nachos-internal` only — it has no external network access.

## Resource usage

NATS is lightweight. Default resource allocation:

```toml
[runtime.resources]
memory = "512MB"
cpus = 0.5
```

These defaults are shared across containers. The bus itself typically uses far less than its allocation.

## Gotchas

- **Bus must start first**: Other containers wait for the bus to be healthy before connecting. If the bus fails to start, everything else will timeout.
- **No persistence**: NATS runs in-memory. Messages are not persisted. If the bus restarts, in-flight messages are lost (sessions are stored in Redis, not the bus).
- **No external access**: The bus is on `nachos-internal` only. It is not reachable from outside Docker.
