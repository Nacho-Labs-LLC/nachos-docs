---
title: "Composio Integrations"
description: "Connect to Gmail, Google Calendar, Drive, Docs, Meet, LinkedIn and more via Composio."
---

# Composio Integrations

Composio provides production-ready integrations with productivity and communication apps. Through Composio's SDK, your assistant can send emails, manage calendar events, create documents, schedule meetings, and interact with LinkedIn — all with proper OAuth authentication and rate limiting.

## What is Composio?

[Composio](https://composio.dev) is an integration platform that handles OAuth flows, API authentication, and action execution for 100+ apps. Instead of implementing each API yourself, you configure apps in Composio's dashboard and execute actions via the Composio SDK.

**Benefits:**
- **OAuth handled for you** — No need to implement OAuth flows
- **Unified API** — Same pattern for all apps
- **Rate limiting** — Built-in protection
- **Action catalog** — Pre-built actions for common tasks
- **Multi-user support** — Each user can connect their own accounts

## Configuration

### Step 1: Get Composio API Key

1. Sign up at [https://app.composio.dev](https://app.composio.dev)
2. Go to Settings → API Keys
3. Create a new API key

### Step 2: Configure nachos.toml

```toml
[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
entity_id = "default"                # Entity ID for multi-user setups
allowed_apps = [                     # Apps you want to enable
  "gmail",
  "googlecalendar",
  "googledocs",
  "googlemeet",
  "googledrive",
  "linkedin"
]
```

**Environment variables:**

```bash
export COMPOSIO_API_KEY="your-composio-api-key"
```

### Step 3: Connect Apps in Composio Dashboard

For each app you want to use:

1. Go to [https://app.composio.dev/apps](https://app.composio.dev/apps)
2. Find the app (Gmail, Google Calendar, etc.)
3. Click "Connect"
4. Complete OAuth flow to authorize access
5. App is now connected for the configured `entity_id`

**Entity IDs:**
- `"default"` — Single-user mode (all users share one connection)
- `"user_123"` — Per-user mode (each user connects their own accounts)

For multi-user setups, see [Composio's entity documentation](https://docs.composio.dev/concepts/entities).

## Supported Apps

| App              | Actions                                      | Use Cases                          |
|------------------|----------------------------------------------|------------------------------------|
| Gmail            | Send, read, search, draft, label emails      | Email automation, notifications    |
| Google Calendar  | Create/update/delete events, list calendar   | Meeting scheduling, reminders      |
| Google Docs      | Create/edit documents, export formats        | Document generation, collaboration |
| Google Meet      | Create meetings, get meeting links           | Video call scheduling              |
| Google Drive     | Upload/download/share files, manage folders  | File management, sharing           |
| LinkedIn         | Post updates, send messages, search profiles | Social media automation            |

See individual skill pages for detailed actions:
- [Gmail Skill](/skills/composio-gmail)
- [Google Calendar Skill](/skills/composio-gcal)
- [Google Docs Skill](/skills/composio-gdocs)
- [Google Meet Skill](/skills/composio-gmeet)
- [Google Drive Skill](/skills/composio-gdrive)
- [LinkedIn Skill](/skills/composio-linkedin)

## Usage

### Action Format

```json
{
  "action": "GMAIL_SEND_EMAIL",
  "app": "gmail",
  "params": {
    "to": "user@example.com",
    "subject": "Meeting Notes",
    "body": "Here are the notes from today's meeting..."
  }
}
```

**Parameters:**

- `action` (string, required) — Composio action name (e.g., `"GMAIL_SEND_EMAIL"`)
- `app` (string, required) — App identifier (e.g., `"gmail"`)
- `params` (object, required) — Action-specific parameters

### Gmail Example

Send an email:

```json
{
  "action": "GMAIL_SEND_EMAIL",
  "app": "gmail",
  "params": {
    "to": "colleague@example.com",
    "subject": "Q1 Report",
    "body": "Please review the attached Q1 report.",
    "attachments": []
  }
}
```

Search emails:

```json
{
  "action": "GMAIL_SEARCH_EMAILS",
  "app": "gmail",
  "params": {
    "query": "from:boss@example.com is:unread",
    "max_results": 10
  }
}
```

### Google Calendar Example

Create a meeting:

```json
{
  "action": "GOOGLECALENDAR_CREATE_EVENT",
  "app": "googlecalendar",
  "params": {
    "summary": "Team Sync",
    "description": "Weekly team sync meeting",
    "start": {
      "dateTime": "2024-02-24T14:00:00-05:00",
      "timeZone": "America/New_York"
    },
    "end": {
      "dateTime": "2024-02-24T15:00:00-05:00",
      "timeZone": "America/New_York"
    },
    "attendees": [
      {"email": "alice@example.com"},
      {"email": "bob@example.com"}
    ]
  }
}
```

List upcoming events:

```json
{
  "action": "GOOGLECALENDAR_LIST_EVENTS",
  "app": "googlecalendar",
  "params": {
    "calendar_id": "primary",
    "time_min": "2024-02-24T00:00:00Z",
    "time_max": "2024-02-25T00:00:00Z",
    "max_results": 10
  }
}
```

### Google Docs Example

Create a document:

```json
{
  "action": "GOOGLEDOCS_CREATE_DOCUMENT",
  "app": "googledocs",
  "params": {
    "title": "Meeting Notes - Feb 24",
    "content": "# Team Meeting\n\n## Attendees\n- Alice\n- Bob\n\n## Topics..."
  }
}
```

### Google Meet Example

Create a meeting:

```json
{
  "action": "GOOGLEMEET_CREATE_MEETING",
  "app": "googlemeet",
  "params": {
    "summary": "Quick Sync",
    "start_time": "2024-02-24T14:00:00-05:00",
    "duration_minutes": 30
  }
}
```

Returns a meeting link that can be shared.

### Google Drive Example

Upload a file:

```json
{
  "action": "GOOGLEDRIVE_UPLOAD_FILE",
  "app": "googledrive",
  "params": {
    "name": "report.pdf",
    "mime_type": "application/pdf",
    "content": "<base64-encoded-content>",
    "folder_id": "optional-folder-id"
  }
}
```

Share a file:

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

### LinkedIn Example

Post an update:

```json
{
  "action": "LINKEDIN_CREATE_POST",
  "app": "linkedin",
  "params": {
    "text": "Excited to share our new AI assistant framework! Check it out at https://nachos.dev",
    "visibility": "PUBLIC"
  }
}
```

## Common Workflows

### 1. Email Triage

> "Check my unread emails from my boss"

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

### 2. Meeting Scheduling

> "Schedule a 1-hour meeting with Alice and Bob tomorrow at 2pm"

```json
{
  "action": "GOOGLECALENDAR_CREATE_EVENT",
  "app": "googlecalendar",
  "params": {
    "summary": "Team Discussion",
    "start": {
      "dateTime": "2024-02-25T14:00:00-05:00",
      "timeZone": "America/New_York"
    },
    "end": {
      "dateTime": "2024-02-25T15:00:00-05:00",
      "timeZone": "America/New_York"
    },
    "attendees": [
      {"email": "alice@example.com"},
      {"email": "bob@example.com"}
    ]
  }
}
```

### 3. Document Generation

> "Create a meeting summary document"

The assistant can:
1. Extract meeting context from conversation
2. Format as structured document
3. Create Google Doc with the content
4. Share with attendees via Drive

### 4. Social Media Update

> "Post about our new feature on LinkedIn"

The assistant can:
1. Craft post text from context
2. Add relevant hashtags
3. Post to LinkedIn
4. Confirm success

### 5. File Sharing

> "Share the Q1 report with the team"

```json
[
  {
    "action": "GOOGLEDRIVE_SEARCH_FILES",
    "app": "googledrive",
    "params": {
      "query": "name contains 'Q1 Report'"
    }
  },
  {
    "action": "GOOGLEDRIVE_SHARE_FILE",
    "app": "googledrive",
    "params": {
      "file_id": "<file-id-from-search>",
      "email": "team@example.com",
      "role": "reader"
    }
  }
]
```

## Security & Privacy

### OAuth Scopes

When you connect an app in Composio, you grant specific OAuth scopes. Review requested permissions carefully.

**Gmail scopes:**
- `gmail.send` — Send emails
- `gmail.readonly` — Read emails
- `gmail.modify` — Modify emails (labels, trash, etc.)

**Calendar scopes:**
- `calendar.events` — Manage events
- `calendar.readonly` — Read events

### Data Access

- Composio has access to your connected accounts
- Actions are executed on behalf of the connected user
- Review Composio's privacy policy and security practices
- Use separate "bot" Google accounts for production deployments

### Rate Limiting

Composio enforces rate limits per app and per entity:
- Gmail: 100 emails/day (adjustable)
- Calendar: 1000 events/day
- Drive: 1000 requests/day

Check Composio dashboard for current limits and usage.

## Allowed Apps Restriction

Control which apps are available:

```toml
[tools.composio]
enabled = true
allowed_apps = ["gmail", "googlecalendar"]  # Only allow Gmail and Calendar
```

Attempts to use other apps will be rejected:

```
App "linkedin" is not in the allowed apps list: gmail, googlecalendar
```

## Error Handling

Common errors:

| Error                          | Cause                              | Solution                              |
|--------------------------------|------------------------------------|---------------------------------------|
| `Composio not configured`      | Missing API key                    | Set `COMPOSIO_API_KEY`                |
| `App not in allowed list`      | App not in `allowed_apps`          | Add app to config                     |
| `OAuth not connected`          | App not connected in dashboard     | Connect app in Composio dashboard     |
| `Invalid entity`               | Wrong entity ID                    | Check `entity_id` in config           |
| `Action not found`             | Invalid action name                | Check Composio action catalog         |
| `Rate limit exceeded`          | Too many API calls                 | Wait or increase limits in dashboard  |

## Multi-User Setup

For multi-user deployments, use per-user entity IDs:

```toml
[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
entity_id = "user_${USER_ID}"  # Dynamic entity per user
```

Each user connects their own Google/LinkedIn accounts via Composio.

**Implementation:**
1. Create entity per user: `composio entity create --id user_123`
2. Generate connection URL: `composio connections create --entity user_123 --app gmail`
3. User completes OAuth flow
4. Actions execute with that user's credentials

See [Composio's multi-tenant guide](https://docs.composio.dev/guides/multi-tenant) for details.

## Example nachos.toml

```toml
[tools.composio]
enabled = true
api_key = "${COMPOSIO_API_KEY}"
entity_id = "default"
allowed_apps = [
  "gmail",
  "googlecalendar",
  "googledocs",
  "googlemeet",
  "googledrive",
  "linkedin"
]
```

## Tips

- Connect apps in Composio dashboard before using them
- Use `default` entity ID for single-user setups
- Use per-user entities for multi-tenant deployments
- Check Composio action catalog for available actions
- Use skills for app-specific guidance (e.g., `composio-gmail` skill)
- Monitor usage in Composio dashboard to avoid rate limits
- Test actions in Composio's playground before automating

## Troubleshooting

**"OAuth not connected" errors:**
1. Go to [https://app.composio.dev/apps](https://app.composio.dev/apps)
2. Check if app shows as "Connected"
3. If not, click "Connect" and complete OAuth flow
4. Ensure entity ID matches config

**"Invalid action" errors:**
1. Check action name spelling
2. Verify action exists in [Composio's action catalog](https://docs.composio.dev/actions)
3. Ensure app supports the action

**Rate limit issues:**
1. Check usage in Composio dashboard
2. Adjust rate limits if on paid plan
3. Implement backoff/retry logic in automation

## Related

- [Gmail Skill](/skills/composio-gmail) — Gmail-specific actions and examples
- [Google Calendar Skill](/skills/composio-gcal) — Calendar management
- [Cron Jobs](/tools/cron) — Automate periodic email checks or calendar updates
- [Security Policies](/security/policies) — Control tool access
