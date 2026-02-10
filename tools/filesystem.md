---
title: "Filesystem"
description: "Workspace file access."
---

# Filesystem

The filesystem tool lets the assistant read and write files in designated directories. Access is scoped to configured paths — the tool cannot escape its allowed directories.

## Configuration

```toml
[tools.filesystem]
enabled = true
paths = ["./workspace"]
write = true
max_file_size = "10MB"
```

| Key             | Type     | Default          | Description                          |
|-----------------|----------|------------------|--------------------------------------|
| `enabled`       | boolean  | `false`          | Enable the filesystem tool           |
| `paths`         | string[] | `["./workspace"]`| Directories the tool can access      |
| `write`         | boolean  | `true`           | Allow write operations               |
| `max_file_size` | string   | `"10MB"`         | Maximum file size for read/write     |

## Read-only mode

To allow reading without writing:

```toml
[tools.filesystem]
enabled = true
paths = ["./workspace", "./docs"]
write = false
```

This drops the security tier from 2 (Elevated) to 1 (Standard).

## What the assistant can do

- List directory contents
- Read file contents
- Create and write files (if `write = true`)
- Move and rename files (if `write = true`)
- Delete files (if `write = true`)

## Path scoping

Paths are relative to the project root (where `nachos.toml` lives). The tool container mounts only the specified directories — it has no visibility into parent directories or the host filesystem.

```toml
# Good: scoped to specific directories
paths = ["./workspace", "./data/exports"]

# Avoid: overly broad access
paths = ["/"]
```

## Gotchas

- **Paths are bind mounts**: The directories must exist on the host before `nachos up`. Nachos does not create them.
- **File size limit**: Files exceeding `max_file_size` are rejected. Increase the limit if you work with large files.
- **No symlink following**: The tool does not follow symlinks that point outside the allowed paths.
- **Concurrent access**: If multiple tools or processes write to the same directory, you may see race conditions. Nachos does not provide file locking.
