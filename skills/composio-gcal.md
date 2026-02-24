---
title: "Google Calendar (Composio)"
description: "Create, update, and manage calendar events through Composio integration."
---

# Google Calendar (Composio)

Manage Google Calendar events through Composio. Create meetings, check availability, update events, and send invitations.

## Setup

1. Configure Composio tool (see [Composio Integration](/tools/composio))
2. Connect Google Calendar in [Composio dashboard](https://app.composio.dev/apps)
3. Add `"googlecalendar"` to `allowed_apps`

```toml
[tools.composio]
enabled = true
allowed_apps = ["googlecalendar"]
```

## Common Actions

### Create Event

```json
{
  "action": "GOOGLECALENDAR_CREATE_EVENT",
  "app": "googlecalendar",
  "params": {
    "summary": "Team Sync",
    "description": "Weekly team meeting",
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

### List Events

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

### Update Event

```json
{
  "action": "GOOGLECALENDAR_UPDATE_EVENT",
  "app": "googlecalendar",
  "params": {
    "event_id": "abc123",
    "summary": "Updated Meeting Title",
    "start": {
      "dateTime": "2024-02-24T15:00:00-05:00"
    }
  }
}
```

### Delete Event

```json
{
  "action": "GOOGLECALENDAR_DELETE_EVENT",
  "app": "googlecalendar",
  "params": {
    "event_id": "abc123"
  }
}
```

## Use Cases

### Meeting Scheduling

> "Schedule a 1-hour meeting with Alice tomorrow at 2pm"

### Check Availability

> "What's on my calendar today?"

### Meeting Reminders

With [Heartbeat](/tools/heartbeat):

```markdown
## Calendar Check

List events in next 2 hours.
If any: Remind user.
Otherwise: HEARTBEAT_OK
```

### Automated Invites

> "Create a weekly standup every Monday at 9am"

## Date/Time Formats

Use ISO 8601 format with timezone:

```json
{
  "dateTime": "2024-02-24T14:00:00-05:00",
  "timeZone": "America/New_York"
}
```

**Common timezones:**
- `America/New_York` — Eastern
- `America/Chicago` — Central
- `America/Los_Angeles` — Pacific
- `Europe/London` — GMT/BST
- `UTC` — Coordinated Universal Time

## Related

- [Composio Integration](/tools/composio)
- [Google Meet](/skills/composio-gmeet) — Video conferencing
- [Gmail](/skills/composio-gmail) — Email integration
- [Heartbeat](/tools/heartbeat) — Calendar monitoring
