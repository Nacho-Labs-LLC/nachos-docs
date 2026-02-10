---
title: "FAQ"
description: "Frequently asked questions."
---

# FAQ

**Does Nachos require Docker?**

Yes. Every component runs in a container. There is no native install path.

**Can I run multiple channels at once?**

Yes. Enable as many channels as you want in `nachos.toml`. Each runs in its own container and communicates independently with the gateway.

**Can I use a local LLM instead of an API?**

Yes. Set `llm.provider = "ollama"` and point `base_url` to your Ollama instance. See [LLM Proxy](/architecture/llm-proxy) for configuration.

**How do I change the LLM model?**

Edit `[llm].model` in `nachos.toml` and run `nachos restart`.

**Is my data sent to third parties?**

Only to the LLM provider you configure (Anthropic, OpenAI, etc.). Nachos itself does not phone home or collect telemetry. With Ollama, everything stays local.

**Can I run Nachos on a server?**

Yes. Nachos works on any machine with Docker. For channels that use websocket/long-polling (Slack socket mode, Telegram), no public URL is needed. For Slack HTTP mode, the server needs a public URL.

**How do I reset a conversation?**

Users can type `/reset` or `/new` in any channel (if commands are enabled). This clears the session state. You can also configure custom reset triggers in `nachos.toml`:

```toml
[runtime.context_management.commands]
reset_triggers = ["/new", "/reset", "!new", "!reset"]
```

**Can I customize the assistant's personality?**

Yes. Edit the `[assistant]` section in `nachos.toml`:

```toml
[assistant]
name = "MyBot"
system_prompt = "You are a helpful coding assistant."
```

**How do I add a custom tool?**

Scaffold a new tool container with `nachos create tool my-tool`, implement the tool interface, define the manifest, and register it with the gateway. See the [CLAUDE.md](/CLAUDE.md) project docs for the full process.

**What happens if the LLM provider goes down?**

If you've configured `fallback_order` in `[llm]`, the llm-proxy automatically tries the next provider. Without a fallback, the assistant returns an error to the user.

**How do I upgrade?**

```bash
pnpm add -g @nachos/cli@latest
nachos down
nachos up --build
```

See [Updating](/install/updating) for details.
