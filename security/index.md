---
title: "Security"
description: "Security overview and recommended defaults."
---

# Security

Nachos is deny-by-default. Every capability — tool access, network egress, channel permissions — must be explicitly granted. Security configuration lives in `nachos.toml` under the `[security]` section and in policy files under `policies/`.

## Security modes

Set `security.mode` in `nachos.toml`:

| Mode           | Description                                                         |
|----------------|---------------------------------------------------------------------|
| **strict**     | All tools disabled. Allowlist-only DMs. Full audit logging.         |
| **standard**   | Common tools enabled. Pairing-based DMs. Balanced security.        |
| **permissive** | Full access. Requires `i_understand_the_risks = true`.             |

```toml
[security]
mode = "standard"
```

Start with `standard`. Only use `permissive` in trusted, single-user environments.

## Key principles

- **Least privilege containers** — non-root, read-only filesystem, dropped capabilities
- **Network isolation** — containers default to `nachos-internal` (no external access)
- **Policy as code** — all security rules live in `policies/` and are version-controlled
- **Audit everything** — all actions logged with user attribution

## Rate limits

```toml
[security.rate_limits]
messages_per_minute = 30
tool_calls_per_minute = 15
llm_requests_per_minute = 30
```

## Pages

- [Policies](/security/policies) — policy engine, allowlists, and tool permissions
- [DLP](/security/dlp) — data loss prevention rules
- [Audit](/security/audit) — audit logging and retention
