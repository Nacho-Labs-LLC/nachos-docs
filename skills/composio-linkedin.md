---
title: "LinkedIn (Composio)"
description: "Post updates and manage LinkedIn presence through Composio integration."
---

# LinkedIn (Composio)

Automate LinkedIn posting and interactions via Composio.

## Setup

```toml
[tools.composio]
enabled = true
allowed_apps = ["linkedin"]
```

## Actions

### Create Post

```json
{
  "action": "LINKEDIN_CREATE_POST",
  "app": "linkedin",
  "params": {
    "text": "Excited to announce our new AI assistant framework! 🚀\n\nCheck it out: https://nachos.dev",
    "visibility": "PUBLIC"
  }
}
```

**Visibility:** `PUBLIC`, `CONNECTIONS`, `LOGGED_IN`

### Send Message

```json
{
  "action": "LINKEDIN_SEND_MESSAGE",
  "app": "linkedin",
  "params": {
    "recipient_id": "user_profile_id",
    "message": "Thanks for connecting!"
  }
}
```

## Use Cases

- Social media automation
- Announcement posting
- Professional networking
- Content distribution

## Tips

- Schedule posts via [Cron](/tools/cron)
- Draft posts in conversation before publishing
- Monitor engagement through LinkedIn analytics

## Related

- [Composio Integration](/tools/composio)
- [Cron](/tools/cron) — Schedule posts
