---
title: "Heartbeat System"
description: "Enable proactive assistant behavior with periodic automated checks."
---

# Heartbeat System

The heartbeat system makes your assistant proactive. Instead of waiting for you to ask, it periodically checks email, calendar, notifications, or any other task you configure. It's built on top of the cron scheduler but designed specifically for assistant "self-initiated" behavior.

## What is a Heartbeat?

A heartbeat is a recurring prompt sent to the assistant at regular intervals. The assistant processes it like any other message and can:

- Check for important information (emails, calendar events)
- Monitor systems (health checks, deployments)
- Provide updates (news, weather, reminders)
- Perform background tasks (cleanup, syncs)
- **Stay silent** if nothing needs attention

The key pattern: **HEARTBEAT_OK**

When the assistant determines nothing needs action, it replies with `HEARTBEAT_OK` instead of a full message. This prevents notification spam while allowing genuine updates to come through.

## Configuration

Add to `nachos.toml`:

```toml
[heartbeat]
enabled = true
intervalMinutes = 30
channel = "discord"
prompt = "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."
```

**Parameters:**

- `enabled` (boolean) — Enable/disable heartbeat system. Default: `false`
- `intervalMinutes` (number) — How often to run (in minutes). Default: 30
- `channel` (string, optional) — Channel to send heartbeat prompts to
- `prompt` (string) — The prompt sent each heartbeat. See [Customizing the Prompt](#customizing-the-prompt)

**How it works:**

1. Heartbeat creates a system cron job that runs every `intervalMinutes`
2. The job injects `prompt` as a system event in the specified `channel`
3. The assistant processes it like a user message
4. If nothing needs attention, assistant replies `HEARTBEAT_OK` (silent, no notification)
5. If something needs attention, assistant sends a real message

## Default Heartbeat Prompt

The recommended default prompt:

```
Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. 
Do not infer or repeat old tasks from prior chats. 
If nothing needs attention, reply HEARTBEAT_OK.
```

**Why this works:**

- Delegates logic to `HEARTBEAT.md` file (easier to update)
- Prevents the assistant from making up tasks
- Requires explicit `HEARTBEAT_OK` to stay silent

## HEARTBEAT.md File

Create a `HEARTBEAT.md` file in your workspace to define what the assistant should check:

```markdown
# Heartbeat Checklist

Check these in rotation (not all at once):

## Morning (8am-12pm)
- Check email for urgent messages
- Review calendar for today's events
- Check GitHub PRs needing review

## Afternoon (12pm-5pm)
- Monitor deployment status
- Check for failed CI runs

## Evening (5pm-11pm)
- Summarize today's activity
- Prepare tomorrow's agenda

## Anytime
- If mentioned in recent chat, follow up
- If system alerts exist, report them

**Otherwise:** Reply `HEARTBEAT_OK`
```

**Tips for HEARTBEAT.md:**

- Keep it concise (short prompts = cheaper, faster)
- Rotate through checks instead of doing all at once
- Be specific about when to check what
- Always end with "Otherwise: Reply HEARTBEAT_OK"

## The HEARTBEAT_OK Pattern

When the assistant determines nothing needs attention, it responds with exactly:

```
HEARTBEAT_OK
```

This special response:
- Does not trigger notifications in most channels
- Does not clutter chat history (can be filtered)
- Signals "I checked, all good"

**When to use HEARTBEAT_OK:**

- No urgent emails
- No upcoming calendar events in the next few hours
- No system alerts
- No new mentions or tasks
- Nothing changed since last check

**When NOT to use HEARTBEAT_OK:**

- Found urgent email → Summarize it
- Meeting in 30 minutes → Remind user
- CI pipeline failed → Alert with details
- Weather forecast shows rain → Suggest bringing umbrella

## Proactive Behavior Examples

### Email Monitoring

Every 30 minutes, check for urgent emails:

**HEARTBEAT.md:**

```markdown
## Email Check

Search Gmail for:
- `is:unread from:boss@company.com`
- `is:unread label:urgent`

If any found: Summarize senders and subjects.
Otherwise: HEARTBEAT_OK
```

**Outcome:**
- Silent most of the time
- Alerts only when important email arrives

### Calendar Awareness

Warn about upcoming meetings:

**HEARTBEAT.md:**

```markdown
## Calendar Check

List events in next 2 hours.

If any found: Remind user with time and attendees.
Otherwise: HEARTBEAT_OK
```

**Outcome:**
- "Meeting with Alice in 30 minutes" (if event exists)
- Silent otherwise

### GitHub Monitoring

Track PRs and CI:

**HEARTBEAT.md:**

```markdown
## GitHub Check (once every 2 hours)

1. Check for PRs awaiting my review
2. Check for failed CI runs on main branch

If any found: Report status.
Otherwise: HEARTBEAT_OK
```

**Outcome:**
- Alert when PR needs review
- Alert when CI fails
- Silent otherwise

### News/Weather

Contextual updates:

**HEARTBEAT.md:**

```markdown
## Morning Routine (8-9am only)

1. Check weather for today
2. If rain/snow expected: Suggest bringing umbrella/coat
3. Otherwise: HEARTBEAT_OK

## Other times

HEARTBEAT_OK
```

**Outcome:**
- Weather suggestion in morning if needed
- Silent rest of day

## Frequency Considerations

| Interval      | Use Case                          | Pros                       | Cons                      |
|---------------|-----------------------------------|----------------------------|---------------------------|
| 5-10 minutes  | Critical system monitoring        | Near real-time alerts      | Higher costs, more spam   |
| 15-30 minutes | Email/calendar monitoring         | Balanced responsiveness    | Recommended for most      |
| 1-2 hours     | News monitoring, low-priority     | Cost-effective             | May miss time-sensitive   |
| Daily         | Summaries, reports                | Minimal overhead           | Not responsive            |

**Recommendation:** Start with 30 minutes, adjust based on needs.

## State Tracking

To avoid repeating checks, track state in a JSON file:

**HEARTBEAT.md:**

```markdown
## State Tracking

Read `memory/heartbeat-state.json` for last check times.

```json
{
  "lastEmailCheck": 1708801234,
  "lastCalendarCheck": 1708798765,
  "lastGitHubCheck": 1708795123
}
```

- Check email if >30min since last
- Check calendar if >15min since last
- Check GitHub if >2hr since last

Update state file after each check.

Otherwise: HEARTBEAT_OK
```

## Quiet Hours

Respect user's quiet time:

**HEARTBEAT.md:**

```markdown
## Quiet Hours (11pm-7am)

Always reply: HEARTBEAT_OK

## Active Hours (7am-11pm)

[... perform checks ...]
```

Or disable heartbeat entirely at night via cron job updates.

## Combining with Cron

Heartbeat is just a cron job. You can:

**List the heartbeat job:**

```json
{
  "action": "nachos_cron_list"
}
```

Look for job named `__system_heartbeat__`.

**Pause heartbeat temporarily:**

```json
{
  "action": "nachos_cron_update",
  "jobId": "cron_heartbeat_id",
  "enabled": false
}
```

**Resume:**

```json
{
  "action": "nachos_cron_update",
  "jobId": "cron_heartbeat_id",
  "enabled": true
}
```

**Change frequency:**

```toml
[heartbeat]
enabled = true
intervalMinutes = 15  # Now every 15 minutes
```

Restart gateway to apply config changes.

## Advanced Patterns

### Rotating Checks

Don't do everything every heartbeat. Rotate:

**HEARTBEAT.md:**

```markdown
## Rotation

Read `memory/heartbeat-rotation.txt` for current rotation.

Rotations:
1. Email + Calendar
2. GitHub + CI
3. Weather + News
4. Health Checks

Run current rotation, then advance to next.

If nothing to report: HEARTBEAT_OK
```

### Context-Aware Checks

Check based on recent conversation:

**HEARTBEAT.md:**

```markdown
## Context Check

Read last 10 messages in this channel.

If someone mentioned:
- "deployment" → Check deployment status
- "email" → Check email
- "meeting" → Check calendar

Otherwise: Standard email/calendar check

If nothing to report: HEARTBEAT_OK
```

### Adaptive Frequency

Increase frequency when important:

```markdown
## Adaptive Frequency

If deployment in progress:
  - Check every 5 minutes
  
If waiting for important email:
  - Check every 10 minutes

Otherwise:
  - Normal 30-minute interval
  - Reply: HEARTBEAT_OK
```

*Note: Requires programmatic cron job updates.*

## Monitoring Heartbeat Health

Check heartbeat status:

```json
{
  "action": "nachos_cron_list",
  "limit": 10
}
```

Look for:
- Job name: `__system_heartbeat__`
- Enabled: `✓`
- Next run: Should be within `intervalMinutes`

**Manually trigger heartbeat:**

```json
{
  "action": "nachos_cron_run",
  "jobId": "cron_heartbeat_id"
}
```

## Common Pitfalls

❌ **Doing too much per heartbeat**
- Checking email, calendar, GitHub, weather, news all at once
- Solution: Rotate checks, prioritize based on time/context

❌ **Forgetting HEARTBEAT_OK**
- Every heartbeat sends a message
- Solution: Always end HEARTBEAT.md with "Otherwise: HEARTBEAT_OK"

❌ **Repeating old tasks**
- "Check deployment" from yesterday's chat keeps running
- Solution: Use state files, check timestamps, be explicit

❌ **Too frequent checks**
- Every 5 minutes = expensive, noisy
- Solution: 30 minutes for most cases, 15 for time-sensitive

❌ **Not respecting quiet hours**
- Notifications at 3am
- Solution: Add time-of-day checks to HEARTBEAT.md

## Disabling Heartbeat

**Via config:**

```toml
[heartbeat]
enabled = false
```

Restart gateway.

**Via cron tool:**

```json
{
  "action": "nachos_cron_update",
  "jobId": "cron_heartbeat_id",
  "enabled": false
}
```

## Example Configuration

**nachos.toml:**

```toml
[heartbeat]
enabled = true
intervalMinutes = 30
channel = "discord"
prompt = "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."
```

**HEARTBEAT.md:**

```markdown
# Heartbeat Routine

## Quiet Hours (11pm-7am Eastern)
Reply: HEARTBEAT_OK

## Morning (7am-9am)
1. Check weather → suggest attire if rain/snow
2. Check calendar → warn if meeting &lt;2hr away
3. Check email (is:unread label:urgent)

## Work Hours (9am-6pm)
- Rotate checks every heartbeat:
  1. Email (urgent/from boss)
  2. Calendar (upcoming events)
  3. GitHub (PRs needing review)
  4. CI/CD (failures on main)

## Evening (6pm-11pm)
- Summarize day once at 6pm
- Otherwise: HEARTBEAT_OK

**Default:** HEARTBEAT_OK
```

## Tips

- Start simple: Just email + calendar
- Use HEARTBEAT.md for easy updates (no config reload needed)
- Track state to avoid redundant checks
- Rotate through checks instead of doing all at once
- Always provide a path to HEARTBEAT_OK
- Test by manually triggering: `nachos_cron_run`
- Monitor costs — frequent heartbeats add up
- Disable during vacations or quiet periods

## Related

- [Cron Scheduling](/tools/cron) — Underlying scheduler system
- [Composio](/tools/composio) — Email and calendar integrations for heartbeat
- [GitHub Tool](/tools/github) — Automate PR monitoring via heartbeat
- [Web Search](/tools/web-search) — News monitoring via heartbeat
