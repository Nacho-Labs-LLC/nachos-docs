---
title: "goplaces"
description: "Search Google Places from the assistant."
---

# goplaces

Search the Google Places API (v1) by text query, resolve addresses to place IDs, and fetch place details and reviews. Human-readable output by default, `--json` for scripting.

## Prerequisites

- `goplaces` binary installed in the gateway container
- `GOOGLE_PLACES_API_KEY` environment variable set

<Warning>
Google Places API is a paid API. Set a budget alert and quota limit in [Google Cloud Console](https://console.cloud.google.com/) before enabling this skill.
</Warning>

## Configuration

Set the API key in `.env`:

```bash
GOOGLE_PLACES_API_KEY=your-api-key
```

Reference it in `nachos.toml`:

```toml
[assistant.skills]
enabled = ["goplaces"]
```

The gateway will inject `GOOGLE_PLACES_API_KEY` into the `goplaces` subprocess automatically.

## What the assistant can do

- Text search for businesses and places by name, category, or description
- Filter results by open-now status, minimum rating, and distance
- Resolve a place name or address to a canonical place ID
- Fetch detailed information: hours, address, phone number, website
- Retrieve user reviews for a place

## Examples

```bash
# Find open coffee shops nearby
goplaces search "coffee" --open-now --min-rating 4 --limit 5

# Search with a specific location
goplaces search "pizza" --lat 40.8 --lng -73.9 --radius-m 3000

# Resolve an address or area name to a place ID
goplaces resolve "Soho, London" --limit 5

# Get details and reviews for a place
goplaces details <place_id> --reviews

# Machine-readable output
goplaces search "sushi" --json
```

## Gotchas

- **Billing**: Every search call is billed. Use `--limit` to avoid fetching more results than needed.
- **API quota**: Set a daily quota in Google Cloud to prevent unexpected charges if the assistant over-calls.
- **Place IDs**: Place IDs from `search` or `resolve` can be passed to `details` for richer information.
- **`--json` flag**: Use it when the assistant needs to process results programmatically. Human output is better for direct display.

## What's next?

- [Skills overview](/skills/index) — all available skills
- [Security policies](/security/policies) — controlling which skills can be called
