---
title: "Agent Exec"
description: "Launch Claude Code CLI as an autonomous coding subprocess."
---

# Agent Exec

The agent exec tool spawns Claude Code CLI (`claude`) as an autonomous subprocess for self-contained coding tasks. Unlike subagents (which run through the Nachos LLM pipeline), agent exec gives the subprocess full Claude Code capabilities including native file editing, search, and terminal access.

## Configuration

```toml
[tools.agent_exec]
enabled = true
max_concurrent = 2          # Max parallel agents
default_timeout = 300       # 5 minutes (seconds)
max_timeout = 1800          # 30 minutes max (seconds)
max_output_buffer = 524288  # 500 KB ring buffer
```

## Security tier

**ELEVATED (2)** -- requires policy approval. Best used in `permissive` security mode since the subprocess has host filesystem access.

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | string | Yes | `"spawn"`, `"status"`, `"output"`, `"cancel"`, `"list"` |
| `task` | string | For spawn | Prompt for the agent |
| `cwd` | string | No | Working directory |
| `timeout` | number | No | Timeout in seconds (default: 300, max: 1800) |
| `agentId` | string | For status/output/cancel | Agent process ID |
| `tail` | number | For output | Return last N lines only |

## Actions

### spawn

Start a new Claude Code subprocess:

```json
{
  "action": "spawn",
  "task": "Fix the failing TypeScript compilation errors in src/",
  "cwd": "/workspace/project",
  "timeout": 600
}
```

Returns an `agentId` for tracking.

### status

Check the status of a running agent:

```json
{
  "action": "status",
  "agentId": "agent-abc123"
}
```

Status values: `running`, `completed`, `failed`, `cancelled`, `timeout`.

### output

Read the agent's output:

```json
{
  "action": "output",
  "agentId": "agent-abc123",
  "tail": 50
}
```

### cancel

Stop a running agent:

```json
{
  "action": "cancel",
  "agentId": "agent-abc123"
}
```

### list

List all agent processes:

```json
{
  "action": "list"
}
```

## Process lifecycle

1. Spawns `claude -p <task> --output-format stream-json --max-turns 50`
2. Passes only `ANTHROPIC_API_KEY`, `PATH`, `HOME`, `USERPROFILE` environment variables
3. SIGTERM on timeout/cancel, SIGKILL after 5 seconds if still alive
4. Completed processes are auto-cleaned after 1 hour

## Resource limits

| Limit | Value |
|-------|-------|
| Max concurrent agents | 2 (configurable) |
| Default timeout | 5 minutes |
| Max timeout | 30 minutes |
| Output buffer | 500 KB (ring buffer -- keeps most recent) |
| Auto-cleanup | 1 hour after completion |

## Agent Exec vs Subagents

| Aspect | Agent Exec | Subagents |
|--------|-----------|-----------|
| **Process model** | `claude` CLI subprocess | LLM API through Nachos pipeline |
| **Tool access** | Full Claude Code capabilities | Nachos tool definitions (sandboxed) |
| **File access** | Host filesystem (cwd) | Isolated workspace per run |
| **Network** | Inherited from host | Controlled via Nachos policy |
| **Result delivery** | Poll via status/output | Announced to channel |
| **Workflows** | Single task only | Multi-step DAG support |
| **Use case** | Coding, refactoring, file operations | Structured async work with policy |

**Use agent exec when** the task requires full filesystem access, Claude Code's built-in capabilities, or is a self-contained coding job.

**Use subagents when** you need policy-controlled execution, result announcements, or multi-step workflows.

## Gotchas

- **Host filesystem access**: The subprocess runs with access to the host filesystem from `cwd`. This is powerful but requires trust.
- **ANTHROPIC_API_KEY required**: The subprocess calls the Anthropic API directly (not through the Nachos LLM proxy).
- **Permissive mode recommended**: Since agent exec has ELEVATED security tier and host access, it works best in permissive mode.
- **Output is a ring buffer**: If the agent produces more than 500 KB of output, older content is discarded.
- **No steering**: Unlike subagents, you cannot inject messages into a running agent exec process.
