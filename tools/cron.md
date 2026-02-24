---
title: "Cron Scheduling"
description: "Schedule recurring tasks, reminders, and automated agent runs with flexible time-based triggers."
---

# Cron Scheduling

The cron system lets your assistant schedule future actions — from one-time reminders to daily automation tasks. It supports three schedule types: one-shot timers (`at`), interval-based (`every`), and cron expressions for complex schedules.

## Configuration

Add to `nachos.toml`:

```toml
[scheduler]
enabled = true
timezone = "America/New_York"  # Optional: Default timezone for cron expressions
check_interval_ms = 60000      # Optional: How often to check for due jobs (default: 1 minute)
```

The scheduler runs in the gateway process and persists jobs to the state database.

## Schedule Types

### 1. `at` — One-Time Execution

Execute once at a specific time.

```json
{
  "scheduleType": "at",
  "scheduleValue": "2024-02-24T15:30:00-05:00"
}
```

**Format:** ISO 8601 timestamp with timezone

**Use cases:**
- "Remind me in 30 minutes"
- "Follow up on this email tomorrow at 9am"
- "Check deployment status at 5pm"

### 2. `every` — Fixed Interval

Execute repeatedly at fixed intervals.

```json
{
  "scheduleType": "every",
  "scheduleValue": "3600000"
}
```

**Format:** Milliseconds between executions

**Common intervals:**
- `60000` — Every minute
- `300000` — Every 5 minutes
- `3600000` — Every hour
- `86400000` — Every 24 hours

**Use cases:**
- "Check for new emails every 30 minutes"
- "Sync data hourly"
- "Run health check every 5 minutes"

### 3. `cron` — Cron Expression

Execute based on cron schedule (supports standard 5-field cron syntax).

```json
{
  "scheduleType": "cron",
  "scheduleValue": "0 9 * * 1-5",
  "timezone": "America/New_York"
}
```

**Format:** `minute hour day-of-month month day-of-week`

**Examples:**

| Schedule           | Cron Expression   | Description                    |
|--------------------|-------------------|--------------------------------|
| Every day at 9am   | `0 9 * * *`       | Daily morning check            |
| Weekdays at 9am    | `0 9 * * 1-5`     | Monday-Friday                  |
| Every hour         | `0 * * * *`       | Top of every hour              |
| Every 15 min       | `*/15 * * * *`    | Every quarter hour             |
| 1st of month       | `0 9 1 * *`       | Monthly report                 |
| Mon/Wed/Fri 2pm    | `0 14 * * 1,3,5`  | Three times per week           |

**Use cases:**
- "Check email every morning at 9am"
- "Run report first day of month"
- "Backup data every night at midnight"

## Action Types

### `systemEvent` — Inject Text into Session

Injects text as if the user sent it. The assistant processes it in the current session.

```json
{
  "actionType": "systemEvent",
  "actionPrompt": "Check my email for anything urgent"
}
```

**Use case:** Periodic checks that continue in the main conversation thread.

**Behavior:**
- Text appears in chat history
- Assistant responds in the same channel
- User sees the assistant's response

### `agentTurn` — Isolated Agent Run

Spawns an isolated agent run with the given prompt. Runs independently from the main session.

```json
{
  "actionType": "agentTurn",
  "actionPrompt": "Run health checks and report any failures"
}
```

**Use case:** Background tasks that shouldn't clutter the main conversation.

**Behavior:**
- Runs in separate session
- No chat history from main session
- Can deliver results to specific channel
- More isolated and clean for automation

## Tools

### `nachos_cron_add` — Create a cron job

```json
{
  "name": "Morning Email Check",
  "description": "Check for urgent emails every morning",
  "scheduleType": "cron",
  "scheduleValue": "0 9 * * 1-5",
  "timezone": "America/New_York",
  "actionType": "systemEvent",
  "actionPrompt": "Check my email for anything urgent and summarize",
  "deliveryChannel": "discord",
  "deliveryAnnounce": false
}
```

**Parameters:**

- `name` (string, required) — Job name for identification
- `description` (string, optional) — What this job does
- `scheduleType` (string, required) — `"at"`, `"every"`, or `"cron"`
- `scheduleValue` (string, required) — Schedule value (format depends on type)
- `timezone` (string, optional) — Timezone for cron schedules. Default: `"UTC"`
- `actionType` (string, required) — `"systemEvent"` or `"agentTurn"`
- `actionPrompt` (string, required) — What the assistant should do
- `deliveryChannel` (string, optional) — Channel to deliver results (e.g., `"discord"`)
- `deliveryAnnounce` (boolean, optional) — Whether to announce execution. Default: `false`

**Returns:**

```
Created cron job "Morning Email Check" (ID: cron_abc123)
Schedule: cron 0 9 * * 1-5
Next run: 2024-02-26 09:00:00
```

### `nachos_cron_list` — List cron jobs

```json
{
  "enabled": true,
  "limit": 50
}
```

**Parameters:**

- `enabled` (boolean, optional) — Filter by enabled status
- `limit` (number, optional) — Max jobs to return. Default: 50

**Returns:**

```
Found 3 cron job(s):

1. [✓] Morning Email Check (cron_abc123)
   Schedule: cron:0 9 * * 1-5
   Next run: Feb 26, 2024 9:00 AM
   Action: systemEvent

2. [✓] Hourly Data Sync (cron_def456)
   Schedule: every:3600000
   Next run: Feb 24, 2024 3:00 PM
   Action: agentTurn

3. [✗] Deployment Reminder (cron_ghi789)
   Schedule: at:2024-02-24T17:00:00Z
   Next run: not scheduled
   Action: systemEvent
```

### `nachos_cron_remove` — Delete a cron job

```json
{
  "jobId": "cron_abc123"
}
```

**Returns:**

```
Deleted cron job "Morning Email Check" (cron_abc123)
```

### `nachos_cron_update` — Update a cron job

```json
{
  "jobId": "cron_abc123",
  "scheduleValue": "0 8 * * 1-5",
  "enabled": true
}
```

**Parameters:**

- `jobId` (string, required) — Job ID to update
- `name` (string, optional) — New name
- `description` (string, optional) — New description
- `scheduleValue` (string, optional) — New schedule
- `timezone` (string, optional) — New timezone
- `actionPrompt` (string, optional) — New prompt
- `enabled` (boolean, optional) — Enable or disable job
- `deliveryChannel` (string, optional) — New delivery channel
- `deliveryAnnounce` (boolean, optional) — New announce setting

**Returns:**

```
Updated cron job "Morning Email Check" (cron_abc123)
Next run: 2024-02-26 08:00:00
```

### `nachos_cron_run` — Trigger job manually

```json
{
  "jobId": "cron_abc123"
}
```

Runs the job immediately without waiting for the next scheduled time.

**Returns:**

```
Manually triggered cron job "Morning Email Check" (run ID: run_xyz789)
```

## Common Workflows

### 1. Simple Reminder

> "Remind me to check the deployment in 30 minutes"

The assistant creates:

```json
{
  "name": "Deployment Check Reminder",
  "scheduleType": "at",
  "scheduleValue": "2024-02-24T15:00:00-05:00",
  "actionType": "systemEvent",
  "actionPrompt": "Reminder: Check the deployment status"
}
```

In 30 minutes, the assistant will send the reminder in the current channel.

### 2. Daily Email Digest

> "Check my email every morning at 9am on weekdays and summarize anything urgent"

```json
{
  "name": "Morning Email Digest",
  "scheduleType": "cron",
  "scheduleValue": "0 9 * * 1-5",
  "timezone": "America/New_York",
  "actionType": "systemEvent",
  "actionPrompt": "Check email for urgent messages and provide a summary",
  "deliveryChannel": "discord"
}
```

### 3. Periodic Health Check

> "Run health checks every 15 minutes and alert me if anything fails"

```json
{
  "name": "Health Check",
  "scheduleType": "cron",
  "scheduleValue": "*/15 * * * *",
  "actionType": "agentTurn",
  "actionPrompt": "Run all health checks. If any fail, send alert with details.",
  "deliveryChannel": "slack",
  "deliveryAnnounce": true
}
```

### 4. Weekly Report

> "Send me a weekly summary every Friday at 5pm"

```json
{
  "name": "Weekly Summary",
  "scheduleType": "cron",
  "scheduleValue": "0 17 * * 5",
  "timezone": "America/New_York",
  "actionType": "agentTurn",
  "actionPrompt": "Generate weekly activity summary and send via email",
  "deliveryChannel": "email"
}
```

### 5. Data Sync

> "Sync data from the API every hour"

```json
{
  "name": "Hourly Data Sync",
  "scheduleType": "every",
  "scheduleValue": "3600000",
  "actionType": "agentTurn",
  "actionPrompt": "Fetch latest data from API and update local cache"
}
```

## Natural Language Examples

Users can create jobs conversationally:

| User Request                                    | Creates Job                     |
|-------------------------------------------------|---------------------------------|
| "Remind me in 20 minutes"                       | `at` job 20 min from now        |
| "Check email every morning at 9am"              | `cron` job: `0 9 * * *`         |
| "Run health check every 5 minutes"              | `every` job: 300000 ms          |
| "Send me a summary every Friday at 5pm"         | `cron` job: `0 17 * * 5`        |
| "Remind me tomorrow at 2pm"                     | `at` job: tomorrow 14:00        |
| "Check for updates every hour"                  | `every` job: 3600000 ms         |

The assistant parses intent and calculates appropriate scheduleType and scheduleValue.

## Schedule Validation

The scheduler validates schedule values:

**Valid `at` values:**
- ISO 8601 timestamps: `"2024-02-24T15:30:00-05:00"`
- Must be in the future

**Valid `every` values:**
- Positive integers (milliseconds)
- Minimum: 60000 (1 minute)

**Valid `cron` values:**
- Standard 5-field cron expressions
- Minute (0-59), Hour (0-23), Day (1-31), Month (1-12), Weekday (0-7)
- Supports ranges (`1-5`), lists (`1,3,5`), steps (`*/15`)

**Invalid schedule returns error:**

```
Invalid schedule value for type "cron": "invalid syntax"
```

## Permissions & Security

### User Isolation

Users can only manage their own jobs. Trying to modify another user's job:

```
You do not have permission to update this job
```

### System Jobs

Jobs created by the system have `userId: __system__` and can only be managed by admins or the heartbeat system.

## Delivery Channels

Jobs can deliver results to specific channels:

```json
{
  "deliveryChannel": "discord",
  "deliveryAnnounce": false
}
```

**Channels:**
- `"discord"` — Send to Discord channel
- `"slack"` — Send to Slack channel
- `"telegram"` — Send to Telegram chat
- `"email"` — Send via email (requires email integration)

**`deliveryAnnounce` behavior:**
- `false` (default) — Silent execution, only send result if non-empty
- `true` — Announce job execution even if no output

## Persistence & Reliability

- Jobs are persisted to the state database
- Survive gateway restarts
- Next run time calculated at creation and after each execution
- If gateway is down during scheduled time, job runs when gateway starts (if configured)

## Performance Considerations

**Check interval:**

```toml
[scheduler]
check_interval_ms = 60000  # Check every minute
```

- Lower values = more precise timing but higher CPU usage
- Higher values = less precise but more efficient
- Default 60s is good for most use cases

**Job limits:**

- No hard limit on number of jobs
- Consider resource usage for high-frequency jobs
- Use `agentTurn` for heavy background tasks

## Error Handling

Common errors:

| Error                     | Cause                           | Solution                          |
|---------------------------|---------------------------------|-----------------------------------|
| `Invalid schedule`        | Malformed schedule value        | Check format for schedule type    |
| `Job not found`           | Invalid job ID                  | List jobs to get valid IDs        |
| `Permission denied`       | Trying to modify other's job    | Can only manage your own jobs     |
| `Schedule in the past`    | `at` time is not in future      | Use future timestamp              |
| `Invalid cron expression` | Malformed cron syntax           | Check cron expression syntax      |

## Cron Expression Reference

### Fields

```
┌───────── minute (0-59)
│ ┌───────── hour (0-23)
│ │ ┌───────── day of month (1-31)
│ │ │ ┌───────── month (1-12)
│ │ │ │ ┌───────── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * *
```

### Special Characters

- `*` — Any value (e.g., `* * * * *` = every minute)
- `*/n` — Every n units (e.g., `*/15 * * * *` = every 15 minutes)
- `n-m` — Range (e.g., `0 9-17 * * *` = 9am-5pm)
- `n,m,o` — List (e.g., `0 9 * * 1,3,5` = Mon/Wed/Fri at 9am)

### Common Patterns

```bash
0 9 * * *          # Daily at 9am
0 */6 * * *        # Every 6 hours
0 9 * * 1-5        # Weekdays at 9am
0 0 1 * *          # First day of month at midnight
*/30 * * * *       # Every 30 minutes
0 9,17 * * 1-5     # Weekdays at 9am and 5pm
0 0 * * 0          # Every Sunday at midnight
```

## Example nachos.toml

```toml
[scheduler]
enabled = true
timezone = "America/New_York"
check_interval_ms = 60000
```

## Tips

- Use `cron` for human-friendly schedules (e.g., "every morning")
- Use `every` for fixed intervals (e.g., "every 5 minutes")
- Use `at` for one-time reminders
- Use `agentTurn` for background automation to keep chat clean
- Use `systemEvent` for interactive reminders
- Set `deliveryAnnounce: false` for quiet background tasks
- List jobs regularly to review active automation
- Disable jobs instead of deleting them if you might need them again

## Related

- [Heartbeat System](/tools/heartbeat) — Proactive assistant checks (built on cron)
- [Composio](/tools/composio) — Automate email/calendar actions with cron
- [GitHub Tool](/tools/github) — Automate PR checks and issue triage
- [Security Policies](/security/policies) — Control scheduler access
