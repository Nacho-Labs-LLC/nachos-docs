---
title: "Tools"
description: "Tooling and capabilities."
---

# Tools

Tools give the assistant capabilities beyond text generation — browsing the web, reading files, running code. Each tool runs in its own Docker container with explicit permissions defined in its manifest.

## Available tools

| Tool         | Description                          | Security tier | Config section         |
|--------------|--------------------------------------|---------------|------------------------|
| Browser      | Web navigation and content extraction | 1 (Standard) | `[tools.browser]`      |
| Filesystem   | Read/write files in workspace        | 1–2           | `[tools.filesystem]`   |
| Code Runner  | Execute code in a sandbox            | 2 (Elevated)  | `[tools.code_runner]`  |

## Enabling a tool

Set `enabled = true` in the tool's config section:

```toml
[tools.browser]
enabled = true
```

Or use the CLI:

```bash
nachos add tool browser
```

Then apply:

```bash
nachos restart
```

## How tools work

1. The LLM decides to use a tool during a conversation
2. The gateway checks the tool call against security policies
3. If allowed, the request is routed over the bus to the tool container
4. The tool executes and returns results to the LLM via the gateway

## Tool security

Tools are governed by [security policies](/security/policies). Each tool has a security tier that determines its default behavior per security mode:

- **Strict mode**: All tools disabled unless explicitly allowed in policy
- **Standard mode**: Tier 0–2 tools allowed, tier 3+ denied
- **Permissive mode**: All tools allowed

Override defaults per tool:

```toml
[tools.overrides.browser]
require_approval = true
rate_limit_per_minute = 10
```

## Tool guides

- [Browser](/tools/browser)
- [Filesystem](/tools/filesystem)
- [Code Runner](/tools/code-runner)
