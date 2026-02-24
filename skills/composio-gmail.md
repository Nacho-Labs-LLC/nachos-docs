---
title: "Gmail (Composio)"
description: "Send, read, search, and manage emails via Gmail through Composio integration."
---

# Gmail (Composio)

Automate Gmail through Composio's integration. Send emails, read messages, search inbox, manage labels, and create drafts.

## Setup

1. Configure Composio tool (see [Composio Integration](/tools/composio))
2. Connect Gmail in [Composio dashboard](https://app.composio.dev/apps)
3. Add `"gmail"` to `allowed_apps` in `nachos.toml`

```toml
[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
allowed_apps = ["gmail"]
```

## Common Actions

### Send Email

```json
{
  "action": "GMAIL_SEND_EMAIL",
  "app": "gmail",
  "params": {
    "to": "recipient@example.com",
    "subject": "Meeting Notes",
    "body": "Here are the notes from today's meeting...",
    "cc": "team@example.com",
    "attachments": []
  }
}
```

### Search Emails

```json
{
  "action": "GMAIL_SEARCH_EMAILS",
  "app": "gmail",
  "params": {
    "query": "from:boss@company.com is:unread",
    "max_results": 10
  }
}
```

**Search syntax:**
- `is:unread` — Unread messages
- `from:user@example.com` — From specific sender
- `subject:meeting` — Subject contains "meeting"
- `label:important` — Has specific label
- `has:attachment` — Has attachments
- `after:2024/02/01` — Date range

### Read Email

```json
{
  "action": "GMAIL_GET_EMAIL",
  "app": "gmail",
  "params": {
    "message_id": "abc123xyz"
  }
}
```

### Create Draft

```json
{
  "action": "GMAIL_CREATE_DRAFT",
  "app": "gmail",
  "params": {
    "to": "recipient@example.com",
    "subject": "Draft Subject",
    "body": "Draft content..."
  }
}
```

### Add Label

```json
{
  "action": "GMAIL_ADD_LABEL",
  "app": "gmail",
  "params": {
    "message_id": "abc123xyz",
    "label_name": "Important"
  }
}
```

## Use Cases

### Email Triage

> "Check my unread emails from my manager"

```json
{
  "action": "GMAIL_SEARCH_EMAILS",
  "app": "gmail",
  "params": {
    "query": "from:manager@company.com is:unread",
    "max_results": 10
  }
}
```

### Send Meeting Summary

> "Email the team a summary of today's meeting"

Assistant can:
1. Extract meeting context from conversation
2. Format as email body
3. Send via Gmail

### Automated Responses

> "Reply to this email with a confirmation"

```json
{
  "action": "GMAIL_SEND_EMAIL",
  "app": "gmail",
  "params": {
    "to": "sender@example.com",
    "subject": "Re: Your Request",
    "body": "Confirmed. I'll take care of this by Friday.",
    "in_reply_to": "original_message_id"
  }
}
```

### Label Management

> "Label all emails from alice as 'Project X'"

```json
[
  {
    "action": "GMAIL_SEARCH_EMAILS",
    "app": "gmail",
    "params": {
      "query": "from:alice@company.com"
    }
  },
  {
    "action": "GMAIL_ADD_LABEL",
    "app": "gmail",
    "params": {
      "message_id": "<each-message-id>",
      "label_name": "Project X"
    }
  }
]
```

## With Heartbeat

Automate periodic email checks:

**HEARTBEAT.md:**

```markdown
## Email Check

Search for:
- `is:unread label:urgent`
- `is:unread from:boss@company.com`

If any found: Summarize senders and subjects.
Otherwise: HEARTBEAT_OK
```

See [Heartbeat System](/tools/heartbeat) for details.

## Permissions

Gmail OAuth scopes required:
- `gmail.send` — Send emails
- `gmail.readonly` — Read emails
- `gmail.modify` — Modify labels, mark read/unread

## Rate Limits

- 100 emails/day (Composio default, adjustable)
- Gmail API: 250 quota units/user/second

## Tips

- Use search filters to narrow results before reading
- Cache frequently accessed email IDs
- Use drafts for approval workflows
- Combine with calendar for meeting follow-ups
- Use labels for organization and automation

## Related

- [Composio Integration](/tools/composio) — Main integration guide
- [Google Calendar](/skills/composio-gcal) — Meeting scheduling
- [Heartbeat](/tools/heartbeat) — Automate email monitoring
- [Cron](/tools/cron) — Schedule email tasks
