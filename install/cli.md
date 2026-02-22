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

Generate and install a completion script for your shell:

```bash
# Bash (Linux)
nachos completion bash | sudo tee /etc/bash_completion.d/nachos > /dev/null

# Bash (macOS with Homebrew)
nachos completion bash > "$(brew --prefix)/etc/bash_completion.d/nachos"

# Zsh
mkdir -p ~/.zsh/completion
nachos completion zsh > ~/.zsh/completion/_nachos
echo 'fpath=(~/.zsh/completion $fpath)' >> ~/.zshrc

# Fish
nachos completion fish > ~/.config/fish/completions/nachos.fish

# PowerShell
nachos completion powershell | Out-String | Invoke-Expression
```

Then restart your shell or source your config file (e.g. `source ~/.zshrc`).

## Gotchas

- **Permission errors on global install**: Use `sudo` or fix your npm/pnpm global directory permissions. Avoid running the CLI itself as root.
- **Command not found after install**: Ensure your pnpm global bin directory is in your `PATH`. Run `pnpm bin -g` to find it.
- **Multiple Node versions**: If you use nvm or fnm, make sure Node 22+ is active before installing.
