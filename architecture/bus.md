---
title: "Message Bus"
description: "NATS-based message routing between all Nachos containers."
---

# Message Bus

The bus is a NATS server that provides request/reply messaging between all Nachos containers. It is the communication backbone -- no container talks directly to another.

## Why NATS

- Lightweight (single binary, minimal memory footprint)
- Built-in request/reply semantics for tool calls
- Pub/sub for broadcast events (policy reload, health, status updates)
- No external dependencies
- Token-based authentication

## Topic structure

All messages are routed via NATS subjects. The topic definitions live in `core/bus/src/topics.ts`.

### Channel topics

| Topic | Direction | Description |
|-------|-----------|-------------|
| `nachos.channel.<id>.inbound` | Platform to Gateway | Normalized user messages |
| `nachos.channel.<id>.outbound` | Gateway to Platform | Assistant responses |

### Status topics

| Topic | Description |
|-------|-------------|
| `nachos.status.<sessionId>.thinking` | LLM is processing |
| `nachos.status.<sessionId>.tool` | Tool execution in progress |
| `nachos.status.<sessionId>.done` | Response complete |
| `nachos.status.<sessionId>.error` | Error occurred |

### Tool topics

| Topic | Description |
|-------|-------------|
| `nachos.tool.<name>.request` | Tool call dispatch (request/reply) |

### System topics

| Topic | Description |
|-------|-------------|
| `nachos.system.policy.reload` | Policy file change notification |
| `nachos.system.health` | Health check broadcasts |
| `nachos.audit.log` | Audit event publishing |
| `nachos.llm.stream.<sessionId>` | LLM streaming chunks |

## Authentication

The NATS bus requires token authentication:

```yaml
command: ["-c", "/etc/nats/nats-server.conf", "--auth", "${NATS_TOKEN:?Required}"]
```

The `NATS_TOKEN` environment variable is required -- Docker Compose fails to start without it. All containers that connect to NATS must provide this token.

## Network

The bus runs on `nachos-internal` only. It has no external network access and is not exposed on host ports by default.

## Gotchas

- **Bus must start first**: Other containers wait for the bus to be healthy before connecting. If the bus fails to start, everything else times out.
- **No persistence**: NATS runs in-memory. Messages are not persisted. If the bus restarts, in-flight messages are lost. Sessions and state are stored separately in Redis/SQLite.
- **No external access**: The bus is on `nachos-internal` only. It is not reachable from outside Docker.
- **Token is required**: Set `NATS_TOKEN` to a strong random value (`openssl rand -hex 32`).
