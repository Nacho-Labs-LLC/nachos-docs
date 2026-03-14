---
title: "Security"
description: "Defense-in-depth security model with deny-by-default policies."
---

# Security

Nachos implements a **defense-in-depth** security architecture with multiple independent layers of protection. The core philosophy is **deny by default** -- every action, tool invocation, network request, and message must be explicitly authorized through policy rules.

## Security modes

Set `security.mode` in `nachos.toml`:

| Mode | Policy File | DM Access | Tools | Filesystem | Network | Audit |
|------|------------|-----------|-------|------------|---------|-------|
| **strict** | `strict.yaml` | Allowlist only | All disabled | Denied | Denied | Full |
| **standard** | `standard.yaml` | Pairing-based | Common tools enabled | `/workspace` only | Allowlisted domains | Full |
| **permissive** | `permissive.yaml` | All allowed | Most tools enabled | Broad (system dirs blocked) | Read allowed | Full |

```toml
[security]
mode = "standard"
```

Start with `standard`. Only use `permissive` in trusted, single-user environments -- it requires setting `i_understand_the_risks = true`.

## Defense-in-depth layers

```
Layer 1: Network Isolation (Docker networks, no external access by default)
Layer 2: Container Hardening (non-root, read-only FS, no-new-privileges, resource limits)
Layer 3: Policy Engine (Cheese -- YAML rules evaluated <1ms)
Layer 4: Security Tier + Approval (RESTRICTED tools require human approval)
Layer 5: SSRF Protection (domain allowlists, private IP blocking, DNS validation)
Layer 6: DLP Scanning (sensitive data detection and blocking)
Layer 7: Rate Limiting (sliding window per user per action)
Layer 8: Audit Logging (every security decision recorded with HMAC integrity)
```

## Rate limits

Rate limits are enforced per user per minute using a sliding window algorithm:

| Action | Strict | Standard | Permissive |
|--------|--------|----------|------------|
| Messages per minute | 20 | 30 | 120 |
| Tool calls per minute | 5 | 15 | 60 |
| LLM requests per minute | 20 | 30 | 120 |

```toml
[security.rate_limits]
messages_per_minute = 30
tool_calls_per_minute = 15
llm_requests_per_minute = 30
```

Rate limiting supports memory (in-process) and Redis (distributed) backends. Redis is used automatically when available, with fallback to memory.

## Approval workflow

Tools with security tier RESTRICTED (3) or higher trigger user approval:

1. Gateway detects the tool requires approval
2. Approval request is sent to the user's channel with tool name, parameters, and a request ID
3. User responds with `/approve <id>` or `/deny <id>`
4. If no response within 2 minutes, the request is auto-denied

Configure approver allowlists:

```toml
[security.approval]
approver_allowlist = ["user_id_1", "user_id_2"]
```

## Security configuration

```toml
[security]
mode = "standard"

[security.dlp]
enabled = true
action = "block"
patterns = ["credit_card", "ssn", "api_key", "password"]

[security.approval]
approver_allowlist = ["user_id_1"]

[security.rate_limits]
messages_per_minute = 30
tool_calls_per_minute = 15
llm_requests_per_minute = 30

[security.audit]
enabled = true
retention_days = 30
log_inputs = true
log_outputs = true
log_tool_calls = true
provider = "sqlite"
path = "./state/audit.db"
```

## Pages

- [Policies](/security/policies) -- Cheese policy engine, rule structure, and examples
- [DLP](/security/dlp) -- data loss prevention scanning and configuration
- [Audit](/security/audit) -- audit logging, providers, and integrity verification
