---
name: remind-myself
description: Set a one-shot reminder. Use when the user asks to be reminded of something at a specific time or after a duration (e.g., "remind me to call the dentist at 3pm", "remind me in 20 minutes to take out the laundry", "remind me tomorrow at 9am").
metadata: {"openclaw":{"requires":{"bins":["jq"]}}}
---

# Reminder

Creates a one-shot cron job that delivers a reminder back to the user's current channel at the specified time.

## Workflow

### 1. Parse the user's message

Extract:

| Field | Notes |
|-------|-------|
| `text` | What to remember (verbatim or paraphrased, concise) |
| `when` | When to deliver — relative ("in 20 minutes", "in 2h") or absolute ("tomorrow at 9am", "Friday at 14:00") |

If `when` is ambiguous, ask for clarification before proceeding.

### 2. Compute the `--at` value

**Relative durations** → pass directly as `<n><unit>`:
- "in 20 minutes" → `20m`
- "in 2 hours" → `2h`
- "in 1 day" → `1d`
- "in 30 seconds" → `30s`

**Absolute times** → convert to ISO 8601 in **Europe/Paris** timezone:
```bash
TZ=Europe/Paris date -d "tomorrow 09:00" --iso-8601=seconds
# → 2026-03-03T09:00:00+01:00
```

### 3. Create the cron job

```json
{
  "tool": "exec",
  "command": "openclaw cron add --name \"reminder-<slug>\" --at <when> --session isolated --message \"⏰ Reminder: <text>\" --announce --channel telegram --to <TELEGRAM_CHAT_ID> --delete-after-run"
}
```

| Flag | Notes |
|------|-------|
| `--name reminder-<slug>` | Short kebab-case name based on the topic (max 20 chars) |
| `--at` | ISO timestamp or relative duration (`20m`, `2h`, `1d`) |
| `--session isolated` | Runs as a dedicated background agent turn |
| `--message` | Prompt for the isolated agent turn |
| `--announce` | Delivers the output to the specified channel |
| `--channel telegram --to <ID>` | Telegram chat ID, defined in `TOOLS.md` |
| `--delete-after-run` | Auto-cleans the one-shot job after firing |

### 4. Verify the cron job was created

After `cron add`, list active cron jobs to confirm the reminder exists:

```json
{
  "tool": "exec",
  "command": "openclaw cron list"
}
```

Check that the output contains the `reminder-<slug>` job. If it does not appear, report the failure to the user — do not assume success.

### 5. Confirm to the user

Only after verification (step 4), confirm with:

```
⏰ Reminder set!
📝 <text>
🕐 <human-readable time> (Europe/Paris)
```

### 6. Error handling

- **Never assume failure without running the command.** Always execute `cron add` and report the actual output.
- **Never invent a diagnosis.** If something fails, show the raw error — do not guess the cause.
- If `openclaw cron add` fails → report the exact error output and do not retry silently
- If the time is in the past → warn the user and ask for a new time
- If the reminder text is empty → ask what to remind them of

## Examples

| User says | `--at` | `--name` |
|-----------|--------|---------|
| "in 20 minutes, remind me to take out the laundry" | `20m` | `reminder-laundry` |
| "remind me to call Alice tomorrow at 14h" | `2026-03-03T14:00:00+01:00` | `reminder-call-alice` |
| "in 2 hours: check the oven" | `2h` | `reminder-oven` |
| "friday at 9am: team standup" | `2026-03-06T09:00:00+01:00` | `reminder-standup` |
