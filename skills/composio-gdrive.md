---
title: "Google Drive (Composio)"
description: "Upload, download, share, and manage files in Google Drive through Composio."
---

# Google Drive (Composio)

Manage Google Drive files and folders via Composio.

## Setup

```toml
[tools.composio]
enabled = true
allowed_apps = ["googledrive"]
```

## Actions

### Upload File

```json
{
  "action": "GOOGLEDRIVE_UPLOAD_FILE",
  "app": "googledrive",
  "params": {
    "name": "report.pdf",
    "mime_type": "application/pdf",
    "content": "<base64-encoded>",
    "folder_id": "optional-folder-id"
  }
}
```

### Share File

```json
{
  "action": "GOOGLEDRIVE_SHARE_FILE",
  "app": "googledrive",
  "params": {
    "file_id": "abc123",
    "email": "user@example.com",
    "role": "reader"
  }
}
```

**Roles:** `reader`, `writer`, `commenter`, `owner`

### Search Files

```json
{
  "action": "GOOGLEDRIVE_SEARCH_FILES",
  "app": "googledrive",
  "params": {
    "query": "name contains 'Q1 Report'"
  }
}
```

### Download File

```json
{
  "action": "GOOGLEDRIVE_DOWNLOAD_FILE",
  "app": "googledrive",
  "params": {
    "file_id": "abc123"
  }
}
```

## Use Cases

- File sharing workflows
- Document backup
- Collaborative file management
- Report distribution

## Related

- [Composio Integration](/tools/composio)
- [Google Docs](/skills/composio-gdocs) — Document creation
