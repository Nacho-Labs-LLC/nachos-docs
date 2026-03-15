---
title: "Shell / Exec"
description: "Execute allowlisted CLI commands directly in the gateway process."
---

# Shell / Exec

The shell tool runs CLI commands directly in the gateway process using `child_process.spawn()` with `shell: false` for injection safety. Only pre-approved binaries can execute, with strict command validation and audit logging.

## Tool names

- `exec` -- primary tool name
- `shell` -- alias

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `command` | string | Yes | Command string |
| `cwd` | string | No | Working directory |
| `env` | Record | No | Additional environment variables |
| `timeout` | number | No | Timeout in milliseconds |

## Resource limits

| Limit | Value |
|-------|-------|
| Max output | 100 KB (stdout and stderr independently) |
| Default timeout | 30 seconds |
| Max timeout | 300 seconds (5 minutes) |
| Build tool timeout | 300 seconds |
| Audit log | JSON-line entries to `/var/log/nachos/shell-audit.log` |

## Security controls

The shell tool implements multiple layers of security:

1. **Binary allowlist** -- only pre-approved binaries can execute
2. **Command substitution blocked** -- backticks, `$()`, `${}`, and process substitution are rejected
3. **Subcommand validation** -- `git`, `docker`, and `ip` have restricted subcommand sets
4. **Readonly enforcement** -- write-capable flags blocked on readonly tools (e.g., `sed -i`)
5. **Security mode gating** -- write operations (git push, rm, mkdir) only allowed in `permissive` mode
6. **Process cleanup** -- SIGTERM on timeout/cancel, SIGKILL after 5 seconds

## Allowed tool groups

| Group | Binaries | Mode |
|-------|----------|------|
| **Skill tools** | goplaces, gifgrep, summarize, gog | All modes |
| **File inspection** | ls, cat, head, tail, file, stat, wc, find | All (readonly) |
| **Text processing** | grep, sed, awk, cut, sort, uniq, tr, diff | All (readonly) |
| **Process inspection** | ps, pgrep, top, htop | All (readonly) |
| **Network info** | netstat, ss, lsof, ip (addr/route/link only) | All (readonly) |
| **Network debug** | ping, curl, wget, dig, nslookup | All (readonly) |
| **System info** | uname, hostname, whoami, pwd, date, uptime, free, df, du | All (readonly) |
| **Data processing** | jq, yq, json | All (readonly) |
| **Git** | git (status, log, diff, show, branch, remote, config) | Readonly; full write in permissive |
| **Docker inspect** | docker (ps, logs, inspect, images, stats, version, info) | Readonly; write ops in permissive |
| **Archive** | tar, unzip, gunzip, bunzip2 | All (readonly) |
| **Build/dev** | npm, npx, pnpm, yarn, node, python3, pip, make, tsc, vitest, eslint, prettier | All (5 min timeout) |
| **File manipulation** | mkdir, cp, mv, rm, touch, chmod, ln, tee | Permissive only |

## Security tier

The security tier varies by tool group and operation:

- Read-only commands: resolved based on the tool group
- Write commands in permissive mode: ELEVATED (2)
- Build/dev commands: STANDARD (1)

## Example usage

The assistant can run shell commands to inspect the environment:

```json
{
  "command": "git log --oneline -10",
  "cwd": "/workspace/project"
}
```

```json
{
  "command": "find . -name '*.ts' | wc -l",
  "cwd": "/workspace"
}
```

## Gotchas

- **Not a shell**: Commands run with `shell: false`, so shell features like pipes and redirects are implemented via argument parsing, not a shell process.
- **Readonly by default**: Most tool groups are readonly in strict and standard modes. Write operations require `permissive` security mode.
- **Command substitution is blocked**: `$(...)`, backticks, and `${...}` in arguments are rejected for security.
- **Output truncation**: Outputs exceeding 100 KB are truncated.
- **Audit trail**: Every shell execution is logged to the audit system with command, arguments, exit code, and duration.
