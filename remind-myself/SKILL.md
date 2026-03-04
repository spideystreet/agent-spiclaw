---
name: remind-myself
description: Set a one-shot reminder. Use when the user asks to be reminded of something at a specific time or after a duration (e.g., "remind me to call the dentist at 3pm", "rappelle-moi dans 20 minutes de sortir le linge", "remind me tomorrow at 9am").
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
| `when` | When to deliver — relative ("in 20 minutes", "dans 2h") or absolute ("demain à 9h", "Friday at 14:00") |

If `when` is ambiguous, ask for clarification before proceeding.

### 2. Compute the `--at` value

**Relative durations** → pass directly as `<n><unit>`:
- "dans 20 minutes" → `20m`
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
  "command": "openclaw cron add --token \"$(jq -r '.gateway.auth.token' ~/.openclaw/openclaw.json)\" --at <when> --system-event \"⏰ Rappel : <text>\" --announce --channel telegram --to <TELEGRAM_CHAT_ID> --delete-after-run --name \"reminder-<slug>\""
}
```

| Flag | Notes |
|------|-------|
| `--token` | Gateway auth token, extracted from `openclaw.json` via `jq` |
| `--at` | ISO timestamp or relative duration (`20m`, `2h`, `1d`) |
| `--system-event` | Message content of the reminder |
| `--announce` | Posts the result back to the user's chat |
| `--channel telegram --to <ID>` | Telegram chat ID, defined in `TOOLS.md` |
| `--delete-after-run` | Auto-cleans the one-shot job after firing |
| `--name reminder-<slug>` | Short kebab-case name based on the topic (max 20 chars) |

### 4. Confirm to the user

After the job is created successfully, confirm with:

```
⏰ Rappel enregistré !
📝 <text>
🕐 <human-readable time> (Europe/Paris)
```

### 5. Error handling

- If `openclaw cron add` fails → report the error and do not retry silently
- If the time is in the past → warn the user and ask for a new time
- If the reminder text is empty → ask what to remind them of

## Examples

| User says | `--at` | `--name` |
|-----------|--------|---------|
| "dans 20 minutes, rappelle-moi de sortir le linge" | `20m` | `reminder-linge` |
| "remind me to call Alice tomorrow at 14h" | `2026-03-03T14:00:00+01:00` | `reminder-call-alice` |
| "in 2 hours: check the oven" | `2h` | `reminder-oven` |
| "friday at 9am: team standup" | `2026-03-06T09:00:00+01:00` | `reminder-standup` |
