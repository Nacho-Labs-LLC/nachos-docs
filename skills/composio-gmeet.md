---
title: "Google Meet (Composio)"
description: "Create Google Meet video conferencing links through Composio."
---

# Google Meet (Composio)

Generate Google Meet video conference links via Composio.

## Setup

```toml
[tools.composio]
enabled = true
allowed_apps = ["googlemeet"]
```

## Actions

### Create Meeting

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

Returns a Meet link that can be shared.

## Use Cases

- Instant meeting link generation
- Scheduled video calls
- Integration with calendar events

## Related

- [Composio Integration](/tools/composio)
- [Google Calendar](/skills/composio-gcal) — Event scheduling
