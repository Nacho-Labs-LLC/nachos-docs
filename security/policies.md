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

| Tier | Level      | Examples                                  | Default in standard mode |
|------|------------|-------------------------------------------|--------------------------|
| 0    | Safe       | web_search, web_fetch_native, goplaces    | Allowed                  |
| 1    | Standard   | browser, filesystem (ro), github, bitbucket | Allowed                  |
| 2    | Elevated   | filesystem (rw), code_runner, composio    | Allowed with audit       |
| 3    | Restricted | shell, cron (modify others' jobs)         | Denied                   |
| 4    | Dangerous  | —                                         | Denied                   |

Override per-tool defaults in `nachos.toml`:

```toml
[tools.overrides.github]
security_tier = 2
require_approval = true
rate_limit_per_minute = 10
```

## New Tool Policies

### GitHub & Bitbucket

Repository and workspace allowlisting:

```yaml
rules:
  - action: "tool.call"
    tool: "github"
    effect: "allow"
    conditions:
      security_mode: ["standard", "permissive"]
      # Enforced via repo_allowlist in nachos.toml
```

**Configuration:**

```toml
[tools.github]
repo_allowlist = ["myorg/backend", "myorg/frontend"]

[tools.bitbucket]
workspace_allowlist = ["myworkspace"]
```

### Web Search & Fetch

Domain restrictions for web_fetch_native:

```toml
[tools.web_fetch_native]
domain_allowlist = ["docs.nachos.dev", "github.com", "*.wikipedia.org"]
```

Safe search enforcement for web_search:

```toml
[tools.web_search]
safe_search = "moderate"  # or "strict"
```

### Composio

App-level restrictions:

```toml
[tools.composio]
allowed_apps = ["gmail", "googlecalendar"]  # Limit to specific apps
```

### Cron & Heartbeat

User isolation — users can only manage their own jobs:

```yaml
rules:
  - action: "tool.call"
    tool: "nachos_cron_add"
    effect: "allow"
    conditions:
      security_mode: ["standard", "permissive"]
      # Ownership enforced automatically

  - action: "tool.call"
    tool: "nachos_cron_remove"
    effect: "allow"
    conditions:
      # Can only delete own jobs
```

System jobs (heartbeat) are protected from user modification.

## Validating policies

```bash
nachos policy validate
```

This checks YAML syntax, unknown action types, and conflicting rules.
