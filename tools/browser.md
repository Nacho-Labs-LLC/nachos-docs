---
title: "Browser"
description: "Playwright-based browser automation running in the gateway process."
---

# Browser

The browser tool provides full browser automation via Playwright MCP. It runs a headless Chromium instance inside the gateway process (lazy-initialized on first use) and exposes 24+ browser actions.

## Configuration

```toml
[tools.browser]
enabled = true
```

**Environment variables:**

| Variable | Description |
|----------|-------------|
| `BROWSER_HEADLESS` | Set to `"false"` for headed mode (default: headless) |
| `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` | Custom Chromium path |
| `BROWSER_ALLOWED_DOMAINS` | Comma-separated domain allowlist |

## Available actions

The browser tool provides a comprehensive set of actions via `@playwright/mcp`:

**Navigation:** `browser_navigate`, `browser_go_back`, `browser_go_forward`

**Content:** `browser_snapshot`, `browser_screenshot`, `browser_console_messages`, `browser_network_requests`

**Interaction:** `browser_click`, `browser_type`, `browser_fill`, `browser_select_option`, `browser_hover`, `browser_drag`, `browser_press_key`, `browser_upload_file`, `browser_handle_dialog`

**Tabs:** `browser_tab_new`, `browser_tab_close`, `browser_tab_list`

**Utility:** `browser_wait`, `browser_close`, `browser_resize`, `browser_evaluate`, `browser_pdf_save`, `browser_install`

## Security

- **Security tier**: STANDARD (1)
- **SSRF protection**: Navigation tools (`browser_navigate`, `browser_tab_new`) validate URLs against private/local IP ranges
- **Domain allowlist**: Optional restriction via `BROWSER_ALLOWED_DOMAINS`

## Chromium launch configuration

Chromium starts with these flags for container compatibility:

```
--no-sandbox --disable-setuid-sandbox --disable-dev-shm-usage --disable-gpu
```

## What the assistant can do

- Navigate to URLs and extract page content
- Take screenshots of web pages
- Fill forms and click elements
- Wait for dynamic content to load
- Execute JavaScript in the page context
- Save pages as PDFs

## Gotchas

- **Resource usage**: Headless Chromium uses significant memory. Ensure Docker has at least 2 GB allocated.
- **Lazy initialization**: Chromium starts on the first browser tool call, not at gateway startup.
- **JavaScript-heavy sites**: Use `browser_wait` for SPAs that need time to render.
- **No persistent state**: Cookies and sessions are not preserved between browser sessions.
- **Domain allowlist is strict**: Only listed domains (or `*` by default) are accessible.
