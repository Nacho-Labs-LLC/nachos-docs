---
title: "summarize"
description: "Summarize URLs, files, and YouTube videos from the assistant."
---

# summarize

Fetch and summarize web pages, local files, PDFs, and YouTube videos. Returns structured summaries or raw transcripts. Useful when the assistant needs to digest external content without using the browser tool.

## Prerequisites

- `summarize` binary installed in the gateway container
- At least one LLM API key set (see below)

## Configuration

```toml
[assistant.skills]
enabled = ["summarize"]
```

Add one or more provider keys to `.env`:

```bash
ANTHROPIC_API_KEY=your-key      # Claude models
OPENAI_API_KEY=your-key         # GPT models
GEMINI_API_KEY=your-key         # Gemini models (recommended for YouTube)
XAI_API_KEY=your-key            # Grok models
```

`summarize` will use whichever key is available. Specify a model explicitly with `--model` to control which provider is called.

## What the assistant can do

- Summarize any public URL or article
- Summarize local files and PDFs
- Transcribe and summarize YouTube videos (best-effort — depends on caption availability)
- Extract raw transcript without summarizing (`--extract-only`)

## Examples

```bash
# Summarize a URL
summarize "https://example.com/article"

# Summarize a local file
summarize "/workspace/report.pdf"

# Summarize a YouTube video
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto

# Get a raw transcript without summarizing
summarize "https://example.com" --extract-only

# Specify a model
summarize "https://example.com" --model google/gemini-2-flash

# JSON output
summarize "https://example.com" --json
```

## Gotchas

- **YouTube transcripts**: Quality depends on whether the video has captions. Auto-generated captions vary. Use `--extract-only` if the summary is missing key information.
- **PDFs**: Large PDFs may exceed model context limits. The tool handles chunking, but very long documents may produce incomplete summaries.
- **Model selection**: If no `--model` is specified, `summarize` picks from whichever keys are available. Gemini models tend to work well for YouTube; Claude or GPT for text.
- **Network access**: `summarize` calls external APIs, so the gateway must have outbound internet access (`nachos-egress` network).

## What's next?

- [Skills overview](/skills/index) — all available skills
- [Browser tool](/tools/browser) — when you need to interact with pages rather than just read them
