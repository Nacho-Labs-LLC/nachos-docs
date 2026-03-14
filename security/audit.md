---
title: "Audit"
description: "Audit logging with multiple providers, HMAC integrity, and structured event schema."
---

# Audit Logging

Every security-relevant event in Nachos is recorded in the audit system. Audit events include timestamps, user attribution, policy rule matches, and structured details.

## Configuration

```toml
[security.audit]
enabled = true
retention_days = 30
log_inputs = true
log_outputs = true
log_tool_calls = true
provider = "sqlite"              # "sqlite" | "file" | "webhook" | "custom" | "composite"
path = "./state/audit.db"       # For sqlite/file providers
batch_size = 100
flush_interval_ms = 5000
```

## Event types

| Event Type | Description |
|------------|-------------|
| `policy_check` | Cheese policy evaluation result |
| `dlp_scan` | DLP scan completed |
| `dlp_block` | DLP blocked a message |
| `rate_limit` | Rate limit check |
| `session_create` | New session created |
| `session_end` | Session ended |
| `tool_execute` | Tool execution |
| `llm_request` | LLM API call |
| `channel_command` | Channel command executed |
| `config_update` | Configuration change |
| `config_reload` | Configuration reloaded |
| `error` | Error event |

## Event schema

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique audit entry identifier |
| `timestamp` | string | ISO 8601 timestamp |
| `instanceId` | string | Gateway instance identifier |
| `userId` | string | User who performed the action |
| `sessionId` | string | Related session identifier |
| `channel` | string | Channel identifier |
| `eventType` | enum | Event category |
| `action` | string | Action that was performed |
| `resource` | string | Resource identifier |
| `outcome` | enum | `allowed`, `denied`, `blocked`, `error` |
| `reason` | string | Reason for the outcome |
| `securityMode` | enum | Security mode at time of event |
| `policyMatched` | string | Policy rule ID that matched |
| `details` | object | Additional structured details |

## Providers

| Provider | Storage | Queryable | Use Case |
|----------|---------|-----------|----------|
| **SQLite** (default) | `state/audit.db` | Yes | Single-instance deployments |
| **File** | `state/audit.log` | No | Simple log files with rotation |
| **Webhook** | HTTP POST | No | External SIEM integration |
| **Custom** | User-provided module | Depends | Custom integrations |
| **Composite** | Multiple providers | Depends | Fan-out to multiple backends |

### SQLite provider

Uses WAL mode for concurrent read/write performance. Schema includes indexes on `timestamp`, `user_id`, and `session_id` for efficient querying. Batch inserts use transactions for atomicity.

### File provider

Supports log rotation:

| Setting | Default | Description |
|---------|---------|-------------|
| `rotateSize` | 10 MB | Rotate when file exceeds this size |
| `maxFiles` | 5 | Maximum rotated files to keep |
| `batchSize` | 50 | Flush buffer size |

### Webhook provider

Posts audit events in batches to a configured URL:

```toml
[security.audit]
provider = "webhook"
url = "https://siem.example.com/audit"
headers = { Authorization = "Bearer ..." }
```

Includes:
- SSRF protection (blocks private/internal addresses)
- Retry logic with exponential backoff (3 attempts)
- Configurable HTTP headers for authentication

## HMAC integrity

Both file and SQLite providers support HMAC-SHA256 signing of audit entries using the `NACHOS_AUDIT_HMAC_SECRET` environment variable:

- Each audit entry is signed and the HMAC is stored alongside the entry
- This provides tamper detection for audit logs
- If the HMAC secret is not configured, a warning is logged

<Warning>
Set `NACHOS_AUDIT_HMAC_SECRET` to a strong random value in production to enable audit log integrity verification.
</Warning>

## Viewing audit logs

```bash
# View all gateway logs (includes audit events)
nachos logs gateway

# Stream logs in real time
nachos logs gateway -f

# Use JSON format for machine processing
# Set runtime.log_format = "json" in nachos.toml
```

The Admin UI also provides an audit log viewer at `/api/audit`.

## Gotchas

- **`log_inputs = true` logs user messages**: Even with DLP redaction, consider the privacy implications.
- **Docker log rotation**: Docker's `json-file` driver has its own rotation. Align with your `retention_days`.
- **Audit is gateway-centric**: Events from tool containers are relayed through the gateway before being logged.
- **HMAC is recommended**: Without `NACHOS_AUDIT_HMAC_SECRET`, audit logs have no tamper detection.
