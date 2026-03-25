# Claude Model Optimization Guide

> Anthropic Claude API optimization settings for OpenClaw

[English](claude-optimization.md) | [한국어](claude-optimization.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config Location

```
~/.openclaw/openclaw.json
```

## 🎯 Key Optimization Settings

### 1. Prompt Caching (`cacheRetention`)

Cache repeated prompts to reduce costs and latency.

| Value | TTL | Description |
|-------|-----|-------------|
| `"none"` | - | Disable caching |
| `"short"` | 5 min | Default for API Key |
| `"long"` | 1 hour | Extended cache (beta) |

⚠️ **API Key only** — Ignored for OAuth/setup-token

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "cacheRetention": "long"
          }
        }
      }
    }
  }
}
```

### 2. Adaptive Thinking (`thinking`)

Let Claude decide when and how much to "think" based on task complexity.

| Value | Description |
|-------|-------------|
| `"off"` | No thinking |
| `"low"` | Minimal thinking |
| `"medium"` | Moderate thinking |
| `"high"` | Always think deeply |
| `"adaptive"` | Auto-decide (default for 4.6) |

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "thinking": "adaptive"
          }
        }
      }
    }
  }
}
```

### 3. Priority Tier (`fastMode`)

Enable Anthropic's priority processing tier.

⚠️ **API Key only**

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "params": {
            "fastMode": true
          }
        }
      }
    }
  }
}
```

### 4. 1M Context Window (`context1m`)

Enable 1 million token context window (beta).

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "context1m": true
          }
        }
      }
    }
  }
}
```

### 5. Model Fallbacks

Automatic failover when primary model fails.

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": [
          "anthropic/claude-sonnet-4-6",
          "google/gemini-2.5-flash"
        ]
      }
    }
  }
}
```

### 6. Model Aliases

Short names for quick model switching.

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "alias": "opus"
        },
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet"
        }
      }
    }
  }
}
```

## 🎯 Recommended Configurations

### High Performance (Max Plan)

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-6"]
      },
      "models": {
        "anthropic/claude-opus-4-6": {
          "alias": "opus",
          "params": {
            "thinking": "adaptive"
          }
        }
      }
    }
  }
}
```

### Cost Optimized (API Key)

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-6",
        "fallbacks": ["anthropic/claude-haiku-4-5"]
      },
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet",
          "params": {
            "cacheRetention": "long",
            "thinking": "low"
          }
        }
      }
    }
  }
}
```

## 📊 Model Comparison

| Model | Price (in/out) | Context | Best For |
|-------|----------------|---------|----------|
| **Opus 4.6** | $5/$25 per MTok | 1M | Complex reasoning, agents |
| **Sonnet 4.6** | $3/$15 per MTok | 1M | Balanced speed + quality |
| **Haiku 4.5** | $1/$5 per MTok | 200k | Fast, simple tasks |

## 📚 References

- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [Anthropic Adaptive Thinking](https://docs.anthropic.com/en/docs/build-with-claude/adaptive-thinking)
- [OpenClaw Anthropic Provider](https://docs.openclaw.ai/providers/anthropic)
