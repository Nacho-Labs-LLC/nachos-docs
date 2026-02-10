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
```

See [Tools](/tools/index) for per-tool configuration.

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
