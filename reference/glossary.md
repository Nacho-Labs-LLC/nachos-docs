---
title: "Glossary"
description: "Key terms used in Nachos."
---

# Glossary

**Bus**
NATS-based message broker that handles all inter-container communication. Every message between services flows through the bus.

**Channel**
A messaging platform integration (Slack, Discord, Telegram, webchat). Each channel runs as a separate Docker container that connects users to the gateway via the bus.

**DLP (Data Loss Prevention)**
Security feature that scans outbound content for sensitive data patterns (credit cards, API keys, SSNs) and applies configurable actions (block, warn, redact, audit).

**Gateway**
The central control plane. Enforces security policies, manages sessions, routes tool calls, and handles audit logging. All messages and actions flow through the gateway.

**LLM Proxy**
Service that abstracts LLM provider APIs (Anthropic, OpenAI, Ollama) into a uniform internal interface. Handles failover between providers and auth profile rotation.

**Manifest**
A `manifest.json` file in each channel or tool container that declares its capabilities, required secrets, and network access needs. Used by the CLI to generate Docker Compose configuration.

**nachos.toml**
The single configuration file for the entire stack. Controls LLM provider, channels, tools, security mode, runtime settings, and assistant personality.

**Pairing**
Authentication mechanism in `standard` security mode where users must provide a token before they can DM the assistant. Set via `NACHOS_PAIRING_TOKEN` in `.env`.

**Policy**
A YAML rule in the `policies/` directory that controls what actions are allowed or denied. Evaluated by the gateway's policy engine at runtime. Policies hot-reload on file change.

**Security mode**
One of three presets (`strict`, `standard`, `permissive`) that control the default behavior of tools, channel access, and audit logging. Set via `security.mode` in `nachos.toml`.

**Security tier**
A numeric level (0–4) assigned to each tool that determines its default policy per security mode. Lower tiers are safer. Tiers: 0 (Safe), 1 (Standard), 2 (Elevated), 3 (Restricted), 4 (Dangerous).

**Session**
A conversation context between a user and the assistant, stored in Redis. Includes message history and state. Expires after a configurable TTL.

**Skill**
A SKILL.md file that documents a CLI tool's capabilities. Loaded into the LLM prompt so it knows how to use the tool. The gateway's shell tool executes the CLI command when called.

**Tool**
A capability container that gives the assistant abilities beyond text generation (browser, filesystem, code runner). Each tool runs in its own Docker container with explicit permissions.
