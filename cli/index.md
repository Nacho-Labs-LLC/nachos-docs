---
title: "CLI"
description: "Overview of the Nachos CLI."
---

# CLI

The Nachos CLI manages your entire stack — initialization, lifecycle, config validation, module scaffolding, state management, and more.

## Install

```bash
pnpm add -g @nachos/cli
```

See [CLI Install](/install/cli) for details including shell completions.

## Usage

```bash
nachos <command> [options]
```

## Command groups

| Group | Commands | Purpose |
| ----- | -------- | ------- |
| **Lifecycle** | `init`, `up`, `down`, `restart` | Create and manage the stack |
| **Observe** | `status`, `logs`, `list` | Monitor running services |
| **Validate** | `validate`, `config validate`, `policy validate` | Check config and policy files |
| **Modules** | `add`, `remove` | Add or remove channels and tools |
| **State** | `memory`, `user-profile` | Manage memory and user profiles |
| **Subagents** | `subagents` | Spawn and inspect subagent runs |
| **Sandbox** | `sandbox` | Inspect and recreate tool sandbox config |
| **Auth** | `auth` | Configure LLM authentication |
| **System** | `doctor`, `debug`, `ui`, `open`, `completion` | Diagnostics and utilities |

## Command aliases

Frequently used commands have short aliases:

| Alias | Full command |
| ----- | ------------ |
| `nachos s` | `nachos status` |
| `nachos l` | `nachos logs` |
| `nachos r` | `nachos restart` |
| `nachos d` | `nachos down` |
| `nachos cfg` | `nachos config` |
| `nachos mem` | `nachos memory` |

## Global options

Every command accepts these flags:

| Flag | Short | Description |
| ---- | ----- | ----------- |
| `--json` | | Machine-readable JSON output |
| `--verbose` | | Debug-level logging |
| `--quiet` | `-q` | Suppress non-essential output |
| `--config <path>` | `-c` | Path to `nachos.toml` |
| `--no-input` | | Disable interactive prompts |
| `--no-color` | | Disable colored output |

## Typical workflow

```bash
nachos init               # Create nachos.toml, .env, policies/
# Edit nachos.toml and .env
nachos config validate    # Check for config errors
nachos up --wait          # Start the stack and wait for healthy
nachos status             # Verify all services are up
nachos logs gateway -f    # Tail gateway logs
# Make config changes
nachos restart            # Apply changes
nachos down               # Stop the stack
```

## Command reference

See [Commands](/cli/commands) for every command with all options and examples.
