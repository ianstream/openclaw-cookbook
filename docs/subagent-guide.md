# Subagent Guide

> Run parallel background tasks with child agents

[English](subagent-guide.md) | [한국어](subagent-guide.ko.md) | [← Back to Cookbook](../README.md)

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
      "subagents": {
        "maxConcurrent": 4,
        "runTimeoutSeconds": 900
      }
    }
  }
}
```

## ⚙️ Options

| Option | Description |
|--------|-------------|
| `maxConcurrent` | Max parallel subagents |
| `runTimeoutSeconds` | Timeout per run (default: 900) |
| `model` | Use cheaper model for subagents |
| `maxSpawnDepth` | Max nesting depth (default: 2) |
| `maxChildrenPerAgent` | Max children per parent |
| `archiveAfterMinutes` | Archive completed sessions |

## 📝 Example Configs

### Cost Optimized

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "model": "openai/gpt-4.1-mini",
        "maxConcurrent": 4,
        "runTimeoutSeconds": 600
      }
    }
  }
}
```

### Orchestrator Pattern

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxSpawnDepth": 2,
        "maxChildrenPerAgent": 5,
        "maxConcurrent": 8
      }
    }
  }
}
```

### Restrict Subagent Tools

```json
{
  "tools": {
    "subagents": {
      "tools": {
        "deny": ["gateway", "cron", "message"]
      }
    }
  }
}
```

## 🏗️ Architecture

```
Main Agent (Opus)
    ├── Subagent 1 (Sonnet) - Research task
    ├── Subagent 2 (Sonnet) - Code review
    └── Subagent 3 (Haiku) - Simple lookup
```

## 💡 Tips

1. **Cost Savings**: Use cheaper model for routine subtasks
2. **Parallel Work**: Set `maxConcurrent: 8` for heavy workloads
3. **Security**: Deny sensitive tools (`gateway`, `cron`) for subagents
4. **Cleanup**: Use `cleanup: "delete"` to auto-remove completed sessions
5. **Depth Limit**: Keep `maxSpawnDepth: 2` to prevent runaway spawning

## 📚 References

- [OpenClaw Subagent Docs](https://docs.openclaw.ai/concepts/subagents)
