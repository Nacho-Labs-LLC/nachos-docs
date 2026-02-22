---
title: "gifgrep"
description: "Search for GIFs and extract stills from the assistant."
---

# gifgrep

Search Tenor and Giphy for GIFs, download clips, and extract still frames or contact sheets. Works headlessly — no TUI in gateway context.

## Prerequisites

- `gifgrep` binary installed in the gateway container
- `GIPHY_API_KEY` (optional — Tenor demo key is used if unset, Giphy requires a key)

## Configuration

```toml
[assistant.skills]
enabled = ["gifgrep"]
```

To use Giphy, add to `.env`:

```bash
GIPHY_API_KEY=your-giphy-key
```

## What the assistant can do

- Search Tenor and Giphy for GIFs by keyword
- Return URLs or download GIF files
- Extract a still frame from a GIF at a specific timestamp
- Generate a contact sheet (multi-frame grid image) from a GIF

## Examples

```bash
# Search and return the top 5 results as URLs
gifgrep cats --max 5 --format url

# JSON output for scripting
gifgrep search --json cats

# Download the top result
gifgrep cats --download --max 1 --format url

# Extract a still at 1.5 seconds
gifgrep still ./clip.gif --at 1.5s -o still.png

# Generate a 9-frame contact sheet
gifgrep sheet ./clip.gif --frames 9 --cols 3 -o sheet.png
```

## Gotchas

- **Tenor vs. Giphy**: Without `GIPHY_API_KEY`, all searches go to Tenor using a demo key. Set `GIPHY_API_KEY` for Giphy access.
- **`--format url`**: Returns direct GIF URLs rather than downloading. More efficient when the assistant just needs to share a link.
- **TUI mode**: `gifgrep tui` opens an interactive browser — not useful in the gateway context. The assistant should use CLI flags instead.
- **Downloaded files**: Files are written to the container filesystem. Ensure the path is accessible if downstream tools need to read them.

## What's next?

- [Skills overview](/skills/index) — all available skills
- [Filesystem tool](/tools/filesystem) — reading and writing files in the gateway
