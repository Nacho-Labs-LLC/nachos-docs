---
title: "GitHub Integration"
description: "Interact with GitHub repositories, issues, PRs, and workflows using the gh CLI."
---

# GitHub Integration

The GitHub tool lets your AI assistant interact with GitHub repositories, issues, pull requests, and workflows through the official `gh` CLI. It runs natively in the gateway process without requiring Docker containers.

## Configuration

Add to `nachos.toml`:

```toml
[tools.github]
enabled = true
default_repo = "owner/repo"              # Optional: Default repository
token_env = "GH_TOKEN"                   # Env var containing GitHub token
repo_allowlist = ["owner/repo", "org/*"] # Optional: Restrict to specific repos
```

**Environment variables:**

- `GH_TOKEN` or `GITHUB_TOKEN` — GitHub personal access token or fine-grained PAT

**Permissions needed:**
- `repo` — Full repository access (for private repos)
- `public_repo` — Public repositories only
- `workflow` — GitHub Actions workflows
- `write:org` — Organization features (if needed)

## Actions

The `github` tool uses action-based routing. All calls use the same tool with different `action` parameters.

### Issue Management

#### `issue_list` — List issues

```json
{
  "action": "issue_list",
  "repo": "owner/repo",
  "state": "open",
  "assignee": "username",
  "labels": ["bug", "priority:high"],
  "limit": 30
}
```

**Parameters:**
- `repo` (string, optional) — Repository in `owner/repo` format. Uses `default_repo` if omitted.
- `state` (string, optional) — `"open"`, `"closed"`, or `"all"`. Default: `"open"`.
- `assignee` (string, optional) — Filter by assignee username.
- `labels` (array, optional) — Filter by label names.
- `limit` (number, optional) — Max results. Default: 30.

**Example:**

> "Show me all open bugs assigned to alice in the nachos repo"

```json
{
  "action": "issue_list",
  "repo": "nachos/core",
  "state": "open",
  "assignee": "alice",
  "labels": ["bug"]
}
```

#### `issue_view` — View issue details

```json
{
  "action": "issue_view",
  "repo": "owner/repo",
  "number": 42
}
```

Returns full issue details including body, comments, labels, assignees, timestamps.

#### `issue_create` — Create an issue

```json
{
  "action": "issue_create",
  "repo": "owner/repo",
  "title": "Bug: Login fails on Safari",
  "body": "Detailed description here...",
  "assignee": "username",
  "labels": ["bug", "browser-compat"]
}
```

**Parameters:**
- `title` (string, required) — Issue title.
- `body` (string, optional) — Issue description (Markdown supported).
- `assignee` (string, optional) — Username to assign.
- `labels` (array, optional) — Labels to apply.

**Example workflow:**

> "Create a bug report from this conversation about the login issue"

The assistant can extract context from the chat and create a properly formatted issue.

#### `issue_comment` — Add comment to issue

```json
{
  "action": "issue_comment",
  "repo": "owner/repo",
  "number": 42,
  "body": "Fixed in PR #123"
}
```

### Pull Request Management

#### `pr_list` — List pull requests

```json
{
  "action": "pr_list",
  "repo": "owner/repo",
  "state": "open",
  "author": "username",
  "base": "main",
  "limit": 30
}
```

**Parameters:**
- `state` (string, optional) — `"open"`, `"closed"`, `"merged"`, or `"all"`.
- `author` (string, optional) — Filter by PR author.
- `base` (string, optional) — Filter by base branch (e.g., `"main"`).

#### `pr_view` — View PR details

```json
{
  "action": "pr_view",
  "repo": "owner/repo",
  "number": 123
}
```

Returns PR metadata, reviews, mergeable status, and comments.

#### `pr_create` — Create a pull request

```json
{
  "action": "pr_create",
  "repo": "owner/repo",
  "title": "Add GitHub integration tool",
  "base": "main",
  "head": "feature/github-tool",
  "body": "Implements GitHub CLI integration.\n\nCloses #42"
}
```

**Parameters:**
- `title` (string, required) — PR title.
- `base` (string, required) — Target branch (usually `"main"` or `"develop"`).
- `head` (string, required) — Source branch.
- `body` (string, optional) — PR description (Markdown supported).

#### `pr_diff` — Get PR diff

```json
{
  "action": "pr_diff",
  "repo": "owner/repo",
  "number": 123
}
```

Returns the unified diff. Output is truncated at 50KB.

**Use case:** "Show me what changed in PR #123"

#### `pr_checks` — Check CI status

```json
{
  "action": "pr_checks",
  "repo": "owner/repo",
  "number": 123
}
```

Returns status of all CI/CD checks for the PR.

**Use case:** "Are the tests passing on PR #123?"

#### `pr_merge` — Merge a pull request

```json
{
  "action": "pr_merge",
  "repo": "owner/repo",
  "number": 123,
  "method": "squash"
}
```

**Parameters:**
- `method` (string, optional) — `"merge"`, `"squash"`, or `"rebase"`. Default: repository default.

**Security note:** This is a destructive action. Consider requiring user confirmation before merging.

### Workflow Management

#### `run_list` — List workflow runs

```json
{
  "action": "run_list",
  "repo": "owner/repo",
  "workflow": "CI",
  "status": "completed",
  "limit": 30
}
```

**Parameters:**
- `workflow` (string, optional) — Filter by workflow name or file.
- `status` (string, optional) — `"completed"`, `"in_progress"`, `"queued"`, etc.

#### `run_view` — View workflow run details

```json
{
  "action": "run_view",
  "repo": "owner/repo",
  "run_id": 1234567890
}
```

Returns run status, conclusion, jobs, and timestamps.

**Use case:** "Why did the CI run fail?"

### Repository & Search

#### `repo_view` — View repository details

```json
{
  "action": "repo_view",
  "repo": "owner/repo"
}
```

Returns repo metadata: description, owner, visibility, default branch, creation date.

#### `search` — Search GitHub

```json
{
  "action": "search",
  "query": "is:open label:bug repo:owner/repo",
  "type": "issue",
  "limit": 30
}
```

**Parameters:**
- `query` (string, required) — GitHub search query syntax.
- `type` (string, optional) — `"issue"`, `"pr"`, or `"code"`. Default: `"issue"`.

**Search query syntax:**
- `is:open` / `is:closed` — Issue/PR state
- `is:pr` / `is:issue` — Filter by type
- `label:bug` — Has label
- `assignee:username` — Assigned to user
- `author:username` — Created by user
- `repo:owner/repo` — In specific repo
- `org:orgname` — In organization

**Example:** "Find all high-priority bugs in the nachos org"

```json
{
  "action": "search",
  "query": "is:open label:priority:high org:nachos-labs-llc",
  "type": "issue"
}
```

#### `api` — Raw GitHub API calls

```json
{
  "action": "api",
  "endpoint": "/repos/owner/repo/issues",
  "http_method": "GET"
}
```

**Parameters:**
- `endpoint` (string, required) — API endpoint (e.g., `"/repos/owner/repo/issues"`).
- `http_method` (string, optional) — HTTP method. Default: `"GET"`.

Use this for API endpoints not covered by dedicated actions.

**Note:** Request body support is not yet implemented. Use for GET requests only.

## Rate Limiting

The tool enforces **30 calls per minute per user** to prevent abuse. If exceeded, you'll see:

```
Rate limit exceeded. Maximum 30 GitHub calls per minute. Try again in 42s.
```

GitHub API also has its own rate limits (5000/hour for authenticated requests). The tool will return API errors if you hit GitHub's limits.

## Security

### Repository allowlist

Restrict which repositories can be accessed:

```toml
[tools.github]
enabled = true
repo_allowlist = [
  "myorg/backend",
  "myorg/frontend",
  "myorg/*"  # All repos in myorg
]
```

Access to unlisted repos will be denied.

### Token security

- **Never hardcode tokens** in configuration files.
- Use environment variables: `${GH_TOKEN}` syntax in config.
- Use fine-grained PATs with minimal required permissions.
- Rotate tokens regularly.

## Common Workflows

### 1. PR Status Check

> "What's the status of PR #123?"

```json
[
  {
    "action": "pr_view",
    "number": 123
  },
  {
    "action": "pr_checks",
    "number": 123
  }
]
```

### 2. Create Issue from Conversation

> "Create a bug report for this login issue"

The assistant extracts:
- Problem description from chat context
- Steps to reproduce
- Expected vs actual behavior

Then creates a properly formatted issue with labels.

### 3. Review CI Failures

> "Why are tests failing on main?"

```json
[
  {
    "action": "run_list",
    "status": "completed",
    "limit": 5
  },
  {
    "action": "run_view",
    "run_id": "<most_recent_failed_run>"
  }
]
```

### 4. Triage Incoming Issues

> "Show me new bugs from the last 24 hours"

```json
{
  "action": "search",
  "query": "is:open label:bug created:>2024-02-01",
  "type": "issue",
  "limit": 50
}
```

### 5. Monitor PR Reviews

> "Which PRs need review?"

```json
{
  "action": "search",
  "query": "is:open is:pr review:required",
  "type": "pr"
}
```

## Requirements

- **GitHub CLI** (`gh`) must be installed in the gateway environment
- Docker image: Pre-installed
- Manual setup: `brew install gh` (macOS) or `apt install gh` (Ubuntu)

Verify installation:

```bash
gh --version
```

## Error Handling

Common errors:

| Error                          | Cause                                    | Solution                            |
|--------------------------------|------------------------------------------|-------------------------------------|
| `GitHub CLI not installed`     | `gh` command not found                   | Install GitHub CLI                  |
| `Authentication failed`        | Invalid or missing token                 | Set valid `GH_TOKEN`                |
| `Repository not in allowlist`  | Trying to access restricted repo         | Add repo to `repo_allowlist`        |
| `Rate limit exceeded`          | Too many calls per minute                | Wait and retry                      |
| `Job not found`                | Invalid issue/PR number                  | Verify number exists                |
| `GITHUB_CLI_ERROR`             | gh CLI command failed                    | Check stderr for details            |

## Example nachos.toml

```toml
[tools.github]
enabled = true
default_repo = "myorg/backend"
token_env = "GITHUB_TOKEN"
repo_allowlist = [
  "myorg/backend",
  "myorg/frontend",
  "myorg/infrastructure"
]
```

## Tips

- Use `default_repo` to avoid repeating repo names in every call
- Use `limit` to control output size (large result sets are truncated at 50KB)
- Combine `pr_view` + `pr_diff` to understand PR context
- Use search for complex queries instead of filtering list results
- Check `pr_checks` before merging PRs

## Related

- [Bitbucket Integration](/tools/bitbucket) — Similar features for Bitbucket
- [Security Policies](/security/policies) — Tool access control
- [Cron Jobs](/tools/cron) — Automate PR checks and issue triage
