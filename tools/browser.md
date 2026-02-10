---
title: "Browser"
description: "Automated browser tool."
---

# Browser

The browser tool gives the assistant controlled web access — navigating pages, extracting content, and interacting with web applications. It runs a headless Chromium instance inside a container.

## Configuration

```toml
[tools.browser]
enabled = true
allowed_domains = ["*"]
headless = true
timeout = 30
```

| Key               | Type     | Default | Description                                |
|-------------------|----------|---------|--------------------------------------------|
| `enabled`         | boolean  | `false` | Enable the browser tool                    |
| `allowed_domains` | string[] | `["*"]` | Domains the browser can access             |
| `headless`        | boolean  | `true`  | Run without GUI (always true in containers)|
| `timeout`         | integer  | `30`    | Page load timeout in seconds               |

## Domain restrictions

Restrict which sites the assistant can browse:

```toml
[tools.browser]
allowed_domains = ["docs.google.com", "github.com", "stackoverflow.com"]
```

With `["*"]`, the browser can access any domain. In `strict` security mode, you should restrict this to known-good domains.

## What the assistant can do

- Navigate to URLs
- Extract page text content
- Take screenshots
- Fill forms and click elements
- Wait for dynamic content to load

## Network behavior

The browser container joins the `nachos-egress` network, which has external internet access. It does not have access to the `nachos-internal` network or other containers.

## Gotchas

- **Resource usage**: Headless Chromium uses significant memory. Ensure Docker has at least 2 GB allocated if using this tool.
- **JavaScript-heavy sites**: The `timeout` setting controls how long to wait for page loads. Increase it for slow SPAs.
- **Authentication**: The browser has no stored cookies or sessions. It cannot access pages that require login unless credentials are provided in the conversation.
- **`allowed_domains` is not a blocklist**: It's a strict allowlist. Only listed domains (or `*`) are accessible.
