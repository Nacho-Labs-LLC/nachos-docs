---
title: "CLI Install"
description: "Install the Nachos CLI globally."
---

# CLI Install

The Nachos CLI is a Node.js package that manages your entire stack — init, up, down, logs, config validation, and more.

## Install with pnpm

```bash
pnpm add -g @nachos/cli
```

## Verify

```bash
nachos --version
```

You should see the installed version number.

## Alternative package managers

```bash
# npm
npm install -g @nachos/cli

# yarn
yarn global add @nachos/cli
```

## Shell completions

Generate completions for your shell:

```bash
# bash
nachos completions bash >> ~/.bashrc

# zsh
nachos completions zsh >> ~/.zshrc
```

## Gotchas

- **Permission errors on global install**: Use `sudo` or fix your npm/pnpm global directory permissions. Avoid running the CLI itself as root.
- **Command not found after install**: Ensure your pnpm global bin directory is in your `PATH`. Run `pnpm bin -g` to find it.
- **Multiple Node versions**: If you use nvm or fnm, make sure Node 22+ is active before installing.
