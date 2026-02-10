---
title: "DLP"
description: "Data loss prevention rules."
---

# Data Loss Prevention (DLP)

DLP rules prevent sensitive data from leaking through assistant responses, tool calls, or channel messages. Rules are evaluated before any outbound action.

## Configuration

```toml
[security.dlp]
enabled = true
action = "warn"
patterns = [
    "credit_card",
    "ssn",
    "api_key",
    "password"
]
```

## Actions

| Action    | Behavior                                                      |
|-----------|---------------------------------------------------------------|
| `block`   | Stops the action entirely. User sees an error.               |
| `warn`    | Action proceeds but a warning is logged and shown to user.   |
| `audit`   | Action proceeds silently. Event is recorded in audit log.    |
| `allow`   | No DLP check for matched patterns.                           |
| `redact`  | Sensitive content is replaced with `[REDACTED]` before delivery. |

## Built-in patterns

| Pattern       | Matches                                    |
|---------------|--------------------------------------------|
| `credit_card` | Visa, Mastercard, Amex card number formats |
| `ssn`         | US Social Security Numbers (XXX-XX-XXXX)   |
| `api_key`     | Common API key prefixes (`sk-`, `xoxb-`)   |
| `password`    | Strings following `password=` or `passwd:`  |

## How it works

1. The assistant generates a response or a tool produces output
2. The DLP engine scans the content against enabled patterns
3. If a match is found, the configured `action` is applied
4. All DLP events are recorded in the audit log (if audit is enabled)

## Recommended setup

- Start with `action = "warn"` to understand what triggers DLP without blocking work
- Review audit logs to tune patterns
- Switch to `action = "block"` or `action = "redact"` once tuned

## Gotchas

- DLP runs on **outbound** content only — it does not scan user input
- Pattern matching is regex-based. False positives are possible with short patterns.
- DLP does not encrypt data. It prevents accidental exposure, not determined exfiltration.
