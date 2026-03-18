---
title: "sessions_spawn"
description: "Spawn background tasks with isolated AI agents."
---

# sessions_spawn

The `sessions_spawn` tool creates an independent AI agent to handle complex, time-consuming tasks in the background while your main conversation continues.

## When to use it

<Tip>
Think of subagents as background workers. If a task takes more than 30 seconds or requires multiple steps, spawn a subagent.
</Tip>

**Good use cases:**
- Research and analysis (e.g., "Research quantum computing trends and write a summary")
- Code audits and reviews (e.g., "Analyze this codebase for security vulnerabilities")
- Long-running operations (e.g., "Generate comprehensive test suite")
- Parallel work (e.g., run multiple analyses simultaneously)

**Don't use for:**
- Simple questions ("What is 2+2?")
- Quick lookups ("What's the weather?")
- Interactive conversations (back-and-forth works better in main session)

## Basic usage

```typescript
await sessions_spawn({
  task: "Research renewable energy trends in 2025 and write a 3-paragraph summary"
});
```

**Response:**
```json
{
  "status": "accepted",
  "runId": "run-abc123",
  "childSessionId": "session-child-xyz",
  "model": "anthropic.claude-sonnet-4-6"
}
```

Your conversation continues immediately. The subagent works in the background and announces results when complete.

## Parameters

### task <span style="color: red;">*</span>

**Type:** `string`

Detailed description of what the subagent should do. Be specific!

<CodeGroup>
```typescript Bad
task: "Research stuff"
```

```typescript Good
task: "Research renewable energy trends in 2025, focusing on solar and wind. Summarize key findings in 3 paragraphs with sources."
```
</CodeGroup>

### model

**Type:** `string` (alias or full model ID)

Choose which AI model to use. Different models offer different trade-offs between speed, cost, and capability.

**Available models:**
- `haiku` - Fast and economical (3× cheaper than Sonnet)
- `sonnet` - Balanced performance and cost (default)
- `opus` - Most capable (5× more expensive than Haiku)

**Examples:**
```typescript
// Quick syntax check
await sessions_spawn({
  task: "Check Python syntax in all .py files",
  model: "haiku"
});

// Deep analysis
await sessions_spawn({
  task: "Review system architecture for scaling bottlenecks",
  model: "opus"
});
```

<Note>
If you don't specify a model, Nachos automatically selects one based on task complexity.
</Note>

### modelHint

**Type:** `'fast' | 'balanced' | 'thorough'`

Suggest a model tier without specifying the exact model.

- `fast` → Haiku (simple tasks)
- `balanced` → Sonnet (default)
- `thorough` → Opus (complex analysis)

**Example:**
```typescript
await sessions_spawn({
  task: "Conduct comprehensive security audit",
  modelHint: "thorough"
});
```

<Info>
`modelHint` is ignored if you explicitly set `model`.
</Info>

### stream

**Type:** `boolean` (default: `false`)

Enable streaming to see partial results as they're generated.

**When to use:**
- Long reports or documentation
- You want to see progress before completion
- Output is >1000 words

**Example:**
```typescript
await sessions_spawn({
  task: "Generate a 50-page security audit report",
  model: "opus",
  stream: true
});
```

You'll receive chunks of the report as they're written, rather than waiting for the full document.

### label

**Type:** `string`

Human-readable label for easy identification.

**Example:**
```typescript
await sessions_spawn({
  task: "Analyze user authentication flow",
  label: "auth-security-audit-2026"
});
```

### runTimeoutSeconds

**Type:** `number` (default: `300`)

Maximum execution time in seconds.

**Examples:**
```typescript
runTimeoutSeconds: 120   // 2 minutes (quick tasks)
runTimeoutSeconds: 600   // 10 minutes (complex tasks)
runTimeoutSeconds: 1800  // 30 minutes (very long tasks)
```

<Warning>
Tasks exceeding the timeout are automatically terminated.
</Warning>

### cleanup

**Type:** `'delete' | 'keep'` (default: `'keep'`)

What to do with the subagent's workspace after completion.

- `keep` - Preserve workspace files (you can access them later)
- `delete` - Remove workspace immediately after completion

**Example:**
```typescript
await sessions_spawn({
  task: "Generate test coverage report",
  cleanup: "keep"  // Keep the report files
});
```

## Auto model selection

If you don't specify `model` or `modelHint`, Nachos analyzes your task and picks the best model:

**→ Opus (thorough)** for:
- Tasks with keywords: *analyze, review, audit, investigate, comprehensive*
- Code analysis: *codebase, vulnerabilities, refactor, bugs*
- Multi-step tasks with numbered lists
- Long tasks (>40 words)

**→ Haiku (fast)** for:
- Short, simple tasks (&lt;8 words)
- No complexity keywords

**→ Sonnet (balanced)** for:
- Everything else (default)

**Examples:**
```typescript
// Auto-selects Opus (has "analyze" + "codebase")
await sessions_spawn({
  task: "Analyze this codebase for security vulnerabilities"
});

// Auto-selects Haiku (short + simple)
await sessions_spawn({
  task: "Fix typo in README"
});

// Auto-selects Sonnet (medium complexity)
await sessions_spawn({
  task: "Summarize this research paper"
});
```

## Complete example

```typescript
await sessions_spawn({
  task: "Research quantum computing trends in 2025 and write a detailed report with:\n" +
        "1. Current state of hardware\n" +
        "2. Major breakthroughs\n" +
        "3. Industry applications\n" +
        "4. Challenges and future outlook",
  label: "quantum-computing-research",
  model: "sonnet",
  stream: true,
  runTimeoutSeconds: 600,
  cleanup: "keep"
});
```

## Monitoring progress

Check on your subagent using the [`subagents`](/tools/subagents) tool:

```typescript
// List all running subagents
await subagents({ action: 'list' });

// Get detailed info
await subagents({ 
  action: 'info', 
  runId: 'run-abc123' 
});
```

## Next steps

<CardGroup cols={2}>
  <Card title="Manage subagents" icon="list-check" href="/tools/subagents">
    Monitor, steer, and stop running subagents
  </Card>
  <Card title="Build workflows" icon="diagram-project" href="/tools/sessions-orchestrate">
    Chain multiple subagents together
  </Card>
  <Card title="Report progress" icon="chart-line" href="/tools/subagent-progress">
    Show status updates from within subagents
  </Card>
</CardGroup>
