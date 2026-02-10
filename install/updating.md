---
title: "Updating"
description: "Update the CLI and rebuild the stack."
---

# Updating

## Update the CLI

```bash
pnpm add -g @nachos/cli@latest
nachos --version
```

## Rebuild containers

After a CLI update, rebuild to pick up new container images:

```bash
nachos down
nachos up --build
```

This pulls the latest base images and rebuilds all containers from scratch.

## Check for breaking changes

Before updating across major versions, check the release notes for migration steps. Config format changes in `nachos.toml` will be flagged by:

```bash
nachos config validate
```

## Rollback

If an update breaks your stack:

1. Install the previous CLI version: `pnpm add -g @nachos/cli@<version>`
2. Rebuild: `nachos down && nachos up --build`

Your `nachos.toml` and `.env` files are not modified by updates, so rolling back the CLI is safe.

## Gotchas

- **Always `nachos down` before `nachos up --build`**: Running `--build` on a live stack can cause transient errors during the rebuild.
- **Docker image cache**: If you suspect stale images, run `docker system prune` to clear the cache before rebuilding.
