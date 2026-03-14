---
title: "Commands"
description: "Full CLI command reference."
---

# Commands

## Global options

These options apply to every command.

| Flag | Short | Description |
| ---- | ----- | ----------- |
| `--json` | | Output as JSON |
| `--verbose` | | Enable verbose/debug output |
| `--quiet` | `-q` | Suppress non-essential output |
| `--config <path>` | `-c` | Path to `nachos.toml` |
| `--no-input` | | Disable interactive prompts |
| `--no-color` | | Disable colored output |

---

## Stack lifecycle

### `nachos init`

Initialize a new Nachos project in the current directory.

```bash
nachos init [options]
```

| Flag | Description |
| ---- | ----------- |
| `--defaults` | Skip prompts and use defaults |
| `--force` | Overwrite existing configuration |

```bash
nachos init
nachos init --defaults
nachos init --force
```

Creates `nachos.toml`, `.env`, and `policies/`.

---

### `nachos up`

Start the stack. Pulls images, creates networks, and starts all enabled containers.

```bash
nachos up [options]
```

| Flag | Description |
| ---- | ----------- |
| `--build` | Force rebuild all images before starting |
| `--wait` | Block until all services pass health checks |
| `--only <services>` | Start only listed services (comma-separated) |
| `--timeout <seconds>` | Health-check wait timeout (default: 60) |

```bash
nachos up
nachos up --wait --timeout 90
nachos up --only gateway,bus
nachos up --build --wait
```

---

### `nachos down`

Stop and remove all stack containers and networks. **Alias: `d`**

```bash
nachos down [options]
nachos d [options]
```

| Flag | Description |
| ---- | ----------- |
| `--volumes` | Also remove named volumes (destroys persisted data) |
| `--force` | Skip confirmation when `--volumes` is set |

```bash
nachos down
nachos down --volumes --force
```

---

### `nachos restart`

Stop and restart the stack. Use after config changes. **Alias: `r`**

```bash
nachos restart [options]
nachos r [options]
```

| Flag | Description |
| ---- | ----------- |
| `--build` | Rebuild images before starting |
| `--wait` | Block until all services are healthy |

```bash
nachos restart
nachos restart --build --wait
```

---

## Observability

### `nachos status`

Show health status for all running containers. **Alias: `s`**

```bash
nachos status
nachos s
nachos s --json
```

Output includes container name, status (healthy/unhealthy/starting), uptime, and port mappings.

---

### `nachos logs`

Tail logs from running services. **Alias: `l`**

```bash
nachos logs [service] [options]
nachos l [service] [options]
```

| Flag | Short | Description |
| ---- | ----- | ----------- |
| `--follow` | `-f` | Follow log output (stream) |
| `--tail <lines>` | | Lines from end to show (default: 50) |
| `--timestamps` | `-t` | Show log timestamps |

```bash
nachos logs               # All services
nachos logs gateway -f    # Gateway, streaming
nachos l bus --tail 200 --timestamps
```

---

### `nachos list`

List all configured modules (channels, tools) from `nachos.toml`.

```bash
nachos list
nachos list --json
```

---

## Validation

### `nachos validate`

Run all checks at once: config, policy, and doctor health checks.

```bash
nachos validate
nachos validate --json
```

Equivalent to running `nachos config validate`, `nachos policy validate`, and `nachos doctor` together. Returns a non-zero exit code if anything fails.

---

### `nachos config validate`

Validate `nachos.toml` for syntax errors, missing required fields, and invalid values. **Alias: `nachos cfg validate`**

```bash
nachos config validate
nachos cfg validate --json
```

Exit code 0 on success, 1 on failure.

---

### `nachos policy validate`

Validate policy files in `policies/` for YAML syntax and rule conflicts.

```bash
nachos policy validate
nachos policy validate --json
```

---

## Modules

### `nachos add`

Add a channel or tool to the configuration. Run with `--interactive` for a guided selection prompt.

```bash
nachos add --interactive
nachos add channel <name> [options]
nachos add tool <name> [options]
```

**Interactive mode:**

```bash
nachos add --interactive
# Prompts for type → channel or tool → specific name
```

**`nachos add channel <name>`**

Available: `slack`, `discord`, `telegram`, `whatsapp`, `webchat`

| Flag | Description |
| ---- | ----------- |
| `--no-enabled` | Add section but leave channel disabled |
| `--mode <mode>` | Connection mode: `socket` or `http` (Slack) |
| `--port <port>` | Port number (webchat) |

```bash
nachos add channel discord
nachos add channel slack --mode socket
nachos add channel webchat --port 3000
```

**`nachos add tool <name>`**

Available: `browser`, `filesystem`, `code-runner`, `shell`, `web_search`, `bootstrap`, `claude_code_mcp`

| Flag | Description |
| ---- | ----------- |
| `--no-enabled` | Add section but leave tool disabled |
| `--paths <paths>` | Allowed paths, comma-separated (filesystem) |
| `--domains <domains>` | Allowed domains, comma-separated (browser) |
| `--languages <langs>` | Enabled languages, comma-separated (code-runner) |
| `--timeout <seconds>` | Timeout in seconds (browser, code-runner) |
| `--memory <size>` | Max memory (code-runner) |

```bash
nachos add tool browser
nachos add tool filesystem --paths ./workspace,./data
nachos add tool code-runner --languages python,typescript --timeout 30
```

---

### `nachos remove`

Remove a module from `nachos.toml`.

```bash
nachos remove <type> <name> [options]
```

| Flag | Description |
| ---- | ----------- |
| `--dry-run` | Show what would be removed without changing anything |
| `--force` | Skip confirmation prompt |

```bash
nachos remove channel discord
nachos remove tool browser --dry-run
nachos remove channel slack --force
```

---

## State management

### `nachos memory`

Manage the memory store for an agent. **Alias: `mem`**

```bash
nachos memory <subcommand> [options]
nachos mem <subcommand> [options]
```

**`nachos memory query`**

Search memory entries and facts.

| Flag | Description |
| ---- | ----------- |
| `--agent-id <id>` | Agent ID (required) |
| `--text <text>` | Full-text search query |
| `--kinds <kinds>` | Filter by kind: `summary`, `preference`, `fact`, `decision`, `task`, `issue` (comma-separated) |
| `--tags <tags>` | Filter by tags (comma-separated) |
| `--limit <n>` | Max results |
| `--offset <n>` | Pagination offset |
| `--user-id <id>` | User ID for audit context |
| `--session-id <id>` | Session ID for audit context |

```bash
nachos memory query --agent-id my-bot --text "breakfast" --limit 5
nachos mem query --agent-id my-bot --kinds preference,fact --json
```

**`nachos memory append-entry`**

Add a new memory entry.

| Flag | Description |
| ---- | ----------- |
| `--agent-id <id>` | Agent ID (required) |
| `--kind <kind>` | Entry kind (required): `summary`, `preference`, `fact`, `decision`, `task`, `issue` |
| `--content <text>` | Entry content (required) |
| `--tags <tags>` | Comma-separated tags |
| `--confidence <score>` | Confidence score 0–1 |
| `--expires-at <ts>` | Expiration timestamp (ISO 8601) |
| `--source <label>` | Provenance source label |
| `--provenance-file <path>` | Path to provenance JSON file |

```bash
nachos memory append-entry \
  --agent-id my-bot \
  --kind preference \
  --content "User loves breakfast tacos" \
  --tags food,preferences
```

**`nachos memory append-fact`**

Add an RDF-style fact triple.

| Flag | Description |
| ---- | ----------- |
| `--agent-id <id>` | Agent ID (required) |
| `--subject <text>` | Fact subject (required) |
| `--predicate <text>` | Fact predicate (required) |
| `--object <text>` | Fact object (required) |
| `--confidence <score>` | Confidence score 0–1 |
| `--source-entry-id <id>` | Source entry ID |

```bash
nachos memory append-fact \
  --agent-id my-bot \
  --subject "User" \
  --predicate "prefers" \
  --object "tacos"
```

**`nachos memory delete`**

Delete a memory entry by ID.

| Flag | Description |
| ---- | ----------- |
| `--agent-id <id>` | Agent ID (required) |
| `--id <entryId>` | Entry ID to delete (required) |

```bash
nachos memory delete --agent-id my-bot --id entry_abc123
```

---

### `nachos user-profile`

Manage per-user profiles for an agent.

```bash
nachos user-profile <subcommand> [options]
```

**`nachos user-profile get`**

```bash
nachos user-profile get --agent-id my-bot --user-id user-1
nachos user-profile get --agent-id my-bot --user-id user-1 --json
```

**`nachos user-profile set`**

| Flag | Description |
| ---- | ----------- |
| `--agent-id <id>` | Agent ID (required) |
| `--user-id <id>` | User ID (required) |
| `--profile <text>` | Profile text |
| `--file <path>` | Read profile text from a file |
| `--source <label>` | Source label |

```bash
nachos user-profile set \
  --agent-id my-bot \
  --user-id user-1 \
  --profile "Prefers short, direct replies"

nachos user-profile set \
  --agent-id my-bot \
  --user-id user-1 \
  --file ./profiles/alice.txt
```

**`nachos user-profile delete`**

```bash
nachos user-profile delete --agent-id my-bot --user-id user-1
```

---

## Subagents

### `nachos subagents`

Manage subagent runs dispatched by the gateway.

```bash
nachos subagents <subcommand> [options]
```

**`nachos subagents spawn <task>`**

Spawn a new subagent run.

| Flag | Description |
| ---- | ----------- |
| `--label <label>` | Human-readable run label |
| `--profile <profile>` | Tool profile to apply |
| `--agent-id <id>` | Override the subagent ID |
| `--model <model>` | Override the model |
| `--thinking <hint>` | Thinking mode hint |
| `--timeout <seconds>` | Run timeout |
| `--cleanup <mode>` | Post-run cleanup: `delete` or `keep` |

```bash
nachos subagents spawn "Summarize this repo" --label summary
nachos subagents spawn "Fix the failing tests" --timeout 300 --cleanup keep
```

**`nachos subagents list`**

```bash
nachos subagents list
nachos subagents list --limit 10 --json
```

**`nachos subagents info <runId>`**

```bash
nachos subagents info run_abc123
nachos subagents info run_abc123 --json
```

**`nachos subagents stop <runId>`**

Stop a queued run before it starts.

```bash
nachos subagents stop run_abc123
```

**`nachos subagents log <runId>`**

Show the message log for a run.

```bash
nachos subagents log run_abc123
nachos subagents log run_abc123 --limit 20
```

**`nachos subagents files list <runId>`**

List files in the subagent's workspace.

| Flag | Description |
| ---- | ----------- |
| `--path <path>` | Subdirectory to list |
| `--recursive` | List recursively |
| `--limit <n>` | Max entries |

```bash
nachos subagents files list run_abc123
nachos subagents files list run_abc123 --path src --recursive
```

**`nachos subagents files get <runId>`**

Fetch a file from the subagent's workspace.

| Flag | Description |
| ---- | ----------- |
| `--path <path>` | File path (required) |
| `--max-bytes <n>` | Max bytes to read |

```bash
nachos subagents files get run_abc123 --path output/report.md
```

---

## Sandbox

### `nachos sandbox`

Inspect and manage the tool sandbox configuration.

```bash
nachos sandbox <subcommand>
```

**`nachos sandbox explain`**

Describe the current sandbox configuration and what it allows.

```bash
nachos sandbox explain
```

**`nachos sandbox list`**

List current sandbox status and active restrictions.

```bash
nachos sandbox list
nachos sandbox list --json
```

**`nachos sandbox recreate`**

Recreate the sandbox configuration from current `nachos.toml`.

| Flag | Description |
| ---- | ----------- |
| `--force` | Skip confirmation prompt |

```bash
nachos sandbox recreate
nachos sandbox recreate --force
```

---

## Plugins

### `nachos plugin add`

Register a plugin from a source.

```bash
nachos plugin add <source> [options]
```

| Flag | Description |
| ---- | ----------- |
| `--source-type <type>` | Source type |
| `--name <name>` | Plugin name |
| `--enable` | Enable plugin after adding |
| `--dry-run` | Show what would be added without changing config |

```bash
nachos plugin add github:user/nachos-plugin-weather
nachos plugin add ./local-plugin --enable
nachos plugin add github:user/plugin --dry-run
```

---

### `nachos plugin remove`

Remove a registered plugin.

```bash
nachos plugin remove <name> [options]
```

| Flag | Description |
| ---- | ----------- |
| `--force` | Skip confirmation prompt |
| `--keep-config` | Keep plugin config section in nachos.toml |
| `--dry-run` | Show what would be removed |

```bash
nachos plugin remove weather
nachos plugin remove weather --force --keep-config
```

---

### `nachos plugin list`

List all registered plugins.

```bash
nachos plugin list
nachos plugin list --json
```

---

## Authentication

### `nachos auth setup-token`

Configure an Anthropic setup-token for LLM authentication.

```bash
nachos auth setup-token [options]
```

| Flag | Description |
| ---- | ----------- |
| `--provider <provider>` | Provider (default: `anthropic`) |
| `--profile <name>` | Profile name (default: `anthropic-subscription`) |
| `--env <name>` | Environment variable name (default: `ANTHROPIC_SETUP_TOKEN`) |
| `--token <token>` | Paste the token non-interactively |
| `--append` | Append to profile order instead of prepending |
| `--write-env` | Write token to `.env` |

```bash
nachos auth setup-token
nachos auth setup-token --token sk_... --write-env
```

---

## System

### `nachos ui`

Open the Admin UI in the browser.

```bash
nachos ui [options]
```

| Flag | Short | Description |
| ---- | ----- | ----------- |
| `--port <port>` | `-p` | Admin UI port (default: 8082) |

```bash
nachos ui
nachos ui --port 8082
```

---

### `nachos open <service>`

Open a service endpoint in the browser.

Available services: `admin`, `webchat`, `gateway`, `nats`, `docs`

```bash
nachos open admin
nachos open webchat
nachos open nats
nachos open docs
nachos open gateway --port 8080
```

---

### `nachos doctor`

Run diagnostic health checks on the environment.

```bash
nachos doctor
nachos doctor --json
```

Checks Docker availability, port conflicts, config validity, dependency versions, disk space, and environment variables.

---

### `nachos debug`

Print detailed debug information about the current environment and config.

```bash
nachos debug
nachos debug --json
```

---

### `nachos completion <shell>`

Generate a shell completion script.

Available shells: `bash`, `zsh`, `fish`, `powershell`

```bash
# Bash (Linux)
nachos completion bash | sudo tee /etc/bash_completion.d/nachos > /dev/null

# Bash (macOS with Homebrew)
nachos completion bash > "$(brew --prefix)/etc/bash_completion.d/nachos"

# Zsh
mkdir -p ~/.zsh/completion
nachos completion zsh > ~/.zsh/completion/_nachos
echo 'fpath=(~/.zsh/completion $fpath)' >> ~/.zshrc

# Fish
nachos completion fish > ~/.config/fish/completions/nachos.fish

# PowerShell
nachos completion powershell | Out-String | Invoke-Expression
```

Then restart your shell or source your config file.

---

## Command aliases

| Alias | Full command |
| ----- | ------------ |
| `nachos s` | `nachos status` |
| `nachos l` | `nachos logs` |
| `nachos r` | `nachos restart` |
| `nachos d` | `nachos down` |
| `nachos cfg` | `nachos config` |
| `nachos mem` | `nachos memory` |
