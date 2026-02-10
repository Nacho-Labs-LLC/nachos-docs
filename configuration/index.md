---
title: "Configuration"
description: "Core configuration files and patterns."
---

# Configuration

Nachos uses two configuration sources:

- **`nachos.toml`** — the single source of truth for your stack (LLM provider, channels, tools, security, runtime)
- **`.env`** — secrets and API keys, kept out of source control

## How configuration is applied

1. Edit `nachos.toml` or `.env`
2. Run `nachos restart` (or `nachos down && nachos up`)
3. The CLI generates Docker Compose manifests from your config and starts the stack

## Validate before applying

```bash
nachos config validate
```

This checks for syntax errors, missing required fields, and invalid option values. Always validate after editing `nachos.toml`.

## Pages

- [nachos.toml](/configuration/nachos-toml) — full configuration reference
- [Environment variables](/configuration/env-vars) — secrets and runtime overrides
