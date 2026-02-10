---
title: "Docker Setup"
description: "Make sure Docker is running before starting Nachos."
---

# Docker Setup

Nachos runs entirely in Docker containers orchestrated by Docker Compose. There is no native install path — Docker is required.

## Install Docker

Choose one:

- **Docker Desktop** (macOS / Windows / Linux) — includes Docker Engine and Compose
- **Docker Engine** (Linux) — install Engine and the Compose plugin separately

## Verify your installation

```bash
docker info
docker compose version
```

Both commands must succeed. Docker Compose v2+ is required (the `docker compose` subcommand, not the legacy `docker-compose` binary).

## Resource recommendations

Nachos runs several containers simultaneously. Recommended minimums:

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU      | 2 cores | 4 cores     |
| Memory   | 2 GB    | 4 GB        |
| Disk     | 5 GB    | 10 GB       |

If you're using Docker Desktop, check **Settings > Resources** to adjust limits.

## Network requirements

Nachos creates two internal Docker networks:

- `nachos-internal` — isolated, no external access
- `nachos-egress` — controlled external access for LLM APIs and channel APIs

No manual network configuration is needed. `nachos up` creates these automatically.

## Gotchas

- **Docker daemon not running**: `nachos up` will fail with a connection error. Start Docker Desktop or run `sudo systemctl start docker`.
- **Port conflicts**: The default webchat port is 8080. If it's taken, change `channels.webchat.port` in `nachos.toml`.
- **Firewall / proxy**: If your Docker host sits behind a corporate proxy, configure Docker's proxy settings so containers can reach LLM APIs.
