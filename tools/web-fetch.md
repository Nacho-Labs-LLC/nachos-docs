---
title: "Web Fetch (Native)"
description: "Fetch and extract content from web pages with SSRF protection and domain allowlisting."
---

# Web Fetch (Native)

The native web fetch tool retrieves and extracts readable content from web pages. It converts HTML to markdown or plain text, making web content consumable by the AI. It runs natively in the gateway process without requiring Docker containers.

## Configuration

Add to `nachos.toml`:

```toml
[tools.web_fetch_native]
enabled = true
timeout_ms = 10000                     # Optional: Request timeout (default: 10s)
max_chars = 50000                      # Optional: Max content length (default: 50KB)
domain_allowlist = [                   # Optional: Restrict to specific domains
  "docs.nachos.dev",
  "github.com",
  "wikipedia.org"
]
```

No authentication required. No API keys needed.

## Usage

### Basic Fetch

```json
{
  "url": "https://docs.nachos.dev/tools/web-search"
}
```

**Returns:**

```
Fetched content from: https://docs.nachos.dev/tools/web-search

---

# Web Search

The web search tool provides real-time web search...
```

### Parameters

```json
{
  "url": "https://example.com/article",
  "extract_mode": "markdown",
  "max_chars": 30000
}
```

**Parameters:**

- `url` (string, required) — HTTP or HTTPS URL to fetch.
- `extract_mode` (string, optional) — `"markdown"` or `"text"`. Default: `"markdown"`.
- `max_chars` (number, optional) — Maximum characters to return. Default: 50000.

### Markdown Mode (Default)

Converts HTML to structured markdown:

```json
{
  "url": "https://example.com/docs",
  "extract_mode": "markdown"
}
```

**Output:**

```markdown
# Documentation

## Getting Started

Follow these steps to install...

- Step 1: Download the package
- Step 2: Run installer

[Learn more](https://example.com/guide)
```

**Preserves:**
- Headers (`<h1>` → `# Header`)
- Links (`<a>` → `[text](url)`)
- Bold/italic (`<strong>` → `**text**`)
- Lists (`<li>` → `- item`)
- Code blocks (`<pre><code>` → ` ```code``` `)

### Text Mode

Converts HTML to plain text (strips all formatting):

```json
{
  "url": "https://example.com/article",
  "extract_mode": "text"
}
```

**Output:**

```
Documentation

Getting Started

Follow these steps to install...

Step 1: Download the package
Step 2: Run installer
```

**Use text mode when:**
- You don't need formatting
- You want to minimize token usage
- You're processing the content programmatically

## SSRF Protection

The tool blocks access to private and local addresses to prevent Server-Side Request Forgery (SSRF) attacks.

**Blocked:**
- `localhost` and `127.0.0.1`
- Private IPv4: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- Link-local: `169.254.0.0/16`
- IPv6 localhost: `::1`
- IPv6 private: `fc00::/7`, `fe80::/10`

**Attempting to access blocked addresses:**

```json
{
  "url": "http://localhost:8080/admin"
}
```

**Returns error:**

```
Access to private/local addresses is not allowed
```

This prevents attackers from using your assistant to scan internal networks or access internal services.

## Domain Allowlisting

Restrict fetching to specific trusted domains:

```toml
[tools.web_fetch_native]
enabled = true
domain_allowlist = [
  "docs.nachos.dev",
  "github.com",
  "*.wikipedia.org"
]
```

**Behavior:**

- `docs.nachos.dev` — Exact match only
- `github.com` — Matches `github.com` and `*.github.com`
- `*.wikipedia.org` — Matches all Wikipedia subdomains

**Attempting to fetch non-allowlisted domain:**

```json
{
  "url": "https://untrusted-site.com"
}
```

**Returns error:**

```
Domain untrusted-site.com is not in the allowlist
```

**Use allowlisting when:**
- Assistant is exposed to untrusted users
- You want to limit information sources
- Compliance requires domain restrictions

## Rate Limiting

**Client-side rate limit:** 10 calls per minute per user.

If exceeded:

```
Rate limit exceeded: maximum 10 calls per minute
```

## Content Processing

### Automatic Cleanup

The tool automatically removes:
- `<script>` tags and JavaScript
- `<style>` tags and CSS
- HTML comments
- Navigation menus
- Footers and sidebars (basic heuristic)

### Entity Decoding

HTML entities are converted to characters:
- `&nbsp;` → space
- `&amp;` → `&`
- `&lt;` → `<`
- `&gt;` → `>`
- `&quot;` → `"`

### Truncation

If content exceeds `max_chars`, it's truncated with notice:

```
... [Content truncated due to length limit]
```

**Adjust `max_chars` based on use case:**
- Quick summaries: 5000-10000
- Full articles: 30000-50000
- Documentation: 50000-100000 (max)

## Common Use Cases

### 1. Read Documentation

> "What does the Nachos GitHub tool documentation say?"

```json
{
  "url": "https://docs.nachos.dev/tools/github",
  "extract_mode": "markdown"
}
```

### 2. Fetch Blog Posts

> "Summarize this article: https://example.com/blog/ai-agents"

```json
{
  "url": "https://example.com/blog/ai-agents",
  "extract_mode": "markdown",
  "max_chars": 20000
}
```

Then the assistant can read and summarize the content.

### 3. Check Website Status

> "What does the homepage say?"

```json
{
  "url": "https://example.com",
  "extract_mode": "text",
  "max_chars": 5000
}
```

### 4. Extract Structured Data

> "Get the pricing information from this page"

```json
{
  "url": "https://example.com/pricing",
  "extract_mode": "markdown"
}
```

The assistant can parse markdown tables and extract pricing.

### 5. Research Workflow

```json
[
  {
    "tool": "web_search",
    "parameters": {
      "query": "nachos framework features"
    }
  },
  {
    "tool": "web_fetch_native",
    "parameters": {
      "url": "<url-from-search-results>"
    }
  }
]
```

First search, then fetch and read the most relevant result.

## Relationship to Docker Web Fetch

Nachos has two web fetch tools:

| Feature                | Native (`web_fetch_native`)      | Docker (`web_fetch`)            |
|------------------------|----------------------------------|---------------------------------|
| Container required     | No (runs in gateway)             | Yes (Docker container)          |
| JavaScript support     | No                               | Yes (can execute JS)            |
| Speed                  | Faster (no container overhead)   | Slower (Docker startup)         |
| Resource usage         | Minimal                          | Higher (container isolation)    |
| SSRF protection        | Built-in                         | Container network isolation     |
| Best for               | Static HTML pages                | Complex SPAs with JavaScript    |

**When to use native:**
- Fetching documentation sites
- Reading blog posts
- Simple HTML pages
- Speed matters
- Minimal resource usage needed

**When to use Docker web-fetch:**
- React/Vue/Angular apps (require JS execution)
- Pages with client-side rendering
- Need full browser capabilities
- Maximum isolation/security needed

## Error Handling

Common errors:

| Error                          | Cause                            | Solution                           |
|--------------------------------|----------------------------------|------------------------------------|
| `Invalid URL format`           | Malformed URL                    | Provide valid HTTP/HTTPS URL       |
| `Only http and https allowed`  | Non-web protocol (ftp, file, etc)| Use HTTP or HTTPS URLs             |
| `Access to private addresses`  | Trying to access localhost/LAN   | Use public URLs only               |
| `Domain not in allowlist`      | Domain restriction enabled       | Add domain to allowlist            |
| `Rate limit exceeded`          | Too many fetches per minute      | Wait and retry                     |
| `HTTP 404`                     | Page not found                   | Check URL                          |
| `HTTP 403`                     | Access forbidden                 | Page may require authentication    |
| `Timeout after {ms}ms`         | Page took too long to respond    | Increase `timeout_ms` or try again |
| `Unsupported content type`     | Not HTML/text (e.g., PDF, image) | Use appropriate tool for file type |

## Security Best Practices

### 1. Always Use HTTPS

```json
{
  "url": "https://example.com"  // ✅ Secure
}
```

Not:

```json
{
  "url": "http://example.com"   // ⚠️ Insecure
}
```

The tool allows HTTP but HTTPS is recommended for security.

### 2. Enable Domain Allowlisting in Production

```toml
[tools.web_fetch_native]
enabled = true
domain_allowlist = [
  "docs.nachos.dev",
  "github.com",
  "trusted-source.com"
]
```

### 3. Set Reasonable Timeouts

```toml
[tools.web_fetch_native]
enabled = true
timeout_ms = 15000  # 15 seconds max
```

Prevents hanging on slow/unresponsive sites.

### 4. Limit Content Size

```toml
[tools.web_fetch_native]
enabled = true
max_chars = 30000  # 30KB max
```

Prevents token exhaustion from massive pages.

## Example nachos.toml

```toml
[tools.web_fetch_native]
enabled = true
timeout_ms = 10000
max_chars = 50000
domain_allowlist = [
  "docs.nachos.dev",
  "github.com",
  "*.wikipedia.org",
  "developer.mozilla.org"
]
```

## Tips

- Use `markdown` mode for structured content (docs, articles)
- Use `text` mode for quick content checks or when tokens matter
- Combine with web search for discovery → fetch → analyze workflow
- Set `max_chars` based on expected content size to avoid truncation
- Check extracted content length before passing to other tools
- For JavaScript-heavy sites, use the Docker `web-fetch` tool instead

## Related

- [Web Search](/tools/web-search) — Find URLs to fetch
- [Browser Tool](/tools/browser) — Full browser automation with JavaScript support
- [Summarize Skill](/skills/summarize) — Summarize fetched content
- [Security Policies](/security/policies) — Domain restrictions and access control
