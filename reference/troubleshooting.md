---
title: "Troubleshooting"
description: "Common errors and fixes."
---

# Troubleshooting

## First steps

```bash
nachos status       # Check container health
nachos logs         # Tail all service logs
nachos doctor       # Run diagnostic checks
docker info         # Verify Docker is running
```

## Common issues

### `nachos up` fails with "Cannot connect to Docker daemon"

Docker is not running. Start Docker Desktop or the Docker daemon:

```bash
# Linux
sudo systemctl start docker

# macOS/Windows
# Open Docker Desktop
```

### Containers show `unhealthy`

Check the logs for the unhealthy container:

```bash
nachos logs gateway
```

Common causes:

- **Missing env vars**: API key not set in `.env`. Check with `nachos config validate`.
- **Port conflict**: Another process is using port 8080 (or whichever port you configured). Change `channels.webchat.port` in `nachos.toml`.
- **Bus not ready**: If the bus fails to start, all other containers will fail health checks. Check `nachos logs bus` first.

### "ANTHROPIC_API_KEY is not set"

Add the key to your `.env` file:

```bash
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

Then restart: `nachos restart`.

### Slack bot not responding

- Verify `channels.slack.enabled = true` in `nachos.toml`
- Check that `SLACK_BOT_TOKEN` and `SLACK_APP_TOKEN` are set in `.env`
- Confirm the bot is invited to the channels listed in `channel_ids`
- Check logs: `nachos logs slack`

### Tool calls failing

- Check that the tool is enabled in `nachos.toml`
- Verify the security mode allows the tool: `nachos policy validate`
- Check tool-specific logs: `nachos logs browser` (or `filesystem`, `code-runner`)

### Config validation errors

```bash
nachos config validate
```

Fix the reported errors in `nachos.toml`. Common issues:

- Typos in section names (`[tool.browser]` instead of `[tools.browser]`)
- Missing required fields (e.g., `provider` in `[llm]`)
- Invalid enum values (e.g., `mode = "safe"` instead of `mode = "strict"`)

### High memory usage

- Disable unused tools (especially `browser` — Chromium is memory-hungry)
- Lower `runtime.resources.memory` per container
- Reduce `max_memory` for the code runner
- Run `docker system prune` to clean up unused images and volumes

## Getting more detail

Set debug logging in `nachos.toml`:

```toml
[runtime]
log_level = "debug"
log_format = "json"
```

Then restart and check logs. JSON format is useful for piping to `jq` or external log aggregators.
