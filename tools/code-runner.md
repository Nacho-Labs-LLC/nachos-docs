---
title: "Code Runner"
description: "Execute Python and JavaScript code in isolated sandboxed containers."
---

# Code Runner

The code runner executes code in isolated Docker containers with no network access. Each language runs in a separate container with its own resource limits.

## Languages

| Language | Container | Security Tier | NATS Topic |
|----------|-----------|---------------|------------|
| Python | `LANGUAGE=python` | RESTRICTED (3) | `nachos.tool.code_runner_python.request` |
| JavaScript | `LANGUAGE=javascript` | STANDARD (1) | `nachos.tool.code_runner_javascript.request` |

## Configuration

```toml
[tools.code_runner]
enabled = true
languages = ["python", "javascript"]
timeout = 30
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `code` | string | Yes | Code to execute |
| `timeout` | number | No | Timeout in seconds (1-30, default: 30) |
| `workdir` | string | No | Working directory, must be within `/tmp` |

## Resource limits

| Limit | Value |
|-------|-------|
| Timeout | 30 seconds max |
| Output buffer | ~10KB before truncation, killed at ~20KB |
| Memory | 512 MB per container |
| CPU | 1.0 per container |
| PIDs | 100 |
| Network | None (no egress) |
| Filesystem | Read-only (except `/tmp`) |

## Workspace access

Code runner containers mount the workspace directory as read-only at `/workspace`. This allows the assistant to read data files during code execution:

```python
# Read a CSV file from the workspace
import csv
with open('/workspace/data/report.csv') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

The `WORKSPACE` environment variable is set to `/workspace` for convenience.

## Python environment

```
PATH=/usr/local/bin:/usr/bin:/bin
PYTHONDONTWRITEBYTECODE=1
PYTHONUNBUFFERED=1
WORKSPACE=/workspace
```

Python includes the standard library. Additional packages must be installed within the execution (they do not persist across runs).

## What the assistant can do

- Write and execute scripts
- Process and transform data
- Read workspace files (read-only)
- Generate output and visualizations
- Install packages within the sandbox (pip, npm) -- non-persistent

## Gotchas

- **Timeout kills the process**: Long-running scripts are terminated at the timeout limit. The assistant sees a timeout error.
- **No persistent state**: Each execution starts fresh. Variables and installed packages do not persist across runs.
- **Memory limit is hard**: Exceeding memory kills the process immediately (OOM).
- **Python is RESTRICTED**: Python code execution requires security tier 3, which may require user approval depending on your security mode.
- **No network access**: Code runner containers have no egress. Use `web_fetch` or `web_search` tools separately for internet access.
- **Output truncation**: Large outputs are truncated to prevent token exhaustion.
