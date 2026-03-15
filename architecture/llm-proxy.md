---
title: "LLM Proxy"
description: "Provider abstraction for Claude, GPT, Ollama, Bedrock, and more."
---

# LLM Proxy

The LLM proxy normalizes different LLM provider APIs into a single internal interface. The gateway talks to the LLM proxy; the proxy talks to whichever provider you have configured.

## Supported providers

| Provider | Config value | API key env var | Notes |
|----------|-------------|-----------------|-------|
| Anthropic | `"anthropic"` | `ANTHROPIC_API_KEY` | Default provider |
| OpenAI | `"openai"` | `OPENAI_API_KEY` | |
| AWS Bedrock | `"bedrock"` | AWS credentials | Requires `region` |
| Ollama | `"ollama"` | (none -- local) | Requires `base_url` |
| Custom | `"custom"` | (varies) | Requires `base_url` |

## Configuration

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
max_tokens = 4096
temperature = 0.7
fallback_order = ["anthropic:claude-haiku"]
context_window = 200000   # Override auto-detected context window
```

### Retry and cooldown

```toml
[llm.retry]
attempts = 3
min_delay_ms = 1000
max_delay_ms = 30000
jitter = 0.1

[llm.cooldowns]
initial_seconds = 60
multiplier = 2
max_seconds = 3600
billing_initial_hours = 1
billing_max_hours = 24
```

## Failover

If the primary model fails (rate limit, timeout, API error), the proxy tries models in `fallback_order`:

```toml
fallback_order = ["anthropic:claude-haiku", "openai:gpt-4o-mini"]
```

Each entry is `provider:model`. The proxy tries them in order until one succeeds or all fail.

## Multiple auth profiles

Configure multiple API keys for the same provider (e.g., multiple Anthropic accounts):

```toml
[[llm.profiles]]
name = "primary"
provider = "anthropic"
api_key_env = "ANTHROPIC_API_KEY"

[[llm.profiles]]
name = "backup"
provider = "anthropic"
api_key_env = "ANTHROPIC_API_KEY_2"

profile_order = ["primary", "backup"]
```

The proxy uses profiles in order, falling back to the next if the current key is rate-limited or invalid.

## Ollama (local models)

For local inference with no external API calls:

```toml
[llm]
provider = "ollama"
model = "llama3.2"
base_url = "http://host.docker.internal:11434"
```

`host.docker.internal` resolves to the host machine from inside Docker. Ollama must be running on the host.

## Network

The LLM proxy joins `nachos-egress` to reach external LLM APIs, and `nachos-internal` to receive requests from the gateway via the bus.

## Gotchas

- **API keys are required**: The proxy fails to start if the configured provider's API key is missing from `.env`.
- **Ollama base_url**: Use `host.docker.internal`, not `localhost`. Inside a container, `localhost` refers to the container itself.
- **Token limits**: `max_tokens` is the response limit, not the context window. The proxy does not enforce input length.
- **Streaming**: Responses are streamed from the LLM through the proxy to the gateway to reduce time-to-first-token.
- **Bedrock**: Requires `region` in config and AWS credentials in the environment.
