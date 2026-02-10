---
title: "Commands"
description: "Full CLI command reference."
---

# Commands

## `nachos init`

Initialize a new Nachos project in the current directory.

```bash
nachos init [--defaults]
```

| Flag         | Description                                    |
|--------------|------------------------------------------------|
| `--defaults` | Skip prompts and use default configuration     |

Creates `nachos.toml`, `.env`, and `policies/` directory.

## `nachos up`

Start the stack. Pulls images, creates networks, and starts all enabled containers.

```bash
nachos up [--build] [--only <services>]
```

| Flag               | Description                                 |
|--------------------|---------------------------------------------|
| `--build`          | Force rebuild all container images           |
| `--only <list>`    | Start only specific services (comma-separated) |

Examples:

```bash
nachos up
nachos up --build
nachos up --only gateway,bus,slack
```

## `nachos down`

Stop and remove all stack containers and networks.

```bash
nachos down
```

## `nachos restart`

Stop and restart the stack. Use after config changes.

```bash
nachos restart
```

Equivalent to `nachos down && nachos up`.

## `nachos status`

Show the health status of all running containers.

```bash
nachos status
```

Output includes container name, status (healthy/unhealthy/starting), uptime, and port mappings.

## `nachos logs`

Tail logs from running services.

```bash
nachos logs [service]
```

| Argument    | Description                               |
|-------------|-------------------------------------------|
| `[service]` | Optional. Filter to a specific service.   |

Examples:

```bash
nachos logs           # All services
nachos logs gateway   # Gateway only
nachos logs slack     # Slack channel only
```

## `nachos config validate`

Validate `nachos.toml` for syntax errors, missing fields, and invalid values.

```bash
nachos config validate
```

Returns exit code 0 on success, 1 on failure with details.

## `nachos policy validate`

Validate policy files in `policies/` for YAML syntax and rule conflicts.

```bash
nachos policy validate
```

## `nachos add channel <name>`

Scaffold a new channel in `nachos.toml`.

```bash
nachos add channel <name>
```

Available channels: `slack`, `discord`, `telegram`

Example:

```bash
nachos add channel slack
```

Adds the `[channels.slack]` section to `nachos.toml` with default values. You still need to set tokens in `.env`.

## `nachos add tool <name>`

Scaffold a new tool in `nachos.toml`.

```bash
nachos add tool <name>
```

Available tools: `browser`, `filesystem`, `code-runner`

Example:

```bash
nachos add tool browser
```

Adds the `[tools.browser]` section to `nachos.toml` with default values.

## `nachos doctor`

Run diagnostic checks on the environment.

```bash
nachos doctor
```

Checks Docker availability, port conflicts, config validity, and service connectivity.
