# Cron Job Guide

> Schedule tasks at specific times

[English](cron-guide.md) | [한국어](cron-guide.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config Location

```
~/.openclaw/openclaw.json
```

## 🎯 Basic Setup

```json
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 2
  }
}
```

## 📝 CLI Examples

### Daily Morning Brief

```bash
openclaw cron add \
  --name "Morning Brief" \
  --cron "0 7 * * *" \
  --tz "Asia/Seoul" \
  --session isolated \
  --message "Summarize today's schedule and inbox." \
  --announce \
  --channel discord \
  --to "channel:YOUR_CHANNEL_ID"
```

### One-Time Reminder

```bash
openclaw cron add \
  --name "Reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: Meeting in 10 minutes" \
  --wake now \
  --delete-after-run
```

### Weekly Report

```bash
openclaw cron add \
  --name "Weekly Report" \
  --cron "0 9 * * 1" \
  --tz "Asia/Seoul" \
  --session isolated \
  --message "Generate weekly activity report." \
  --announce
```

## ⚙️ Schedule Types

| Type | Example |
|------|---------|
| `--cron` | `"0 7 * * *"` (Every day 7 AM) |
| `--at` | `"20m"` (In 20 minutes) |
| `--every` | `"2h"` (Every 2 hours) |

## 🔧 Options

| Option | Description |
|--------|-------------|
| `--session isolated` | Run in separate session (recommended) |
| `--session main` | Run in main session |
| `--announce` | Send result to channel |
| `--delete-after-run` | One-time job |
| `--model` | Use specific model |
| `--lightContext` | Skip workspace files |

## 📝 JSON Format (Tool Call)

```json
{
  "name": "Morning Brief",
  "schedule": {
    "kind": "cron",
    "expr": "0 7 * * *",
    "tz": "Asia/Seoul"
  },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize today's schedule.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:123456789012345678"
  }
}
```

## 💡 Tips

1. **Use Isolated Sessions**: Prevents polluting main session context
2. **Cost Savings**: Use `--model openai/gpt-4.1-mini` for routine tasks
3. **Light Context**: Use `--lightContext` to skip workspace loading
4. **Exact Time**: Use `--exact` to disable top-of-hour stagger

## 🛠️ Management Commands

```bash
openclaw cron list              # List all jobs
openclaw cron runs <jobId>      # View run history
openclaw cron remove <jobId>    # Delete job
openclaw cron run <jobId>       # Run immediately
```

## 📚 References

- [OpenClaw Cron Docs](https://docs.openclaw.ai/concepts/cron)
