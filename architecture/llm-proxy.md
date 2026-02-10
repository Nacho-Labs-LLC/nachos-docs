---
title: "LLM Proxy"
description: "Provider abstraction for Claude, GPT, Ollama, and more."
---

# LLM Proxy

The llm-proxy normalizes different LLM provider APIs into a single internal interface. The gateway talks to the llm-proxy; the llm-proxy talks to whichever LLM provider you've configured.

## Supported providers

| Provider   | Config value   | API key env var      |
|------------|----------------|----------------------|
| Anthropic  | `"anthropic"`  | `ANTHROPIC_API_KEY`  |
| OpenAI     | `"openai"`     | `OPENAI_API_KEY`     |
| Ollama     | `"ollama"`     | (none — local)       |
| Custom     | `"custom"`     | (varies)             |

## Configuration

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
max_tokens = 4096
temperature = 0.7
fallback_order = ["anthropic:claude-haiku"]
```

## Failover

If the primary model fails (rate limit, timeout, API error), the llm-proxy tries models in `fallback_order`:

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

The llm-proxy joins `nachos-egress` to reach external LLM APIs, and `nachos-internal` to receive requests from the gateway.

## Gotchas

- **API keys are required**: The proxy will fail to start if the configured provider's API key is missing from `.env`.
- **Ollama base_url**: Use `host.docker.internal`, not `localhost`. Inside a container, `localhost` refers to the container itself.
- **Token limits**: `max_tokens` is the response limit, not the context window. The proxy does not enforce input length — that's the LLM's responsibility.
- **Streaming**: Responses are streamed from the LLM through the proxy to the gateway to reduce time-to-first-token.
