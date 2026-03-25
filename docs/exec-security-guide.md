# Exec Security Guide

> Control command execution permissions and sandboxing

[English](exec-security-guide.md) | [한국어](exec-security-guide.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config Location

```
~/.openclaw/openclaw.json
```

## 🎯 Basic Setup

```json
{
  "tools": {
    "exec": {
      "security": "full"
    }
  }
}
```

## ⚙️ Security Modes

| Mode | Description |
|------|-------------|
| `"full"` | All commands allowed |
| `"allowlist"` | Only whitelisted commands |
| `"deny"` | No exec allowed |

## 🔒 Elevated Commands

Allow specific users to run host commands:

```json
{
  "tools": {
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "discord": ["user:YOUR_USER_ID"],
        "telegram": ["+821012345678"]
      }
    }
  }
}
```

## 🐳 Docker Sandboxing

Run commands in isolated containers:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "agent",
        "workspaceAccess": "rw",
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "network": "none",
          "memory": "1g",
          "cpus": 1
        }
      }
    }
  }
}
```

### Sandbox Modes

| Mode | Description |
|------|-------------|
| `"off"` | No sandboxing |
| `"non-main"` | Only subagents sandboxed |
| `"all"` | Everything sandboxed |

## 📝 Example Configs

### Production (Strict)

```json
{
  "tools": {
    "exec": {
      "security": "allowlist",
      "backgroundMs": 10000,
      "timeoutSec": 300
    },
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "discord": ["user:YOUR_ADMIN_ID"]
      }
    }
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "docker": {
          "network": "none"
        }
      }
    }
  }
}
```

### Development (Relaxed)

```json
{
  "tools": {
    "exec": {
      "security": "full",
      "timeoutSec": 1800
    }
  }
}
```

## 💡 Tips

1. **Production**: Use `security: "allowlist"` for safety
2. **Network Isolation**: Set `network: "none"` in sandbox
3. **Memory Limits**: Set `memory: "1g"` to prevent resource abuse
4. **Elevated Users**: Only allow trusted users for host commands

## 📚 References

- [OpenClaw Exec Docs](https://docs.openclaw.ai/concepts/exec)
- [OpenClaw Sandbox Docs](https://docs.openclaw.ai/concepts/sandbox)
