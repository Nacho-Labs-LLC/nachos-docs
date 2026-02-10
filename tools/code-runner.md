---
title: "Code Runner"
description: "Execute code safely with guardrails."
---

# Code Runner

The code runner executes code in an isolated sandbox. The assistant can write and run scripts to answer questions, transform data, or generate outputs.

## Configuration

```toml
[tools.code_runner]
enabled = true
runtime = "sandboxed"
languages = ["python", "javascript", "typescript"]
timeout = 30
max_memory = "256MB"
```

| Key          | Type     | Default                                | Description                    |
|--------------|----------|----------------------------------------|--------------------------------|
| `enabled`    | boolean  | `false`                                | Enable the code runner         |
| `runtime`    | string   | `"sandboxed"`                          | `"sandboxed"` or `"native"`   |
| `languages`  | string[] | `["python", "javascript", "typescript"]` | Allowed languages            |
| `timeout`    | integer  | `30`                                   | Execution timeout in seconds   |
| `max_memory` | string   | `"256MB"`                              | Memory limit per execution     |

## Runtime modes

| Mode        | Description                                          | Security tier |
|-------------|------------------------------------------------------|---------------|
| `sandboxed` | Runs in an isolated container with no network access | 2 (Elevated)  |
| `native`    | Runs on the host. Requires `security.mode = "permissive"` | 3 (Restricted) |

Use `sandboxed` unless you have a specific reason not to. The sandbox has:

- No network access
- Read-only filesystem (except `/tmp`)
- Memory and CPU limits
- PID limits to prevent fork bombs

## Supported languages

- **Python** — includes standard library and common packages (numpy, pandas, requests)
- **JavaScript** — Node.js runtime
- **TypeScript** — compiled and run with tsx

Restrict to only what you need:

```toml
languages = ["python"]
```

## What the assistant can do

- Write and execute scripts
- Process data and generate output
- Install packages within the sandbox (pip, npm)
- Read files from the workspace (if filesystem tool is also enabled and paths overlap)

## Gotchas

- **Timeout kills the process**: Long-running scripts are terminated at the `timeout` limit. The assistant sees a timeout error.
- **No persistent state**: Each execution starts fresh. Variables and installed packages do not persist across runs.
- **Memory limit is hard**: Exceeding `max_memory` kills the process immediately (OOM).
- **Native mode is dangerous**: It gives the assistant arbitrary code execution on your host. Only use in trusted, single-user environments with `security.mode = "permissive"`.
