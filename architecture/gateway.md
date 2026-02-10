---
title: "Gateway"
description: "Security and orchestration layer."
---

# Gateway

The gateway is the control plane of Nachos. Every message and tool call flows through it. It enforces security policies, manages sessions, and routes requests between channels, the LLM, and tools.

## Responsibilities

- **Policy enforcement** — evaluates every action against loaded policies before allowing it
- **Session management** — tracks conversation state per user and channel
- **Tool routing** — dispatches tool calls to the correct container and returns results
- **Audit logging** — emits structured audit events for all security-relevant actions
- **DLP scanning** — checks outbound content against data loss prevention rules
- **Skill loading** — parses SKILL.md files and injects CLI tool documentation into LLM prompts

## Key source files

| File                                     | Purpose                        |
|------------------------------------------|--------------------------------|
| `core/gateway/src/router.ts`             | Message routing logic          |
| `core/gateway/src/salsa/policy/evaluator.ts` | Policy evaluation engine  |
| `core/gateway/src/tools/shell-tool.ts`   | CLI tool execution             |
| `core/gateway/src/skills/skill-loader.ts`| SKILL.md parsing               |

## How requests flow through the gateway

1. Message arrives from the bus (published by a channel)
2. Gateway checks user against allowlist/pairing rules
3. Session state is loaded (or created for new conversations)
4. Message + session context is sent to llm-proxy
5. LLM response is received. If it contains tool calls:
   - Each tool call is checked against policies
   - Allowed calls are dispatched to tool containers via the bus
   - Results are fed back to the LLM for the next turn
6. Final response passes through DLP scanning
7. Response is published to the bus for delivery to the channel

## Session storage

Sessions are stored in Redis by default:

```toml
[runtime.state.session]
provider = "redis"
redis_url = "redis://localhost:6379"
ttl_seconds = 86400
```

Sessions expire after `ttl_seconds` of inactivity (default: 24 hours).

## Network

The gateway joins both `nachos-internal` (to communicate with bus, tools) and `nachos-egress` (to reach the llm-proxy for external LLM calls).
