---
title: "Bootstrap"
description: "Internal tool for managing bootstrap blocks."
---

# Bootstrap

The bootstrap tool lets the assistant read and update bootstrap blocks for the current agent. Bootstrap blocks are injected into the system prompt before identity, user profile, and memory.

This is an internal tool handled by the gateway and backed by the state layer. It does not run in a tool container.

## Configuration

```toml
[tools.bootstrap]
enabled = true
```

| Key       | Type    | Default | Description                     |
|-----------|---------|---------|---------------------------------|
| `enabled` | boolean | `true`  | Enable the bootstrap tool       |

You can also toggle via environment variable:

```bash
TOOL_BOOTSTRAP_ENABLED=false
```

## Actions

All actions use a shared schema with an `action` field.

### Get current blocks

```json
{
  "action": "get"
}
```

### Set blocks

```json
{
  "action": "set",
  "content": {
    "agents": "...",
    "soul": "...",
    "tools": "...",
    "identity": "...",
    "user": "...",
    "bootstrap": "..."
  },
  "identityCompleted": true
}
```

### Delete blocks

```json
{
  "action": "delete"
}
```

## Notes

- Blocks are stored per agent id (derived from the session user id, falling back to the session id).
- When disabled, the tool is not exposed to the model and calls return `TOOL_DISABLED`.
- When `identityCompleted` is true, the gateway removes the `bootstrap` block and records completion in the identity profile metadata.
- After identity completion, the bootstrap tool is locked; use `/identity reset` to restart onboarding.
- Subagent sessions only receive the `agents` and `tools` blocks.

## Identity reset

Operators can reset onboarding with a session command:

```
/identity reset
```

This clears the identity profile and restores the default bootstrap guidance.

## Prompt assembly order

Bootstrap blocks are inserted into the system prompt after the base prompt, then identity, user profile, memory entries/facts, and skills. Each block is rendered as:

```
Bootstrap Blocks:

[KEY]
content
```

Keys are ordered: `agents`, `soul`, `tools`, `identity`, `user`, `bootstrap`, then any extra keys alphabetically.

## Security and policy

Bootstrap tool actions are routed through the state layer and evaluated by Cheese. The gateway maps state actions to `read`/`write`/`call` and passes `metadata.stateAction` (e.g. `state.bootstrap.read`). Policies can allow or deny by matching `metadata.stateAction`.
