---
title: "DLP"
description: "Data loss prevention scanning for sensitive data in messages and tool outputs."
---

# Data Loss Prevention (DLP)

The DLP security layer scans messages and tool I/O for sensitive data patterns using the `@nacho-labs/nachos-dlp` library. Both tool inputs and outputs are scanned before they reach the LLM context.

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

| Action | Behavior |
|--------|----------|
| `block` | Message is rejected entirely. Default for programmatic config. |
| `warn` / `alert` | Message is allowed but an alert is generated and logged. |
| `audit` | Message is allowed; finding is recorded in the audit log. |
| `allow` | Message is allowed with no action (findings still returned). |
| `redact` | Sensitive data is replaced with redaction markers; message is allowed. |

## Built-in patterns

| Pattern | Matches |
|---------|---------|
| `credit_card` | Visa, Mastercard, Amex card number formats |
| `ssn` | US Social Security Numbers (XXX-XX-XXXX) |
| `api_key` | Common API key prefixes (`sk-`, `xoxb-`) |
| `password` | Strings following `password=` or `passwd:` |

## Severity levels

DLP findings are classified by severity: `critical`, `high`, `medium`, `low`, `info`.

The default configuration checks `critical` and `high` severity patterns with a minimum confidence threshold of `0.6`.

## How it works

1. Tool inputs are scanned by the `DLPSecurityLayer` before execution
2. Tool outputs are scanned before returning to the LLM context
3. If a pattern matches, the configured `action` is applied
4. Blocked results return a `DLP_BLOCKED` error code -- original content is never exposed
5. All DLP events are recorded in the audit log (if audit is enabled)

## Secure channels

Channels can be registered as "secure" (e.g., admin channels that legitimately contain secrets). Secure channels use a reduced DLP ruleset that only enforces `critical` and `high` severity patterns, skipping PII and lower-severity detections.

## Fast-path prefilter

An optional fast-path prefilter skips the full DLP scan when no trigger keywords or patterns match. This reduces latency for messages that are unlikely to contain sensitive data.

## Recommended setup

1. Start with `action = "warn"` to understand what triggers DLP without blocking work
2. Review audit logs to tune patterns and identify false positives
3. Switch to `action = "block"` or `action = "redact"` once tuned

For production deployments, use `action = "block"` to prevent accidental exposure.

## Gotchas

- **Both inputs and outputs are scanned**: Unlike simpler DLP systems, Nachos scans both tool inputs (to prevent injecting sensitive data) and outputs (to prevent leaking it).
- **Pattern matching uses the nachos-dlp library**: This provides more sophisticated detection than simple regex, including confidence scoring.
- **DLP does not encrypt data**: It prevents accidental exposure, not determined exfiltration.
- **False positives are possible**: Short patterns or common formats may trigger false matches. Use `warn` mode first to tune.
