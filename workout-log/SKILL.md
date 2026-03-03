---
name: workout-log
description: Log a workout session (strength, cardio, mobility, or football) and insert it into the life_db database. Use when the user shares their training session, exercises, sets, reps, weights, session duration, feelings, or asks to record a workout.
---

# Workout Log

Parses workout sessions from natural or structured input and inserts them into the `sport` schema of `life_db`.

## Connection

Credentials are in `~/.openclaw/services/life-db/.env`.

## Workflow

### 1. Parse the user's message

Extract from free text or structured input:

| Field | Type | Notes |
|-------|------|-------|
| `session_date` | YYYY-MM-DD | Default: today |
| `session_type` | strength / cardio / mobility / football | Infer from context |
| `duration_min` | float | Total session duration |
| `feeling` | 1–10 integer | Overall session feeling |
| `notes` | string | Optional |
| `exercises[]` | array | One object per exercise (omit for football if no drill detail) |

Each exercise:

| Field | Type | Notes |
|-------|------|-------|
| `exercise_name` | string | Normalize consistently |
| `sets` | int | Strength only |
| `reps` | int | Strength only |
| `weight_kg` | float | null for bodyweight |
| `duration_min` | float | Cardio / mobility / football drills |
| `distance_km` | float | Cardio only |
| `order_in_session` | int | Order as mentioned |
| `notes` | string | Optional |

### 2. Confirm with the user

Show a summary before inserting:

```
📅 {DD/MM/YYYY} · {Type} · {duration} min · Feeling {feeling}/10
• {Exercise 1} — {sets}×{reps} @ {weight} kg
• {Exercise 2} — …

On save ? (oui / non)
```

### 3. Insert

Only after confirmation, use the `exec` tool:

```json
{
  "tool": "exec",
  "command": "bash -c 'set -a; source ~/.openclaw/services/life-db/.env; uv run --project ~/.openclaw ~/.openclaw/workspace/skills/workout-log/scripts/insert_workout.py <json>'"
}
```

Replace `<json>` with the minified JSON payload wrapped in single quotes.

Replace `<json>` with the minified JSON payload (no newlines, properly shell-escaped).

### 4. Confirm result

**On `OK`** — respond with exactly this format, nothing more:

```
✅ {Type} · {duration} min · Feeling {feeling}/10 · #{session_id}
{comment}
```

`{comment}` rules — **1 line, max 60 chars, no emoji**:
- feeling ≥ 8 → short hype line
- feeling 5–7 → encouragement
- feeling ≤ 4 → grind acknowledgement

**On `ERROR`** — report the error as-is, do not retry without user input.
