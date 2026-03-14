---
title: "Filesystem"
description: "Read, write, edit, and patch files in workspace directories."
---

# Filesystem

The filesystem toolset runs in isolated Docker containers on the `nachos-internal` network. It provides four operations: read, write, edit, and patch. Each mode runs via the `TOOL_MODE` environment variable and communicates via NATS request/reply.

## Tools

| Tool | Security Tier | Purpose |
|------|--------------|---------|
| `filesystem_read` | SAFE (0) | Read files, list directories, get file metadata |
| `filesystem_write` | ELEVATED (2) | Write, create, delete files; create directories |
| `filesystem_edit` | ELEVATED (2) | Line-based file editing (replace, insert, delete) |
| `filesystem_patch` | ELEVATED (2) | Apply unified diff patches |

## Configuration

```toml
[tools.filesystem]
enabled = true
paths = ["./workspace"]
write = true
max_file_size = "10MB"
```

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable the filesystem tool |
| `paths` | string[] | `["./workspace"]` | Directories the tool can access |
| `write` | boolean | `true` | Allow write operations |
| `max_file_size` | string | `"10MB"` | Maximum file size for read/write |

## filesystem_read

Read file contents, list directories, or get file metadata.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | `"read" \| "list" \| "stat"` | Yes | Operation to perform |
| `path` | string | Yes | File or directory path |
| `encoding` | string | No | `"utf-8"`, `"ascii"`, `"base64"`, `"hex"`, `"binary"` (default: `"utf-8"`) |

**Returns:**
- `read`: File contents as text
- `list`: JSON with `path`, `entries[]` (name, type, path), `count`
- `stat`: JSON with `path`, `type`, `size`, `created`, `modified`, `mode`

## filesystem_write

Create, overwrite, or delete files and directories.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | `"write" \| "create" \| "delete" \| "mkdir"` | Yes | Operation |
| `path` | string | Yes | File or directory path |
| `content` | string | For write/create | File content |
| `recursive` | boolean | No | Create parent dirs for mkdir |

- `write`: Overwrites existing file (fails if file does not exist)
- `create`: Creates new file (fails if file exists, auto-creates parent directories)
- `delete`: Removes a file (not directories)
- `mkdir`: Creates directory, optionally recursive

## filesystem_edit

Line-based editing: replace, insert, or delete lines.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | `"replace" \| "insert" \| "delete"` | Yes | Edit action |
| `path` | string | Yes | File path |
| `line` | number | Yes | Line number (1-based) |
| `content` | string | For replace/insert | New content |
| `count` | number | No | Lines to delete (default: 1) |

## filesystem_patch

Apply unified diff patches to files.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `path` | string | Yes | File to patch |
| `patch` | string | Yes | Unified diff content |
| `reverse` | boolean | No | Apply in reverse |
| `dryRun` | boolean | No | Test without applying |

## Path scoping and security

- Paths are validated via `PathValidator` -- restricted to configured `ALLOWED_PATHS`
- Directory traversal is blocked (`../` patterns rejected)
- Paths are relative to the project root (where `nachos.toml` lives)
- The tool container mounts only specified directories -- no visibility into parent or host filesystem

```toml
# Good: scoped to specific directories
paths = ["./workspace", "./data/exports"]

# Avoid: overly broad access
paths = ["/"]
```

## Read-only mode

To allow reading without writing, set `write = false`. This drops the security tier from ELEVATED (2) to STANDARD (1):

```toml
[tools.filesystem]
enabled = true
paths = ["./workspace", "./docs"]
write = false
```

## Gotchas

- **Paths are bind mounts**: The directories must exist on the host before `nachos up`. Nachos does not create them.
- **File size limit**: Files exceeding `max_file_size` (default 10 MB) are rejected.
- **No symlink following**: The tool does not follow symlinks that point outside the allowed paths.
- **Concurrent access**: Multiple tools writing to the same directory may cause race conditions. Nachos does not provide file locking.
