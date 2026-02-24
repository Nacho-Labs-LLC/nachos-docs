---
title: "nachos.toml"
description: "Main configuration reference."
---

# nachos.toml

This is the single source of truth for your Nachos stack. It controls the LLM provider, channels, tools, security policies, and runtime settings.

## Minimal example

```toml
[nachos]
name = "my-assistant"
version = "1.0"

[llm]
provider = "anthropic"
model = "claude-sonnet-4-20250514"

[security]
mode = "standard"

[channels.webchat]
enabled = true
port = 8080
```

## Sections

### `[nachos]` — Project metadata

| Key       | Type   | Description            |
|-----------|--------|------------------------|
| `name`    | string | Name of your assistant |
| `version` | string | Config version         |

### `[llm]` — LLM provider

| Key              | Type     | Default       | Description                                |
|------------------|----------|---------------|--------------------------------------------|
| `provider`       | string   | `"anthropic"` | `"anthropic"`, `"openai"`, `"ollama"`, `"custom"` |
| `model`          | string   | —             | Model identifier                           |
| `fallback_order` | string[] | `[]`          | Fallback models if primary fails           |
| `max_tokens`     | integer  | `4096`        | Max response tokens                        |
| `temperature`    | float    | `0.7`         | 0.0 = deterministic, 1.0 = creative       |

For Ollama (local models):

```toml
[llm]
provider = "ollama"
model = "llama3.2"
base_url = "http://host.docker.internal:11434"
```

### `[channels.*]` — Messaging channels

Each channel has its own section. Common fields:

| Key       | Type    | Description                        |
|-----------|---------|------------------------------------|
| `enabled` | boolean | Whether the channel is active      |
| `token`   | string  | Bot token (use `${ENV_VAR}` syntax)|

See [Channels](/channels/index) for per-channel configuration.

### `[tools.*]` — Tool containers

Each tool has its own section:

```toml
[tools.browser]
enabled = true
allowed_domains = ["*"]
headless = true
timeout = 30

[tools.filesystem]
enabled = true
paths = ["./workspace"]
write = true
max_file_size = "10MB"

[tools.code_runner]
enabled = true
runtime = "sandboxed"
languages = ["python", "javascript", "typescript"]
timeout = 30
max_memory = "256MB"

[tools.github]
enabled = true
default_repo = "owner/repo"
token_env = "GH_TOKEN"
repo_allowlist = ["owner/repo"]

[tools.bitbucket]
enabled = true
default_workspace = "myworkspace"
auth_type = "app_password"
username_env = "BITBUCKET_USERNAME"
password_env = "BITBUCKET_APP_PASSWORD"
workspace_allowlist = ["myworkspace"]

[tools.web_search]
enabled = true
api_key = "${BRAVE_API_KEY}"
default_country = "US"
safe_search = "moderate"
max_results = 10

[tools.web_fetch_native]
enabled = true
timeout_ms = 10000
max_chars = 50000
domain_allowlist = ["docs.nachos.dev", "github.com"]

[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
entity_id = "default"
allowed_apps = ["gmail", "googlecalendar", "googledocs", "googlemeet", "googledrive", "linkedin"]
```

See [Tools](/tools/index) for per-tool configuration.

### `[scheduler]` — Cron scheduling

```toml
[scheduler]
enabled = true
timezone = "America/New_York"
check_interval_ms = 60000
```

See [Cron Scheduling](/tools/cron) for details.

### `[heartbeat]` — Proactive checks

```toml
[heartbeat]
enabled = true
intervalMinutes = 30
channel = "discord"
prompt = "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."
```

See [Heartbeat System](/tools/heartbeat) for details.

### `[security]` — Security mode and policies

| Key    | Type   | Default      | Description                                   |
|--------|--------|--------------|-----------------------------------------------|
| `mode` | string | `"standard"` | `"strict"`, `"standard"`, or `"permissive"`  |

See [Security](/security/index) for DLP, rate limits, and audit settings.

### `[runtime]` — Runtime settings

| Key             | Type   | Default     | Description                    |
|-----------------|--------|-------------|--------------------------------|
| `state_dir`     | string | `"./state"` | Persistent state storage       |
| `log_level`     | string | `"info"`    | `"debug"`, `"info"`, `"warn"`, `"error"` |
| `log_format`    | string | `"pretty"`  | `"pretty"` or `"json"`       |
| `redis_url`     | string | —           | Redis connection for sessions  |

### `[assistant]` — Personality

```toml
[assistant]
name = "Nachos"
system_prompt = """
You are a helpful AI assistant running in the Nachos framework.
"""
```

## Applying changes

```bash
nachos config validate
nachos restart
```
