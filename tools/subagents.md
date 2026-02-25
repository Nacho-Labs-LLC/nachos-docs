---
title: "subagents"
description: "Monitor and manage background tasks and workflows."
---

# subagents

The `subagents` tool lets you monitor, control, and interact with running background tasks (subagents) and multi-step workflows.

## Actions

The tool supports multiple actions for different management tasks:

- [`list`](#list) - List all your subagent runs
- [`info`](#info) - Get detailed information about a specific run
- [`log`](#log) - View conversation logs
- [`stop`](#stop) - Stop a queued subagent
- [`steer`](#steer) - Send instructions to a running subagent
- [`files_list`](#files_list) - List workspace files
- [`files_get`](#files_get) - Read workspace files
- [`workflow_list`](#workflow_list) - List all workflows
- [`workflow_info`](#workflow_info) - Get workflow status and results

## list

List all subagent runs for your current session.

```typescript
await subagents({ action: 'list' });
```

**Response:**
```json
{
  "runs": [
    {
      "runId": "run-abc123",
      "status": "running",
      "task": "Analyze codebase for security vulnerabilities",
      "label": "security-audit",
      "model": "anthropic.claude-opus-4-6-v1",
      "stream": true,
      "createdAt": "2026-02-24T14:00:00Z",
      "startedAt": "2026-02-24T14:00:05Z",
      "progress": [
        {
          "timestamp": "2026-02-24T14:02:00Z",
          "status": "Analyzing file 23 of 50",
          "percentage": 46
        }
      ]
    },
    {
      "runId": "run-def456",
      "status": "completed",
      "task": "Generate test suite",
      "label": "test-generation",
      "model": "anthropic.claude-sonnet-4-6",
      "completedAt": "2026-02-24T13:45:00Z",
      "durationMs": 45000
    }
  ]
}
```

**Status values:**
- `queued` - Waiting to start
- `running` - Currently executing
- `completed` - Successfully finished
- `failed` - Encountered an error
- `cancelled` - Stopped by user

## info

Get detailed information about a specific subagent run.

```typescript
await subagents({ 
  action: 'info', 
  runId: 'run-abc123' 
});
```

**Response:**
```json
{
  "runId": "run-abc123",
  "status": "running",
  "task": "Analyze codebase for security vulnerabilities",
  "label": "security-audit",
  "model": "anthropic.claude-opus-4-6-v1",
  "stream": true,
  "createdAt": "2026-02-24T14:00:00Z",
  "startedAt": "2026-02-24T14:00:05Z",
  "progress": [
    {
      "timestamp": "2026-02-24T14:00:30Z",
      "status": "Starting analysis",
      "percentage": 0
    },
    {
      "timestamp": "2026-02-24T14:02:00Z",
      "status": "Analyzing file 23 of 50",
      "percentage": 46,
      "metadata": {
        "filesProcessed": 23,
        "totalFiles": 50,
        "issuesFound": 3
      }
    }
  ],
  "streamChunks": [
    "# Security Analysis Report\n\n",
    "## Executive Summary\n\n",
    "Found 3 critical vulnerabilities:\n",
    "1. SQL injection in auth.py\n",
    "..."
  ],
  "workspaceDir": "/workspace/run-abc123"
}
```

<Info>
**New in v2.0:** The `info` response now includes `model`, `stream`, `progress`, and `streamChunks` fields for better visibility.
</Info>

## log

Retrieve the conversation log for a subagent.

```typescript
await subagents({ 
  action: 'log', 
  runId: 'run-abc123',
  limit: 50  // Optional: default 50
});
```

**Response:**
```json
{
  "runId": "run-abc123",
  "messages": [
    {
      "role": "user",
      "content": "Analyze codebase for security vulnerabilities"
    },
    {
      "role": "assistant",
      "content": "I'll analyze the codebase systematically...",
      "toolCalls": [ /* ... */ ]
    },
    // ... more messages
  ]
}
```

## stop

Stop a **queued** subagent before it starts.

<Warning>
You can only stop subagents that are still queued. Running subagents cannot be stopped (they'll complete or timeout).
</Warning>

```typescript
await subagents({ 
  action: 'stop', 
  runId: 'run-def456' 
});
```

**Response:**
```json
{
  "success": true,
  "message": "Subagent run-def456 stopped"
}
```

**Error if already running:**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_STATE",
    "message": "Cannot stop run that is already running"
  }
}
```

## steer

Send additional instructions to a **running** subagent.

```typescript
await subagents({ 
  action: 'steer', 
  runId: 'run-abc123',
  message: 'Focus on OWASP Top 10 vulnerabilities only'
});
```

**Response:**
```json
{
  "success": true,
  "message": "Steering message sent to subagent"
}
```

The subagent receives your message and adjusts its approach accordingly.

**Good steering messages:**
- "Focus on X instead of Y"
- "Add more detail about Z"
- "Skip section A and move to B"
- "Prioritize recent sources (2024-2025)"
- "Wrap up and provide current findings"

<Tip>
Steering is useful for redirecting long-running tasks without waiting for them to complete.
</Tip>

## files_list

List files in a subagent's workspace.

```typescript
await subagents({ 
  action: 'files_list', 
  runId: 'run-abc123' 
});
```

**Response:**
```json
{
  "runId": "run-abc123",
  "workspace": "/workspace/run-abc123",
  "files": [
    {
      "name": "research-notes.md",
      "size": 2345,
      "type": "file"
    },
    {
      "name": "sources.txt",
      "size": 1123,
      "type": "file"
    },
    {
      "name": "data",
      "type": "directory"
    }
  ]
}
```

<Note>
Workspaces are only available for subagents configured with workspace isolation.
</Note>

## files_get

Read a file from a subagent's workspace.

```typescript
await subagents({ 
  action: 'files_get', 
  runId: 'run-abc123',
  relativePath: 'research-notes.md'
});
```

**Response:**
```json
{
  "runId": "run-abc123",
  "path": "research-notes.md",
  "content": "# Research Notes\n\n## Renewable Energy Trends...",
  "size": 2345
}
```

<Warning>
Path traversal attempts (`../`, absolute paths) are blocked for security.
</Warning>

## workflow_list

List all workflows for your current session.

```typescript
await subagents({ action: 'workflow_list' });
```

**Response:**
```json
{
  "workflows": [
    {
      "workflowId": "workflow-xyz789",
      "status": "running",
      "label": "data-analysis-workflow",
      "stepCount": 5,
      "currentBatch": 2,
      "totalBatches": 4,
      "createdAt": "2026-02-24T14:00:00Z",
      "startedAt": "2026-02-24T14:00:05Z"
    },
    {
      "workflowId": "workflow-abc123",
      "status": "completed",
      "label": "bug-fix-workflow",
      "stepCount": 5,
      "totalBatches": 5,
      "completedAt": "2026-02-24T13:30:00Z",
      "durationMs": 120000
    }
  ]
}
```

## workflow_info

Get detailed workflow status and step results.

```typescript
await subagents({ 
  action: 'workflow_info', 
  workflowId: 'workflow-xyz789' 
});
```

**Response:**
```json
{
  "workflowId": "workflow-xyz789",
  "status": "running",
  "label": "data-analysis-workflow",
  "currentBatch": 2,
  "totalBatches": 4,
  "stepResults": {
    "fetch": {
      "stepId": "fetch",
      "runId": "run-001",
      "status": "completed",
      "result": "Fetched 150 records from API",
      "durationMs": 2500
    },
    "validate": {
      "stepId": "validate",
      "runId": "run-002",
      "status": "completed",
      "result": "All records valid, no errors found",
      "durationMs": 1200
    },
    "analyze": {
      "stepId": "analyze",
      "runId": "run-003",
      "status": "running",
      "progress": [
        {
          "timestamp": "2026-02-24T14:05:00Z",
          "status": "Processing batch 2 of 5",
          "percentage": 40
        }
      ]
    },
    "report": {
      "stepId": "report",
      "status": "queued"
    }
  },
  "createdAt": "2026-02-24T14:00:00Z",
  "startedAt": "2026-02-24T14:00:05Z"
}
```

**Step statuses:**
- `queued` - Waiting for dependencies to complete
- `running` - Currently executing
- `completed` - Finished successfully
- `failed` - Encountered an error

<Info>
Each completed step includes its result text, which is automatically passed to dependent steps.
</Info>

## Access control

<Note>
You can only access your own subagents and workflows. Attempting to access another user's runs returns "not found" errors.
</Note>

Access is granted if:
- You're in the same session that spawned the subagent
- OR you're the same user (across different sessions)

## Common patterns

### Check on long-running task

```typescript
// Spawn a long task
const spawn = await sessions_spawn({
  task: "Comprehensive security audit of entire codebase",
  runTimeoutSeconds: 1800  // 30 minutes
});

// ... do other work ...

// Check progress later
const status = await subagents({
  action: 'info',
  runId: spawn.runId
});

console.log(status.progress); // See latest progress updates
```

### Monitor workflow execution

```typescript
// Start workflow
const workflow = await sessions_orchestrate({
  steps: [ /* ... */ ],
  label: 'data-pipeline'
});

// Poll for completion
const checkStatus = async () => {
  const info = await subagents({
    action: 'workflow_info',
    workflowId: workflow.workflowId
  });
  
  if (info.status === 'completed') {
    console.log('Workflow complete!');
    console.log(info.stepResults);
  } else {
    console.log(`Batch ${info.currentBatch} of ${info.totalBatches}`);
  }
};
```

### Read generated report

```typescript
// After subagent completes
const files = await subagents({
  action: 'files_list',
  runId: 'run-abc123'
});

// Read the report
const report = await subagents({
  action: 'files_get',
  runId: 'run-abc123',
  relativePath: 'security-report.md'
});

console.log(report.content);
```

## Next steps

<CardGroup cols={3}>
  <Card title="Spawn subagents" icon="rocket" href="/tools/sessions-spawn">
    Create background tasks
  </Card>
  <Card title="Build workflows" icon="diagram-project" href="/tools/sessions-orchestrate">
    Multi-step orchestration
  </Card>
  <Card title="Report progress" icon="chart-line" href="/tools/subagent-progress">
    Update status from subagents
  </Card>
</CardGroup>
