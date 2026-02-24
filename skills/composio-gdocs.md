---
title: "Google Docs (Composio)"
description: "Create and edit Google Docs through Composio integration."
---

# Google Docs (Composio)

Create, edit, and export Google Docs via Composio.

## Setup

```toml
[tools.composio]
enabled = true
allowed_apps = ["googledocs"]
```

## Actions

### Create Document

```json
{
  "action": "GOOGLEDOCS_CREATE_DOCUMENT",
  "app": "googledocs",
  "params": {
    "title": "Meeting Notes - Feb 24",
    "content": "# Meeting Summary\n\n## Attendees\n- Alice\n- Bob"
  }
}
```

### Update Document

```json
{
  "action": "GOOGLEDOCS_UPDATE_DOCUMENT",
  "app": "googledocs",
  "params": {
    "document_id": "abc123",
    "content": "Updated content..."
  }
}
```

## Use Cases

- Generate meeting notes from conversation
- Create reports and summaries
- Document generation workflows
- Collaborative editing

## Related

- [Composio Integration](/tools/composio)
- [Google Drive](/skills/composio-gdrive) — File management
