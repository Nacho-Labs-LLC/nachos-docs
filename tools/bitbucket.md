---
title: "Bitbucket Integration"
description: "Interact with Bitbucket repositories, PRs, issues, and pipelines via REST API."
---

# Bitbucket Integration

The Bitbucket tool enables interaction with Bitbucket Cloud repositories, pull requests, issues, and pipelines through the official REST API v2.0. It runs natively in the gateway process.

## Configuration

Add to `nachos.toml`:

```toml
[tools.bitbucket]
enabled = true
default_workspace = "myworkspace"        # Optional: Default workspace
auth_type = "app_password"               # "app_password" or "oauth"
username_env = "BITBUCKET_USERNAME"      # For app password auth
password_env = "BITBUCKET_APP_PASSWORD"  # For app password auth
workspace_allowlist = ["myworkspace"]    # Optional: Restrict workspaces
```

**Authentication methods:**

### App Password (recommended)

1. Go to Bitbucket Settings → Personal settings → App passwords
2. Create app password with required permissions:
   - Repositories: Read, Write
   - Pull requests: Read, Write
   - Issues: Read, Write
   - Pipelines: Read
3. Set environment variables:

```bash
export BITBUCKET_USERNAME="your-username"
export BITBUCKET_APP_PASSWORD="your-app-password"
```

### OAuth 2.0

1. Create an OAuth consumer in workspace settings
2. Get access token
3. Set environment variable:

```bash
export BITBUCKET_TOKEN="your-oauth-token"
```

```toml
[tools.bitbucket]
enabled = true
auth_type = "oauth"
token_env = "BITBUCKET_TOKEN"
```

## Actions

### Repository Management

#### `repo_list` — List repositories

```json
{
  "action": "repo_list",
  "workspace": "myworkspace",
  "limit": 25
}
```

**Parameters:**
- `workspace` (string, optional) — Workspace slug. Uses `default_workspace` if omitted.
- `limit` (number, optional) — Max results. Default: 25.

**Returns:** Repository slugs, names, descriptions, visibility, timestamps.

#### `repo_view` — View repository details

```json
{
  "action": "repo_view",
  "workspace": "myworkspace",
  "repo_slug": "my-repo"
}
```

**Returns:** Full repository metadata including settings, branches, and links.

### Pull Request Management

#### `pr_list` — List pull requests

```json
{
  "action": "pr_list",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "state": "OPEN",
  "limit": 25
}
```

**Parameters:**
- `workspace` (string, required) — Workspace slug.
- `repo_slug` (string, required) — Repository slug.
- `state` (string, optional) — `"OPEN"`, `"MERGED"`, `"DECLINED"`, or `"SUPERSEDED"`.
- `limit` (number, optional) — Max results. Default: 25.

**Example:**

> "Show me all open PRs in the backend repo"

```json
{
  "action": "pr_list",
  "workspace": "myorg",
  "repo_slug": "backend",
  "state": "OPEN"
}
```

#### `pr_view` — View PR details

```json
{
  "action": "pr_view",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pr_id": 42
}
```

**Returns:** Full PR details including description, reviewers, approvals, merge status.

#### `pr_create` — Create a pull request

```json
{
  "action": "pr_create",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "title": "Add user authentication",
  "source_branch": "feature/auth",
  "destination_branch": "develop",
  "description": "Implements JWT-based authentication.\n\nCloses PROJ-123"
}
```

**Parameters:**
- `title` (string, required) — PR title.
- `source_branch` (string, required) — Branch with changes.
- `destination_branch` (string, required) — Target branch.
- `description` (string, optional) — PR description (Markdown supported).

#### `pr_diff` — Get PR diff

```json
{
  "action": "pr_diff",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pr_id": 42
}
```

Returns the unified diff. Truncated at 50KB if large.

#### `pr_merge` — Merge a pull request

```json
{
  "action": "pr_merge",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pr_id": 42,
  "merge_strategy": "squash"
}
```

**Parameters:**
- `merge_strategy` (string, optional) — `"merge_commit"`, `"squash"`, or `"fast_forward"`.

**Security note:** Destructive action. Consider requiring confirmation.

#### `pr_approve` — Approve a pull request

```json
{
  "action": "pr_approve",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pr_id": 42
}
```

**Use case:** "Approve PR #42 after reviewing the changes"

#### `pr_comment` — Comment on a pull request

```json
{
  "action": "pr_comment",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pr_id": 42,
  "body": "LGTM! Ship it! 🚀"
}
```

### Issue Management

#### `issue_list` — List issues

```json
{
  "action": "issue_list",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "state": "open",
  "limit": 25
}
```

**Parameters:**
- `state` (string, optional) — Filter by state (exact values depend on your issue tracker configuration).
- `limit` (number, optional) — Max results. Default: 25.

#### `issue_view` — View issue details

```json
{
  "action": "issue_view",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "issue_id": 123
}
```

#### `issue_create` — Create an issue

```json
{
  "action": "issue_create",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "title": "Bug: Login page not responsive",
  "content": "The login form breaks on mobile devices...",
  "kind": "bug",
  "priority": "major"
}
```

**Parameters:**
- `title` (string, required) — Issue title.
- `content` (string, optional) — Issue description.
- `kind` (string, optional) — `"bug"`, `"enhancement"`, `"proposal"`, or `"task"`.
- `priority` (string, optional) — `"trivial"`, `"minor"`, `"major"`, `"critical"`, or `"blocker"`.

#### `issue_comment` — Add comment to issue

```json
{
  "action": "issue_comment",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "issue_id": 123,
  "body": "Fixed in PR #42"
}
```

### Pipeline Management

#### `pipeline_list` — List pipelines

```json
{
  "action": "pipeline_list",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "limit": 25
}
```

**Returns:** Pipeline runs with UUIDs, states, build numbers, timestamps.

**Use case:** "Show me recent pipeline runs"

#### `pipeline_view` — View pipeline details

```json
{
  "action": "pipeline_view",
  "workspace": "myworkspace",
  "repo_slug": "my-repo",
  "pipeline_uuid": "{12345678-1234-1234-1234-123456789012}"
}
```

**Returns:** Full pipeline run details including steps, logs, and artifacts.

**Use case:** "Why did the last pipeline fail?"

### Search

#### `search` — Search code

```json
{
  "action": "search",
  "workspace": "myworkspace",
  "query": "function authenticate",
  "limit": 25
}
```

**Parameters:**
- `query` (string, required) — Search query.
- `limit` (number, optional) — Max results. Default: 25.

**Note:** Searches across all repositories in the workspace.

## Rate Limiting

The tool enforces **30 calls per minute per user** to prevent abuse. Bitbucket API also has rate limits (enforced per app password or OAuth token).

## Security

### Workspace allowlist

Restrict which workspaces can be accessed:

```toml
[tools.bitbucket]
enabled = true
workspace_allowlist = ["myworkspace", "myorg"]
```

Access to unlisted workspaces will be denied.

### Credential security

- **Never hardcode credentials** in configuration files.
- Use environment variables for usernames and passwords.
- Rotate app passwords regularly.
- Use minimal required permissions when creating app passwords.

## Common Workflows

### 1. PR Review Workflow

> "Show me all PRs waiting for review"

```json
{
  "action": "pr_list",
  "workspace": "myorg",
  "repo_slug": "backend",
  "state": "OPEN"
}
```

Then review specific PRs:

```json
[
  {
    "action": "pr_view",
    "pr_id": 42
  },
  {
    "action": "pr_diff",
    "pr_id": 42
  }
]
```

If approved:

```json
{
  "action": "pr_approve",
  "pr_id": 42
}
```

### 2. Pipeline Status Check

> "Did the latest deploy succeed?"

```json
[
  {
    "action": "pipeline_list",
    "limit": 5
  },
  {
    "action": "pipeline_view",
    "pipeline_uuid": "{most-recent-uuid}"
  }
]
```

### 3. Issue Triage

> "Create a bug report for the login issue"

The assistant extracts context from conversation and creates:

```json
{
  "action": "issue_create",
  "workspace": "myorg",
  "repo_slug": "frontend",
  "title": "Bug: Login fails on Safari",
  "content": "User reported login failure on Safari 17...",
  "kind": "bug",
  "priority": "major"
}
```

### 4. Code Search

> "Where is the authentication logic?"

```json
{
  "action": "search",
  "workspace": "myorg",
  "query": "authenticate user",
  "limit": 10
}
```

### 5. Merge Approved PRs

> "Merge PR #42 with squash"

```json
{
  "action": "pr_merge",
  "workspace": "myorg",
  "repo_slug": "backend",
  "pr_id": 42,
  "merge_strategy": "squash"
}
```

## Error Handling

Common errors:

| Error                            | Cause                                      | Solution                              |
|----------------------------------|--------------------------------------------|---------------------------------------|
| `Authentication not configured`  | Missing credentials                        | Set `BITBUCKET_USERNAME` and password |
| `Workspace not in allowlist`     | Trying to access restricted workspace      | Add to `workspace_allowlist`          |
| `Rate limit exceeded`            | Too many calls per minute                  | Wait and retry                        |
| `Bitbucket API error (401)`      | Invalid credentials                        | Check app password                    |
| `Bitbucket API error (404)`      | Resource not found                         | Verify workspace/repo/PR exists       |
| `Bitbucket API error (403)`      | Insufficient permissions                   | Update app password permissions       |

## Example nachos.toml

```toml
[tools.bitbucket]
enabled = true
default_workspace = "myorg"
auth_type = "app_password"
username_env = "BITBUCKET_USERNAME"
password_env = "BITBUCKET_APP_PASSWORD"
workspace_allowlist = ["myorg", "myorg-staging"]
```

## Comparison with GitHub Tool

| Feature                | Bitbucket                  | GitHub                    |
|------------------------|----------------------------|---------------------------|
| Authentication         | App password or OAuth      | PAT or GitHub App         |
| CLI dependency         | No (uses API directly)     | Yes (requires `gh`)       |
| Workspace/Org model    | Workspace-based            | Org/User-based            |
| Pipeline/Actions       | Pipelines API              | Workflow runs API         |
| Issue tracking         | Built-in issue tracker     | GitHub Issues             |
| Merge strategies       | merge_commit/squash/ff     | merge/squash/rebase       |

## Tips

- Use `default_workspace` to avoid repeating workspace in every call
- Check pipeline status before merging PRs
- Use `pr_approve` before `pr_merge` for audit trail
- Search is workspace-wide — useful for finding code patterns
- Pipeline UUIDs are returned with brackets: `{uuid}` — use them as-is

## Related

- [GitHub Integration](/tools/github) — Similar features for GitHub
- [Security Policies](/security/policies) — Tool access control
- [Cron Jobs](/tools/cron) — Automate PR checks and pipeline monitoring
