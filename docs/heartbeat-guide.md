# Heartbeat Guide

> Periodic agent check-ins and background tasks

[English](heartbeat-guide.md) | [한국어](heartbeat-guide.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config Location

```
~/.openclaw/openclaw.json
```

## 🎯 Basic Setup

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "1h"
      }
    }
  }
}
```

## ⚙️ Options

| Option | Description |
|--------|-------------|
| `every` | Interval (`"30m"`, `"1h"`, `"2h"`) |
| `target` | Where to send (`"last"`, `"none"`, `"discord"`) |
| `lightContext` | Only load HEARTBEAT.md (saves tokens) |
| `activeHours` | Only run during specific hours |
| `model` | Use cheaper model for heartbeats |

## 📝 Example Configs

### With Active Hours

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",
        "target": "last",
        "lightContext": true,
        "activeHours": {
          "start": "09:00",
          "end": "23:00",
          "timezone": "Asia/Seoul"
        }
      }
    }
  }
}
```

### Cost Optimized (Cheaper Model)

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "1h",
        "model": "openai/gpt-4.1-mini",
        "lightContext": true
      }
    }
  }
}
```

### Send to Specific Channel

```json
{
  "agents": {
    "list": [
      {
        "id": "ops",
        "heartbeat": {
          "every": "1h",
          "target": "discord",
          "to": "channel:123456789012345678",
          "prompt": "Check HEARTBEAT.md. Report only urgent items. Otherwise HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

## 📄 HEARTBEAT.md Example

Create `~/.openclaw/workspace/HEARTBEAT.md`:

```markdown
# Heartbeat Checklist

## Every Check
- [ ] Check inbox for urgent emails
- [ ] Review calendar for upcoming meetings

## Daily (Once)
- [ ] Summarize yesterday's activity
- [ ] Update memory files
```

## 💡 Tips

1. **Save Tokens**: Use `lightContext: true` to only load HEARTBEAT.md
2. **Save Cost**: Use cheaper model for heartbeats
3. **Quiet Hours**: Set `activeHours` to avoid night notifications
4. **Response**: Reply `HEARTBEAT_OK` when nothing needs attention

## 📚 References

- [OpenClaw Heartbeat Docs](https://docs.openclaw.ai/concepts/heartbeat)
