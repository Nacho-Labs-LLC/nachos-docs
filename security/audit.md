---
title: "Audit"
description: "Audit logging and review workflows."
---

# Audit Logging

Audit logs capture security-sensitive events: tool calls, DLP triggers, policy decisions, and user interactions. All events include timestamps and user attribution.

## Configuration

```toml
[security.audit]
enabled = true
retention_days = 30
log_inputs = true
log_outputs = true
log_tool_calls = true
```

| Key              | Type    | Default | Description                              |
|------------------|---------|---------|------------------------------------------|
| `enabled`        | boolean | `true`  | Enable audit logging                     |
| `retention_days` | integer | `30`    | Days to retain audit logs                |
| `log_inputs`     | boolean | `true`  | Log user messages (redacted by DLP)      |
| `log_outputs`    | boolean | `true`  | Log assistant responses                  |
| `log_tool_calls` | boolean | `true`  | Log all tool invocations with arguments  |

## What gets logged

| Event type        | Logged data                                    |
|-------------------|------------------------------------------------|
| `user.message`    | Channel, user ID, message content (if enabled) |
| `tool.call`       | Tool name, arguments, result status            |
| `tool.denied`     | Tool name, policy rule that denied it          |
| `dlp.trigger`     | Pattern matched, action taken, content snippet |
| `session.start`   | User ID, channel, timestamp                    |
| `session.end`     | Duration, message count                        |
| `policy.reload`   | Policy file changed, validation result         |

## Viewing logs

```bash
# View all logs
nachos logs

# Filter to gateway (where audit events are emitted)
nachos logs gateway

# JSON format for piping to other tools
# Set runtime.log_format = "json" in nachos.toml
```

Audit events are written to stdout in the gateway container. In `json` log format, each audit event is a structured JSON line that can be forwarded to external logging systems.

## Retention

Logs are retained in Docker's log driver for the number of days set by `retention_days`. For long-term storage, configure Docker's logging driver to forward to an external system (Elasticsearch, CloudWatch, etc.).

## Gotchas

- **`log_inputs = true` logs user messages**: Even with DLP redaction, consider the privacy implications.
- **Docker log rotation**: Docker's default `json-file` driver has its own rotation settings. Make sure they align with your `retention_days`.
- **Audit is gateway-only**: Events from tool containers are relayed through the gateway before being logged.
