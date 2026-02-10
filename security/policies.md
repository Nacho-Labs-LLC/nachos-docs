---
title: "Policies"
description: "Policy engine and allowlist patterns."
---

# Policies

Policies control what the assistant can do — which tools it can call, which users can interact with it, and what network access containers get. Policies are YAML files in the `policies/` directory, evaluated by the gateway at runtime.

## How policies work

1. A user sends a message or the LLM requests a tool call
2. The gateway's policy evaluator checks the action against loaded policies
3. If the action is allowed, it proceeds. If denied, the user sees an error.

Policies hot-reload on file change — no restart required.

## Policy file structure

```yaml
# policies/default.yaml
rules:
  - action: "tool.call"
    tool: "browser"
    effect: "allow"
    conditions:
      security_mode: ["standard", "permissive"]

  - action: "tool.call"
    tool: "shell"
    effect: "deny"
    conditions:
      security_mode: ["strict", "standard"]

  - action: "dm.initiate"
    effect: "allow"
    conditions:
      user_allowlist: ["U123", "U456"]
```

## User allowlists

Control who can DM the assistant per channel:

```toml
# nachos.toml
[[channels.slack.servers]]
id = "T123456"
channel_ids = ["C111", "C222"]
user_allowlist = ["U123", "U456"]
```

Users not on the allowlist are silently ignored in `strict` mode, or prompted for a pairing token in `standard` mode.

## Tool security tiers

Tools are assigned security tiers that determine their default policy:

| Tier | Level      | Examples                  | Default in standard mode |
|------|------------|---------------------------|--------------------------|
| 0    | Safe       | web_fetch, goplaces       | Allowed                  |
| 1    | Standard   | browser, filesystem (ro)  | Allowed                  |
| 2    | Elevated   | filesystem (rw), code_runner | Allowed with audit    |
| 3    | Restricted | shell                     | Denied                   |
| 4    | Dangerous  | —                         | Denied                   |

Override per-tool defaults in `nachos.toml`:

```toml
[tools.overrides.goplaces]
security_tier = 2
require_approval = true
rate_limit_per_minute = 10
```

## Validating policies

```bash
nachos policy validate
```

This checks YAML syntax, unknown action types, and conflicting rules.
