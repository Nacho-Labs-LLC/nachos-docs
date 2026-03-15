---
title: "Memory"
description: "Persistent memory system for storing and retrieving facts, preferences, and decisions."
---

# Memory

The memory tools provide persistent storage and retrieval for the assistant's long-term knowledge. Memories are stored in the state layer and can be searched by text, tags, or kind.

## Tools

| Tool | Security Tier | Purpose |
|------|--------------|---------|
| `memory_search` | SAFE (0) | Search stored memories |
| `memory_get` | SAFE (0) | Read specific memory files |
| `memory_write` | ELEVATED (2) | Save a new memory entry |
| `memory_delete` | ELEVATED (2) | Delete a memory entry |

## memory_search

Search memories by text query, tags, or kind.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `query` | string | Yes | Search text |
| `kinds` | string[] | No | Filter by kind |
| `tags` | string[] | No | Filter by tags |
| `limit` | number | No | Max results (default: 10) |
| `semantic` | boolean | No | Use semantic search (default: false) |
| `minSimilarity` | number | No | Min similarity for semantic search (0-1, default: 0.7) |

**Memory kinds:** `summary`, `preference`, `fact`, `decision`, `task`, `issue`

## memory_get

Read specific memory files.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `path` | string | Yes | Memory file path |
| `from` | number | No | Start line (1-indexed) |
| `lines` | number | No | Number of lines to read |

**Allowed paths:** `MEMORY.md`, `memory/`, `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`, `IDENTITY.md`

## memory_write

Save a memory entry for future recall.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `content` | string | Yes | Memory content |
| `kind` | string | Yes | Entry type (see kinds above) |
| `tags` | string[] | No | Tags for categorization |

## memory_delete

Delete a specific memory entry by ID.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `id` | string | Yes | Entry ID from `memory_search` |

## Rate limiting

All memory tools share a 10 calls/minute/session rate limit enforced by the ToolExecutor.

## CLI management

You can also manage memories via the CLI:

```bash
# Search memories
nachos memory query --agent-id my-bot --text "breakfast" --limit 5

# Add a memory
nachos memory append-entry \
  --agent-id my-bot \
  --kind preference \
  --content "User prefers short, direct replies" \
  --tags communication,style

# Add a fact
nachos memory append-fact \
  --agent-id my-bot \
  --subject "User" \
  --predicate "prefers" \
  --object "tacos"

# Delete a memory
nachos memory delete --agent-id my-bot --id entry_abc123
```

## Gotchas

- **Write operations are ELEVATED**: Creating and deleting memories requires the ELEVATED security tier, which is controlled by policy.
- **Semantic search is optional**: When `semantic: true`, the system uses vector similarity. This requires a configured semantic search provider.
- **Session-scoped rate limits**: The 10 calls/minute limit is per session, not per user.
