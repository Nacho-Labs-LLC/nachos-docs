---
title: "Tools"
description: "Capabilities beyond text generation: browsing, files, code, search, and more."
---

# Tools

Tools give the assistant capabilities beyond text generation -- browsing the web, reading files, running code, managing GitHub repos, and executing autonomous tasks. Tools are divided into container-based (isolated Docker containers) and gateway-local (running in the gateway process).

## Tool overview

| Tool | Type | Security Tier | Description |
|------|------|---------------|-------------|
| [Filesystem](/tools/filesystem) | Container | 0 (read) / 2 (write) | Read, write, edit, and patch files |
| [Code Runner](/tools/code-runner) | Container | 1-3 | Execute Python and JavaScript in a sandbox |
| [Browser](/tools/browser) | Gateway-local | 1 (Standard) | Playwright-based browser automation |
| [Shell / Exec](/tools/shell) | Gateway-local | Varies | Execute allowlisted CLI commands |
| [Agent Exec](/tools/agent-exec) | Gateway-local | 2 (Elevated) | Launch Claude Code as autonomous subprocess |
| [Web Fetch](/tools/web-fetch) | Gateway-local | 0 (Safe) | Fetch and extract web page content |
| [Web Search](/tools/web-search) | Gateway-local | 0 (Safe) | Search the web via Brave Search API |
| [Memory](/tools/memory) | Gateway-local | 0-2 | Search, read, write, and delete memories |
| [GitHub](/tools/github) | Gateway-local | Varies | Issues, PRs, workflows via `gh` CLI |
| [Bitbucket](/tools/bitbucket) | Gateway-local | Varies | Repos, PRs, issues via REST API |
| [Cron](/tools/cron) | Gateway-local | N/A | Scheduled job management |
| [Bootstrap](/tools/bootstrap) | Gateway-local | 0 (Safe) | Identity and user profile configuration |

## Security tiers

Every tool is assigned a security tier that determines policy enforcement and approval requirements:

| Tier | Name | Description | Examples | Approval required |
|------|------|-------------|---------|-------------------|
| 0 | **SAFE** | Read-only, no side effects | `filesystem_read`, `memory_search`, `web_search` | No |
| 1 | **STANDARD** | Sandboxed operations | `browser`, `code_runner_javascript` | No |
| 2 | **ELEVATED** | Write access, requires policy | `filesystem_write`, `agent_exec`, `memory_write` | No |
| 3 | **RESTRICTED** | Requires explicit user approval | `copilot` | Yes |
| 4 | **DANGEROUS** | Typically blocked | (none currently) | Yes |

## How tools work

1. The LLM decides to use a tool during a conversation
2. The gateway's **ToolExecutor** normalizes the tool name and runs DLP on inputs
3. The **ToolCoordinator** resolves the security tier and checks the Cheese policy
4. If the tier is RESTRICTED (3+), the **ApprovalManager** asks the user for confirmation
5. Allowed calls are routed: container tools via NATS, local tools via direct function call
6. Tool outputs pass through DLP scanning before returning to the LLM
7. All tool executions are recorded in the audit log

## Enabling tools

Set `enabled = true` in the tool's config section:

```toml
[tools.browser]
enabled = true
```

Or use the CLI:

```bash
nachos add tool browser
nachos restart
```

## Parallel execution

The coordinator automatically determines if tool calls in a batch can run in parallel:

- **Duplicate tools**: Same tool appears twice -- sequential
- **Write-then-read**: Write tool followed by read on the same resource -- sequential
- **Otherwise**: Parallel via `Promise.all`

## Tool guides

### Built-in
- [Browser](/tools/browser) -- web automation
- [Filesystem](/tools/filesystem) -- file access
- [Code Runner](/tools/code-runner) -- sandboxed execution
- [Shell / Exec](/tools/shell) -- CLI commands
- [Agent Exec](/tools/agent-exec) -- autonomous coding

### Web
- [Web Search](/tools/web-search) -- Brave Search API
- [Web Fetch](/tools/web-fetch) -- page content extraction

### Integrations
- [GitHub](/tools/github) -- repository management
- [Bitbucket](/tools/bitbucket) -- Bitbucket Cloud

### State
- [Memory](/tools/memory) -- persistent memory system
- [Cron](/tools/cron) -- scheduled tasks
