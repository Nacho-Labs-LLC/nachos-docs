---
title: "subagent_progress"
description: "Report progress updates from running subagents."
---

# subagent_progress

<Note>
This tool is only available **within subagents**. Main sessions cannot call it.
</Note>

The `subagent_progress` tool allows running subagents to report status updates, percentage completion, and structured metadata about their progress.

## Why report progress?

For long-running tasks (>1 minute), progress updates:
- Give users confidence the subagent is working
- Show estimated completion
- Enable early intervention if something's wrong
- Provide real-time visibility into what's happening

## Basic usage

From within a subagent:

```typescript
await subagent_progress({
  status: "Analyzing file 23 of 50",
  percentage: 46
});
```

Users can see this update by checking the subagent's status:

```typescript
await subagents({ 
  action: 'info', 
  runId: 'run-abc123' 
});
```

## Parameters

### status <span style="color: red;">*</span>

**Type:** `string`

Human-readable description of current progress.

<CodeGroup>
```typescript Bad
status: "Working..."
```

```typescript Good
status: "Analyzing file 23 of 50"
status: "Running security scan (step 2/5)"
status: "Fetching data from API (batch 3/10)"
```
</CodeGroup>

<Tip>
Be specific! Include concrete numbers and phase information.
</Tip>

### percentage

**Type:** `number` (0-100)

Progress percentage (optional but recommended).

```typescript
percentage: 0    // Just started
percentage: 46   // 46% complete
percentage: 100  // Finished
```

### metadata

**Type:** `Record<string, unknown>`

Structured data for programmatic tracking (optional).

```typescript
metadata: {
  filesProcessed: 23,
  totalFiles: 50,
  issuesFound: 3,
  currentPhase: "static-analysis"
}
```

<Info>
Metadata is useful for building dashboards or tracking specific metrics.
</Info>

## Examples

### File processing loop

```typescript
for (let i = 0; i < files.length; i++) {
  await analyzeFile(files[i]);
  
  // Report progress every 10 files
  if (i % 10 === 0 || i === files.length - 1) {
    await subagent_progress({
      status: `Analyzed ${i + 1} of ${files.length} files`,
      percentage: Math.floor(((i + 1) / files.length) * 100),
      metadata: {
        filesProcessed: i + 1,
        totalFiles: files.length
      }
    });
  }
}
```

### Phase transitions

```typescript
// Phase 1: Data collection
await subagent_progress({
  status: "Phase 1: Collecting data",
  percentage: 0,
  metadata: { phase: "collection" }
});

// ... work ...

await subagent_progress({
  status: "Phase 1: Data collection complete",
  percentage: 25,
  metadata: { 
    phase: "collection", 
    recordsCollected: 150 
  }
});

// Phase 2: Validation
await subagent_progress({
  status: "Phase 2: Validating data",
  percentage: 25,
  metadata: { phase: "validation" }
});

// ... work ...

await subagent_progress({
  status: "Phase 2: Validation complete",
  percentage: 50,
  metadata: { 
    phase: "validation",
    recordsValidated: 150,
    errors: 0
  }
});

// Phase 3: Analysis
await subagent_progress({
  status: "Phase 3: Running analysis",
  percentage: 50,
  metadata: { phase: "analysis" }
});
```

### Security audit with details

```typescript
let criticalIssues = 0;
let warnings = 0;

for (const file of codeFiles) {
  const results = await scanFile(file);
  criticalIssues += results.critical.length;
  warnings += results.warnings.length;
  
  await subagent_progress({
    status: `Scanning ${file.name} (${filesScanned}/${totalFiles})`,
    percentage: Math.floor((filesScanned / totalFiles) * 100),
    metadata: {
      filesScanned,
      totalFiles,
      criticalIssues,
      warnings,
      currentFile: file.name
    }
  });
  
  filesScanned++;
}
```

## Throttling

<Warning>
Progress updates are automatically throttled to **1 second minimum** between calls.
</Warning>

If you call `subagent_progress` more frequently, extra calls are silently ignored.

```typescript
// ❌ Bad: Too frequent (throttled)
for (let i = 0; i < 10000; i++) {
  await processItem(i);
  await subagent_progress({ 
    status: `Item ${i}`, 
    percentage: (i / 10000) * 100 
  });
}

// ✅ Good: Report every 100 items or 5% progress
for (let i = 0; i < 10000; i++) {
  await processItem(i);
  
  if (i % 100 === 0) {
    await subagent_progress({ 
      status: `Processed ${i} of 10000 items`, 
      percentage: Math.floor((i / 10000) * 100) 
    });
  }
}
```

## Best practices

### ✅ Do

- Report progress every **5-10% completion**
- Include **concrete numbers** when possible (`23 of 50` not "some files")
- Update when **switching phases** ("Analyzing → Testing → Reporting")
- Add **metadata** for structured tracking
- Report at **meaningful milestones** (not arbitrary intervals)

### ❌ Don't

- Report more than **once per second** (throttled anyway)
- Report only at **0% and 100%** (defeats the purpose)
- Use **vague statuses** like "Working..." (be specific)
- Spam updates on **every loop iteration** (batch your reports)

## Viewing progress

Users see progress updates when checking subagent status:

```typescript
await subagents({ 
  action: 'info', 
  runId: 'run-abc123' 
});
```

**Response includes progress history:**
```json
{
  "runId": "run-abc123",
  "status": "running",
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
    },
    {
      "timestamp": "2026-02-24T14:04:30Z",
      "status": "Analysis complete",
      "percentage": 100,
      "metadata": {
        "filesProcessed": 50,
        "totalFiles": 50,
        "issuesFound": 7
      }
    }
  ]
}
```

## Error handling

### Not a subagent

```json
{
  "success": false,
  "error": {
    "code": "NOT_ALLOWED",
    "message": "Only subagents can report progress"
  }
}
```

### Invalid percentage

```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Percentage must be between 0 and 100"
  }
}
```

### Subagent not running

```json
{
  "success": false,
  "error": {
    "code": "INVALID_STATE",
    "message": "Can only report progress for running subagents"
  }
}
```

## Next steps

<CardGroup cols={2}>
  <Card title="Spawn subagents" icon="rocket" href="/tools/sessions-spawn">
    Create background tasks that can report progress
  </Card>
  <Card title="Monitor subagents" icon="chart-line" href="/tools/subagents">
    View progress updates and subagent status
  </Card>
</CardGroup>
