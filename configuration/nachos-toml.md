---
title: "nachos.toml"
description: "Complete configuration reference for the Nachos stack."
---

# nachos.toml

This is the single source of truth for your Nachos stack. It controls the LLM provider, channels, tools, security policies, runtime settings, and more.

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
```

## Sections

### `[nachos]` -- Project metadata

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `name` | string | Yes | Name of your assistant |
| `version` | string | Yes | Config version |

### `[llm]` -- LLM provider

```toml
[llm]
provider = "anthropic"                    # "anthropic" | "openai" | "ollama" | "bedrock" | "custom"
model = "claude-sonnet-4-20250514"
fallback_order = ["anthropic:claude-haiku"]
max_tokens = 4096                         # 1 - 1,000,000
temperature = 0.7                         # 0.0 - 2.0
base_url = "http://localhost:11434"       # For ollama/custom
region = "us-east-1"                      # For bedrock
context_window = 200000                   # Override auto-detected context window

# Multi-profile auth (optional)
[[llm.profiles]]
name = "anthropic-primary"
provider = "anthropic"
api_key_env = "ANTHROPIC_API_KEY"

profile_order = ["anthropic-primary"]

[llm.retry]
attempts = 3
min_delay_ms = 1000
max_delay_ms = 30000
jitter = 0.1

[llm.cooldowns]
initial_seconds = 60
multiplier = 2
max_seconds = 3600
```

### `[channels.*]` -- Messaging channels

Each channel has its own section. See [Channels](/channels/index) for per-channel details.

```toml
[channels.slack]
enabled = true
mode = "socket"
app_token = "${SLACK_APP_TOKEN}"
bot_token = "${SLACK_BOT_TOKEN}"

[channels.discord]
enabled = true
token = "${DISCORD_BOT_TOKEN}"

[channels.telegram]
enabled = true
token = "${TELEGRAM_BOT_TOKEN}"

[channels.whatsapp]
enabled = true
token = "${WHATSAPP_TOKEN}"
phone_number_id = "${WHATSAPP_PHONE_NUMBER_ID}"
verify_token = "${WHATSAPP_VERIFY_TOKEN}"
app_secret = "${WHATSAPP_APP_SECRET}"

[channels.matrix]
enabled = true
homeserver_url = "https://matrix.org"
access_token = "${MATRIX_ACCESS_TOKEN}"
user_id = "@bot:matrix.org"
device_id = "NACHOS_BOT"
```

### `[tools.*]` -- Tool containers and integrations

```toml
[tools.browser]
enabled = true

[tools.filesystem]
enabled = true
paths = ["./workspace"]
write = true
max_file_size = "10MB"

[tools.code_runner]
enabled = true
languages = ["python", "javascript"]
timeout = 30

[tools.agent_exec]
enabled = true
max_concurrent = 2
default_timeout = 300
max_timeout = 1800

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

[tools.web_fetch]
enabled = true
timeout_ms = 10000
max_chars = 50000
domain_allowlist = []

[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
entity_id = "default"
allowed_apps = ["gmail", "googlecalendar", "googledocs"]
```

See [Tools](/tools/index) for per-tool configuration.

### `[security]` -- Security mode and policies

```toml
[security]
mode = "standard"                         # "strict" | "standard" | "permissive"
# i_understand_the_risks = true           # Required for permissive mode

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
batch_size = 100
flush_interval_ms = 5000
```

See [Security](/security/index) for details.

### `[runtime]` -- Runtime settings

```toml
[runtime]
state_dir = "./state"
config_dir = "./config"
workspace_dir = "./workspace"
log_level = "info"                        # "debug" | "info" | "warn" | "error"
log_format = "pretty"                     # "pretty" | "json"
redis_url = "redis://localhost:6379"
gateway_streaming_passthrough = false

[runtime.resources]
memory = "512MB"
cpus = 0.5
pids_limit = 100

[runtime.context_management.sliding_window]
enabled = true
mode = "token-based"                      # "token-based" | "message-based" | "hybrid"

[runtime.context_management.commands]
enabled = true
reset_triggers = ["/new", "/reset"]
context_triggers = ["/context"]

[runtime.state.sessions]
provider = "sqlite"                       # "sqlite" | "postgres"

[runtime.state.semantic]
provider = "local"                        # "local" | "qdrant"

[runtime.subagents]
enabled = false
max_concurrent = 1

[runtime.sandbox]
mode = "off"                              # "off" | "non-main" | "all"
```

### `[assistant]` -- Personality

```toml
[assistant]
name = "Nachos"
system_prompt = """
You are a helpful AI assistant running in the Nachos framework.
"""
```

### `[skills]` -- Skill-backed CLI tools

```toml
[skills]
allow = ["goplaces", "gifgrep"]
deny = ["gog"]
hot_reload = true
debounce_ms = 500
```

### `[scheduler]` -- Cron scheduling

```toml
[scheduler]
enabled = true
timezone = "America/New_York"
check_interval_ms = 60000
```

### `[heartbeat]` -- Proactive checks

```toml
[heartbeat]
enabled = true
intervalMinutes = 30
channel = "discord"
prompt = "Read HEARTBEAT.md if it exists. Follow it strictly."
```

### `[admin]` -- Admin UI

```toml
[admin]
enabled = true
port = 8082
```

### `[gateway.subagent]` -- Subagent orchestration

```toml
[gateway.subagent]
mode = "host"
max_concurrent = 2
default_timeout_seconds = 300

[gateway.subagent.models]
haiku = "anthropic.claude-haiku-4-5-20251001-v1:0"
sonnet = "anthropic.claude-sonnet-4-6"
opus = "anthropic.claude-opus-4-6-v1"
auto_select = true
default_model = "sonnet"

[gateway.subagent.streaming]
enabled = true
deliver_to_requester = true
chunk_throttle_ms = 500

[gateway.subagent.announce]
enabled = true
```

## Environment variable references

Use `${VAR_NAME}` syntax to reference environment variables:

```toml
[channels.slack]
bot_token = "${SLACK_BOT_TOKEN}"
```

At startup, Nachos resolves these from `.env` or the host environment. Host variables take precedence over `.env`.

## Applying changes

```bash
nachos config validate    # Check for errors
nachos restart            # Apply changes
```
