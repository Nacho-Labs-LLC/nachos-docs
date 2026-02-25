---
title: "sessions_orchestrate"
description: "Build multi-step workflows with dependencies."
---

# sessions_orchestrate

Define complex workflows with multiple steps that execute in a specific order, with dependencies and parallel execution.

## When to use workflows

<Tip>
Use workflows when your task naturally breaks down into sequential or parallel steps with dependencies.
</Tip>

**Perfect for:**
- **Sequential processes**: Research → Analyze → Summarize
- **Parallel execution**: Run tests + Build docs + Security scan (simultaneously)
- **Diamond patterns**: Fetch data → [Validate, Enrich] → Merge results
- **Complex pipelines**: Multi-step data processing or analysis

**Example scenarios:**
- Bug fix: Investigate → Fix → Test → Review → Document
- Research synthesis: [Web search, Docs search, Code search] → Combine findings
- Data pipeline: Fetch → Validate → Transform → Load

## Basic usage

```typescript
await sessions_orchestrate({
  steps: [
    {
      id: 'research',
      task: 'Research renewable energy trends in 2025'
    },
    {
      id: 'analyze',
      task: 'Analyze the research findings',
      dependsOn: ['research']
    },
    {
      id: 'summarize',
      task: 'Write a 2-paragraph summary',
      dependsOn: ['analyze']
    }
  ],
  label: 'renewable-energy-report'
});
```

**Response:**
```json
{
  "status": "accepted",
  "workflowId": "workflow-xyz789",
  "stepCount": 3,
  "batches": [
    ["research"],
    ["analyze"],
    ["summarize"]
  ]
}
```

The workflow executes steps in order. Results from earlier steps are automatically passed to dependent steps.

## Workflow patterns

### Linear (sequential)

Each step waits for the previous one to complete.

```typescript
await sessions_orchestrate({
  steps: [
    { id: 'fetch', task: 'Fetch user data' },
    { id: 'validate', task: 'Validate data', dependsOn: ['fetch'] },
    { id: 'process', task: 'Process data', dependsOn: ['validate'] }
  ]
});
```

**Execution:**
```
Batch 1: fetch
Batch 2: validate
Batch 3: process
```

### Parallel

Independent steps run simultaneously.

```typescript
await sessions_orchestrate({
  steps: [
    { id: 'web', task: 'Search web for quantum computing' },
    { id: 'docs', task: 'Search internal docs' },
    { id: 'code', task: 'Search codebase' },
    { 
      id: 'synthesis', 
      task: 'Combine all findings',
      dependsOn: ['web', 'docs', 'code']
    }
  ]
});
```

**Execution:**
```
Batch 1 (parallel): web, docs, code
Batch 2: synthesis (waits for all 3)
```

### Diamond

Split work, then merge results.

```typescript
await sessions_orchestrate({
  steps: [
    { id: 'fetch', task: 'Fetch customer data' },
    { 
      id: 'validate', 
      task: 'Validate data integrity',
      dependsOn: ['fetch']
    },
    { 
      id: 'enrich', 
      task: 'Enrich with external sources',
      dependsOn: ['fetch']
    },
    { 
      id: 'merge', 
      task: 'Merge validated and enriched data',
      dependsOn: ['validate', 'enrich']
    }
  ]
});
```

**Execution:**
```
Batch 1: fetch
Batch 2 (parallel): validate, enrich
Batch 3: merge
```

## Step parameters

Each workflow step supports these fields:

### id <span style="color: red;">*</span>

**Type:** `string`

Unique identifier for this step. Used for dependencies and result tracking.

```typescript
id: 'fetch-data'
id: 'analyze-results'
id: 'generate-report'
```

<Warning>
Step IDs must be unique within a workflow. Duplicate IDs will cause validation errors.
</Warning>

### task <span style="color: red;">*</span>

**Type:** `string`

Description of what this step should do. Results from dependent steps are automatically appended.

```typescript
task: 'Analyze the security vulnerabilities in the authentication module'
```

### dependsOn

**Type:** `string[]` (array of step IDs)

Which steps must complete before this one can start.

```typescript
dependsOn: ['fetch']          // Wait for one step
dependsOn: ['validate', 'enrich']  // Wait for multiple steps
```

### model

**Type:** `string`

AI model for this step (see [sessions_spawn](/tools/sessions-spawn#model) for options).

Different steps can use different models:

```typescript
steps: [
  { 
    id: 'fetch', 
    task: 'Fetch data from API',
    model: 'haiku'  // Fast, cheap
  },
  { 
    id: 'analyze', 
    task: 'Deep security analysis',
    dependsOn: ['fetch'],
    model: 'opus'  // Thorough, expensive
  }
]
```

### modelHint

**Type:** `'fast' | 'balanced' | 'thorough'`

Model suggestion (see [sessions_spawn](/tools/sessions-spawn#modelhint)).

```typescript
{
  id: 'audit',
  task: 'Comprehensive security audit',
  modelHint: 'thorough'
}
```

### stream

**Type:** `boolean`

Enable streaming for this step's output.

```typescript
{
  id: 'report',
  task: 'Generate 50-page report',
  stream: true
}
```

## Result passing

Results from dependent steps are automatically injected into the task prompt.

**Workflow:**
```typescript
{
  steps: [
    {
      id: 'research',
      task: 'Research solar energy trends'
    },
    {
      id: 'analyze',
      task: 'Identify the top 3 trends',
      dependsOn: ['research']
    }
  ]
}
```

**What the 'analyze' step receives:**
```
Task: Identify the top 3 trends

**Results from 'research':**
Solar capacity grew 45% in 2025, driven by...
```

<Info>
You don't need to manually reference previous results — they're automatically included!
</Info>

## Workflow parameters

### steps <span style="color: red;">*</span>

**Type:** `WorkflowStep[]`

Array of workflow steps (see [Step parameters](#step-parameters) above).

### label

**Type:** `string`

Human-readable name for this workflow.

```typescript
label: 'bug-fix-workflow-issue-123'
label: 'customer-data-enrichment'
```

### continueOnFailure

**Type:** `boolean` (default: `false`)

Whether to continue if a step fails.

- `false` (default) - Stop the entire workflow on first failure
- `true` - Continue executing independent steps even if some fail

```typescript
await sessions_orchestrate({
  steps: [ /* ... */ ],
  continueOnFailure: true  // Keep going despite failures
});
```

## Real-world example: Bug fix workflow

```typescript
await sessions_orchestrate({
  steps: [
    {
      id: 'investigate',
      task: 'Find root cause of login bug in auth service',
      modelHint: 'thorough'
    },
    {
      id: 'fix',
      task: 'Implement the fix based on investigation',
      dependsOn: ['investigate'],
      model: 'sonnet'
    },
    {
      id: 'tests',
      task: 'Write comprehensive tests for the fix',
      dependsOn: ['fix'],
      model: 'haiku'
    },
    {
      id: 'review',
      task: 'Code review the changes and tests',
      dependsOn: ['fix', 'tests'],
      modelHint: 'thorough'
    },
    {
      id: 'document',
      task: 'Update documentation with fix details',
      dependsOn: ['review'],
      model: 'sonnet',
      stream: true
    }
  ],
  label: 'fix-login-bug'
});
```

**Execution plan:**
```
Batch 1: investigate (Opus)
Batch 2: fix (Sonnet)
Batch 3: tests (Haiku)
Batch 4: review (Opus, waits for both fix AND tests)
Batch 5: document (Sonnet, streaming)
```

## Validation

Nachos validates your workflow before execution:

<AccordionGroup>
  <Accordion title="✅ Valid workflows">
    - No duplicate step IDs
    - All dependencies reference existing steps
    - No circular dependencies
    - Within step count limits (default: 50 steps)
  </Accordion>

  <Accordion title="❌ Common errors">
    **Duplicate step IDs:**
    ```typescript
    // ❌ Error: DUPLICATE_STEP_ID
    steps: [
      { id: 'step1', task: '...' },
      { id: 'step1', task: '...' }
    ]
    ```

    **Missing dependencies:**
    ```typescript
    // ❌ Error: MISSING_DEPENDENCY
    steps: [
      { id: 'step1', task: '...' },
      { id: 'step2', task: '...', dependsOn: ['step3'] }
      // step3 doesn't exist!
    ]
    ```

    **Circular dependencies:**
    ```typescript
    // ❌ Error: CYCLE_DETECTED
    steps: [
      { id: 'a', task: '...', dependsOn: ['b'] },
      { id: 'b', task: '...', dependsOn: ['a'] }
    ]
    ```
  </Accordion>
</AccordionGroup>

## Monitoring workflows

Track workflow progress with the [`subagents`](/tools/subagents) tool:

```typescript
// List all workflows
await subagents({ action: 'workflow_list' });

// Get detailed status
await subagents({ 
  action: 'workflow_info', 
  workflowId: 'workflow-xyz789' 
});
```

**Response shows step results:**
```json
{
  "workflowId": "workflow-xyz789",
  "status": "running",
  "currentBatch": 2,
  "totalBatches": 4,
  "stepResults": {
    "fetch": {
      "status": "completed",
      "result": "Fetched 150 records",
      "durationMs": 2500
    },
    "analyze": {
      "status": "running"
    },
    "report": {
      "status": "queued"
    }
  }
}
```

## Configuration

Admins can configure workflow limits in `nachos.toml`:

```toml
[gateway.subagent.workflows]
max_steps = 50     # Maximum steps per workflow
max_depth = 5      # Maximum nested workflow depth
```

## Next steps

<CardGroup cols={2}>
  <Card title="Spawn subagents" icon="rocket" href="/tools/sessions-spawn">
    Learn about individual background tasks
  </Card>
  <Card title="Manage workflows" icon="list-check" href="/tools/subagents">
    Monitor and control workflow execution
  </Card>
</CardGroup>
