---
title: "Web Search"
description: "Search the web using Brave Search API for real-time information retrieval."
---

# Web Search

The web search tool provides real-time web search capabilities using the Brave Search API. It runs natively in the gateway process and returns ranked results with titles, URLs, and snippets.

## Configuration

Add to `nachos.toml`:

```toml
[tools.web_search]
enabled = true
api_key = "${BRAVE_API_KEY}"       # Required: Brave Search API key
default_country = "US"              # Optional: Default country code
safe_search = "moderate"            # Optional: Safe search level
max_results = 10                    # Optional: Default result count
```

**Environment variables:**

```bash
export BRAVE_API_KEY="your-brave-api-key"
```

Get your API key at [https://brave.com/search/api/](https://brave.com/search/api/)

## Usage

### Basic Search

```json
{
  "query": "what is nachos framework"
}
```

**Returns:**

```
Found 10 results for: "what is nachos framework"

1. Nachos - AI Assistant Framework
   https://nachos.dev
   Nachos is an open-source framework for building AI assistants...

2. GitHub - nachos-labs/nachos
   https://github.com/nachos-labs/nachos
   Production-ready AI assistant framework with tool calling...
   (2 days ago)

...
```

### Parameters

```json
{
  "query": "ai agents",
  "count": 5,
  "country": "US",
  "search_lang": "en",
  "freshness": "pw",
  "safe_search": "moderate"
}
```

**Parameters:**

- `query` (string, required) — Search query text.
- `count` (number, optional) — Number of results to return (1-20). Default: 10.
- `country` (string, optional) — Two-letter country code (e.g., `"US"`, `"DE"`, `"GB"`). Default: config `default_country` or `"US"`.
- `search_lang` (string, optional) — Language code for results (e.g., `"en"`, `"de"`, `"fr"`).
- `freshness` (string, optional) — Filter by discovery time:
  - `"pd"` — Past day (last 24 hours)
  - `"pw"` — Past week
  - `"pm"` — Past month
  - `"py"` — Past year
  - Date range: `"YYYY-MM-DDtoYYYY-MM-DD"` (e.g., `"2024-01-01to2024-02-01"`)
- `safe_search` (string, optional) — `"off"`, `"moderate"`, or `"strict"`. Default: `"moderate"`.

### Regional Search

```json
{
  "query": "best restaurants near me",
  "country": "DE",
  "search_lang": "de"
}
```

Returns results relevant to Germany in German.

### Time-Sensitive Searches

```json
{
  "query": "AI news",
  "freshness": "pd",
  "count": 10
}
```

Returns only results discovered in the last 24 hours.

### Date Range Queries

```json
{
  "query": "climate change reports",
  "freshness": "2024-01-01to2024-03-01"
}
```

Returns results published between January 1 and March 1, 2024.

## Rate Limiting

**Client-side rate limit:** 20 calls per minute per user.

**Brave API rate limit:** Depends on your Brave Search API plan. Free tier typically allows 2000 queries/month.

If rate limit is exceeded:

```
Rate limit exceeded: maximum 20 calls per minute
```

Or from Brave API:

```
Brave Search API rate limit exceeded
```

## Output Format

Results are formatted as human-readable text:

```
Found {count} results for: "{query}"

1. {title}
   {url}
   {description}
   ({age})

2. {title}
   {url}
   {description}

...
```

- **Title** — Page title
- **URL** — Full URL
- **Description** — Search snippet (may be empty)
- **Age** — Time since publication (when available, e.g., "2 days ago")

Output is truncated at 50KB if results are very large.

## Common Use Cases

### 1. Research & Fact-Checking

> "Search for recent information about quantum computing breakthroughs"

```json
{
  "query": "quantum computing breakthroughs 2024",
  "freshness": "pm",
  "count": 10
}
```

### 2. News Monitoring

> "What's the latest news about AI regulations?"

```json
{
  "query": "AI regulations news",
  "freshness": "pd",
  "count": 15
}
```

### 3. Technical Documentation

> "Find Python async/await best practices"

```json
{
  "query": "python asyncio best practices tutorial",
  "count": 5
}
```

### 4. Local/Regional Information

> "Find German documentation for Docker"

```json
{
  "query": "Docker Dokumentation",
  "country": "DE",
  "search_lang": "de"
}
```

### 5. Historical Research

> "Find articles about the 2020 pandemic from early 2020"

```json
{
  "query": "COVID-19 pandemic",
  "freshness": "2020-01-01to2020-03-31"
}
```

## Comparison with Web Fetch

| Feature                | Web Search                    | Web Fetch                     |
|------------------------|-------------------------------|-------------------------------|
| Purpose                | Find relevant pages           | Read specific page content    |
| Input                  | Search query                  | URL                           |
| Output                 | List of results (title/URL)   | Page content (markdown/text)  |
| Use when               | Don't know the URL            | Already have the URL          |
| Rate limit             | 20/min                        | 10/min                        |

**Workflow example:** Use web search to find URLs, then use web fetch to read content:

1. Search: `"nachos framework documentation"`
2. Get URL: `https://docs.nachos.dev/tools/web-search`
3. Fetch: Read full page content

## Error Handling

Common errors:

| Error                       | Cause                        | Solution                          |
|-----------------------------|------------------------------|-----------------------------------|
| `Invalid API key`           | Missing or wrong API key     | Set valid `BRAVE_API_KEY`         |
| `Rate limit exceeded`       | Too many queries per minute  | Wait and retry                    |
| `API rate limit exceeded`   | Brave API monthly limit hit  | Upgrade Brave API plan            |
| `Invalid parameters`        | Missing or invalid query     | Provide valid `query` string      |
| `API error: 401`            | Authentication failed        | Check API key                     |
| `API error: 429`            | Brave rate limit             | Wait or upgrade plan              |

## Privacy & Safety

**Safe search modes:**

- `"off"` — No filtering (may return adult content)
- `"moderate"` (default) — Filters explicit adult content
- `"strict"` — Aggressive filtering (family-safe)

**Privacy:**

- Brave Search is privacy-focused (no user tracking)
- Queries are sent directly to Brave Search API
- No query logs are stored by Nachos (unless you enable audit logging)

## Example nachos.toml

```toml
[tools.web_search]
enabled = true
api_key = "${BRAVE_API_KEY}"
default_country = "US"
safe_search = "moderate"
max_results = 10
```

## Tips

- Use `freshness` filters for news and time-sensitive queries
- Adjust `count` based on task (5 for quick answers, 20 for research)
- Combine with web fetch for deep dives: search → fetch → summarize
- Use specific keywords to improve relevance (e.g., "python tutorial" vs "python")
- Regional searches work best with localized queries in the local language

## Related

- [Web Fetch](/tools/web-fetch) — Fetch and read web page content
- [Browser Tool](/tools/browser) — Full browser automation with JavaScript
- [Summarize Skill](/skills/summarize) — Summarize search results or fetched pages
- [Cron Jobs](/tools/cron) — Automate periodic searches for monitoring
