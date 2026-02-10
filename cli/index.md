---
title: "CLI"
description: "Overview of the Nachos CLI."
---

# CLI

The Nachos CLI is the primary interface for managing your stack. It handles initialization, lifecycle management, configuration validation, and module scaffolding.

## Install

```bash
pnpm add -g @nachos/cli
```

See [CLI Install](/install/cli) for details.

## Usage

```bash
nachos <command> [options]
```

## Command groups

| Group          | Commands                          | Purpose                         |
|----------------|-----------------------------------|---------------------------------|
| **Lifecycle**  | `init`, `up`, `down`, `restart`   | Create and manage the stack     |
| **Observe**    | `status`, `logs`                  | Monitor running services        |
| **Validate**   | `config validate`, `policy validate` | Check config and policy files |
| **Scaffold**   | `add channel`, `add tool`         | Add new modules to the stack    |

## Global options

```bash
nachos --version    # Print CLI version
nachos --help       # Show help for any command
nachos <cmd> --help # Show help for a specific command
```

## Typical workflow

```bash
nachos init                # Create nachos.toml, .env, policies/
# Edit nachos.toml and .env
nachos config validate     # Check for errors
nachos up                  # Start the stack
nachos status              # Verify health
nachos logs                # Tail service logs
# Make config changes
nachos restart             # Apply changes
nachos down                # Stop the stack
```

## Command reference

See [Commands](/cli/commands) for the full list with options and examples.
