---
title: "gog"
description: "Google Workspace access from the assistant — Gmail, Calendar, Drive, and more."
---

# gog

Access Google Workspace services from the assistant: Gmail, Calendar, Drive, Contacts, Sheets, and Docs. Uses OAuth 2.0 — requires a one-time setup before the assistant can call it.

## Prerequisites

- `gog` binary installed in the gateway container
- Google Cloud project with OAuth 2.0 credentials
- OAuth credentials file mounted into the gateway container

<Warning>
`gog` can send emails and modify calendar events. Only enable it for trusted users, and review your [policy rules](/security/policies) to restrict write operations if needed.
</Warning>

## Setup (one time)

Set up OAuth credentials before enabling the skill:

```bash
# Register your Google Cloud OAuth credentials
gog auth credentials /path/to/client_secret.json

# Authorize the account for the services you need
gog auth add you@gmail.com --services gmail,calendar,drive,contacts,docs,sheets

# Verify the setup
gog auth list
```

The authorized token is stored on disk. Mount the credentials directory into the gateway container so it persists across restarts.

## Configuration

```toml
[assistant.skills]
enabled = ["gog"]
```

## What the assistant can do

- Search, read, and send Gmail messages
- List and query calendar events
- Search Google Drive files
- Read cells from Google Sheets
- Export Google Docs as text
- Look up contacts

## Examples

```bash
# Search Gmail for recent messages
gog gmail search 'newer_than:7d' --max 10

# Send an email
gog gmail send --to alice@example.com --subject "Hi" --body "Hello"

# List calendar events in a date range
gog calendar events <calendarId> --from 2026-02-01T00:00:00Z --to 2026-03-01T00:00:00Z

# Search Drive
gog drive search "invoice" --max 10 --json

# Read a spreadsheet range
gog sheets get <sheetId> "Sheet1!A1:D10" --json

# Export a Doc as plain text
gog docs export <docId> --format txt --out /tmp/doc.txt
```

## Gotchas

- **Write operations**: `gog` can send email and create calendar events. By default, the assistant should confirm with the user before performing write operations. Use policy rules to enforce this.
- **Calendar IDs**: Use `primary` as the calendar ID for the default calendar. Run `gog calendar list` to see all calendars.
- **Token expiry**: OAuth tokens expire. If `gog` starts failing with auth errors, re-run `gog auth add` for the affected account.
- **`--json` flag**: Always use `--json` for scripting or when the assistant needs to process results. Human output is for display only.
- **Scope selection**: Only authorize the services you actually need (`--services gmail,calendar`). This limits the blast radius if credentials are ever compromised.

## What's next?

- [Skills overview](/skills/index) — all available skills
- [Security policies](/security/policies) — restrict write access to sensitive tools
