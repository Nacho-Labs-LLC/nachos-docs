---
title: "Skills"
description: "CLI tools the assistant can call directly from the gateway."
---

# Skills

Skills are CLI executables built into the gateway container. Unlike [tool containers](/tools/index), skills run in the gateway process itself — the assistant calls them like any other tool, and the gateway spawns the binary as a subprocess.

## How skills work

When the assistant needs to look up a place, search for a GIF, or summarize a URL, it calls the skill as a tool. The gateway:

1. Validates the call against the allowlist and policy rules
2. Spawns the CLI binary with the requested arguments
3. Captures output (stdout, stderr) and returns it to the LLM
4. Enforces timeouts and output limits

From the assistant's perspective, skills are indistinguishable from any other tool.

## Available skills

<CardGroup cols={2}>
  <Card title="goplaces" icon="map-pin" href="/skills/goplaces">
    Search Google Places — find restaurants, businesses, and points of interest by name or location.
  </Card>
  <Card title="gifgrep" icon="film" href="/skills/gifgrep">
    Search Tenor and Giphy for GIFs, download clips, and extract still frames or contact sheets.
  </Card>
  <Card title="summarize" icon="file-lines" href="/skills/summarize">
    Summarize URLs, local files, PDFs, and YouTube videos. Transcription for audio/video content.
  </Card>
  <Card title="gog" icon="google" href="/skills/gog">
    Google Workspace access — Gmail, Calendar, Drive, Contacts, Sheets, and Docs via OAuth.
  </Card>
</CardGroup>

## Enabling skills

Skills are enabled in `nachos.toml` under `[assistant.skills]`. Each skill requires its binary to be present in the gateway container and any listed environment variables to be set.

```toml
[assistant.skills]
enabled = ["goplaces", "gifgrep", "summarize", "gog"]
```

## Prerequisites per skill

| Skill | Binary | Required env vars |
|-------|--------|-------------------|
| `goplaces` | `goplaces` | `GOOGLE_PLACES_API_KEY` |
| `gifgrep` | `gifgrep` | `GIPHY_API_KEY` (optional) |
| `summarize` | `summarize` | At least one LLM API key |
| `gog` | `gog` | OAuth credentials file |

## Security

Skills run inside the gateway container with the gateway's network access. They do not have direct internet access unless the gateway joins `nachos-egress`. Skills that call external APIs (goplaces, summarize, gog) require the gateway to have outbound access.

Policy rules control which skills the assistant can call and with what arguments. See [Policies](/security/policies) for details.

## What's next?

- [goplaces](/skills/goplaces) — Google Places search
- [gifgrep](/skills/gifgrep) — GIF search and download
- [summarize](/skills/summarize) — URL and file summarization
- [gog](/skills/gog) — Google Workspace integration
- [Security policies](/security/policies) — Controlling skill access
