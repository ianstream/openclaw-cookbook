# OpenClaw Token Optimization Guide

> Context management and cost optimization for long-running sessions

[English](token-optimization.md) | [한국어](token-optimization.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config File Location

```
~/.openclaw/openclaw.json
```

## 🔧 Two Optimization Mechanisms

| Feature | Context Pruning | Compaction |
|---------|-----------------|------------|
| **Target** | Old tool results | Entire conversation |
| **Method** | In-memory removal (per request) | Summarize & compress |
| **Persistence** | No session file changes | Saved to JSONL |
| **Purpose** | Cache cost reduction | Context window management |

## 1️⃣ Context Pruning

### Concept
- Removes **old tool results** from memory right before each LLM call
- Does NOT rewrite session files (*.jsonl)
- Works with Anthropic cache TTL to reduce costs

### Why?
- Anthropic prompt caching only applies within TTL
- After TTL expires, next request re-caches full prompt
- **Pruning = Reduced cache-write costs**

### Basic Config

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "5m"
      }
    }
  }
}
```

### Advanced Config

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "45m",
        "keepLastAssistants": 3,
        "softTrimRatio": 0.3,
        "hardClearRatio": 0.5,
        "minPrunableToolChars": 50000,
        "softTrim": {
          "maxChars": 4000,
          "headChars": 1500,
          "tailChars": 1500
        },
        "hardClear": {
          "enabled": true,
          "placeholder": "[Old tool result content cleared]"
        }
      }
    }
  }
}
```

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mode` | `"off"` | `"cache-ttl"` = Enable TTL-based pruning |
| `ttl` | `"5m"` | Prune after this duration since last API call |
| `keepLastAssistants` | `3` | Protect last N assistant responses |
| `softTrimRatio` | `0.3` | Threshold for soft-trim |
| `hardClearRatio` | `0.5` | Threshold for hard-clear |
| `minPrunableToolChars` | `50000` | Only prune tool results larger than this |

### Soft vs Hard Pruning

| Type | Behavior |
|------|----------|
| **Soft-trim** | Keep head + tail, insert `...` in middle |
| **Hard-clear** | Replace entire result with placeholder |

### What's NOT pruned
- ❌ User messages
- ❌ Assistant messages  
- ❌ Tool results with images
- ✅ Only toolResult messages

### Prune Specific Tools Only

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "tools": {
          "allow": ["exec", "read"],
          "deny": ["*image*"]
        }
      }
    }
  }
}
```

## 2️⃣ Compaction

### Concept
- When context window fills up, **summarizes old conversation**
- Summary is saved to session JSONL
- Future requests use: summary + recent messages

### Basic Config

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "default"
      }
    }
  }
}
```

### Advanced Config

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "model": "openrouter/anthropic/claude-sonnet-4-6",
        "identifierPolicy": "strict",
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000
        }
      }
    }
  }
}
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `mode` | `"default"` = Auto compaction |
| `reserveTokensFloor` | Minimum reserved tokens (kept after compaction) |
| `model` | Separate model for compaction (optional) |
| `identifierPolicy` | `"strict"` = preserve identifiers, `"off"` = ignore |
| `memoryFlush.enabled` | Run memory save turn before compaction |
| `memoryFlush.softThresholdTokens` | Trigger flush when this many tokens remain |

### Manual Compaction

```
/compact Focus on decisions and open questions
```

## 🎯 Recommended Settings

### Max Plan Users (Quality over cost)

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "6h",
        "keepLastAssistants": 3
      },
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000
        }
      }
    }
  }
}
```

### API Key Users (Cost optimization)

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "30m",
        "keepLastAssistants": 2,
        "minPrunableToolChars": 30000,
        "hardClear": {
          "enabled": true
        }
      },
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 30000,
        "model": "anthropic/claude-sonnet-4-6"
      }
    }
  }
}
```

## 🔍 Check Status

```bash
# Current session status
/status

# Check compaction count (🧹 Compactions: N)
```

## 📚 References

- [OpenClaw Compaction Docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/compaction.md)
- [Session Pruning Docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/session-pruning.md)
- [Gateway Configuration](https://docs.openclaw.ai/gateway/configuration)

## License

MIT
