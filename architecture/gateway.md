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
- **Scheduler** — manages cron jobs and heartbeat system for time-based automation
- **Native tools** — executes GitHub, Bitbucket, web search/fetch, and Composio tools directly in process

## Key source files

| File                                     | Purpose                        |
|------------------------------------------|--------------------------------|
| `core/gateway/src/router.ts`             | Message routing logic          |
| `core/gateway/src/cheese/policy/evaluator.ts` | Policy evaluation engine  |
| `core/gateway/src/tools/shell-tool.ts`   | CLI tool execution             |
| `core/gateway/src/skills/skill-loader.ts`| SKILL.md parsing and hot reload|
| `core/gateway/src/scheduler/scheduler.ts`| Cron job scheduler             |
| `core/gateway/src/scheduler/heartbeat.ts`| Heartbeat system               |
| `core/gateway/src/tools/github-tools.ts` | GitHub integration             |
| `core/gateway/src/tools/bitbucket-tools.ts` | Bitbucket integration       |
| `core/gateway/src/tools/web-search-tools.ts` | Brave Search API          |
| `core/gateway/src/tools/web-fetch-tools.ts` | Native web fetching        |
| `core/gateway/src/tools/composio-tools.ts` | Composio integrations       |
| `core/gateway/src/tools/cron-tools.ts`   | Cron management tools          |

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

## New Components

### Scheduler

The scheduler runs cron jobs and the heartbeat system. It:
- Checks for due jobs every `check_interval_ms` (default: 60 seconds)
- Executes `systemEvent` actions by injecting text into sessions
- Spawns `agentTurn` actions as isolated runs
- Persists jobs to the state database
- Supports `at` (one-shot), `every` (interval), and `cron` (expression) schedules

See [Cron Scheduling](/tools/cron) and [Heartbeat System](/tools/heartbeat).

### Skill Hot Reload

Skills are now watched for changes and automatically reloaded:
- Gateway watches the `skills/*/SKILL.md` files
- When a skill file changes, it's reparsed and injected on next LLM call
- No restart required for skill updates
- Enables rapid skill development iteration

### Native Tools

Several tools now run directly in the gateway process instead of Docker containers:
- **GitHub** — Uses `gh` CLI for repository, PR, issue, and workflow management
- **Bitbucket** — Direct REST API v2.0 calls for repos, PRs, issues, pipelines
- **Web Search** — Brave Search API for web queries
- **Web Fetch** — Native HTML-to-markdown converter with SSRF protection
- **Composio** — Composio SDK for Gmail, Calendar, Docs, Meet, Drive, LinkedIn

Benefits:
- Faster execution (no container overhead)
- Lower resource usage
- Simpler deployment

## Network

The gateway joins both `nachos-internal` (to communicate with bus, tools) and `nachos-egress` (to reach the llm-proxy for external LLM calls).
