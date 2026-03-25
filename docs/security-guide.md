# Security Guide

> Protect your OpenClaw deployment from unauthorized access

[English](security-guide.md) | [한국어](security-guide.ko.md) | [← Back to Cookbook](../README.md)

---

## ⚠️ Important: Personal Assistant Trust Model

OpenClaw is designed as a **personal assistant** (single-user/single-trust-boundary).

- ✅ One user per gateway
- ❌ NOT a multi-tenant security boundary
- If multiple untrusted users need access → run **separate gateways**

---

## 🔒 Quick Security Audit

```bash
# Run security audit
openclaw security audit

# Deep audit (includes live probe)
openclaw security audit --deep

# Auto-fix some issues
openclaw security audit --fix
```

---

## 🎯 Security Checklist

### 1. Gateway Authentication (Critical!)

**Always set a gateway auth token:**

```json
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "YOUR_LONG_RANDOM_TOKEN"
    }
  }
}
```

Generate a token:
```bash
openclaw doctor --generate-gateway-token
```

### 2. Network Binding

| Mode | Exposure | Use Case |
|------|----------|----------|
| `"loopback"` | Local only | **Default, safest** |
| `"lan"` | Local network | Requires firewall |
| `"tailnet"` | Tailscale only | Recommended for remote |

```json
{
  "gateway": {
    "bind": "loopback"
  }
}
```

⚠️ **Never expose without authentication!**

### 3. DM Access Control

```json
{
  "channels": {
    "discord": {
      "dmPolicy": "pairing",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

| Policy | Description |
|--------|-------------|
| `"pairing"` | Requires approval code (recommended) |
| `"allowlist"` | Only specified users |
| `"open"` | ⚠️ Anyone can DM (dangerous) |
| `"disabled"` | No DMs |

### 4. Group Chat Protection

```json
{
  "channels": {
    "discord": {
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": true,
          "channels": {
            "*": { "allow": true }
          }
        }
      }
    }
  }
}
```

- Always use `requireMention: true` in public groups
- Avoid `groupPolicy: "open"` unless necessary

### 5. Tool Restrictions

Deny dangerous tools for untrusted contexts:

```json
{
  "tools": {
    "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
  }
}
```

### 6. Exec Security

```json
{
  "tools": {
    "exec": {
      "security": "allowlist"
    },
    "elevated": {
      "enabled": false
    }
  }
}
```

| Security Mode | Description |
|---------------|-------------|
| `"full"` | All commands allowed |
| `"allowlist"` | Only whitelisted commands |
| `"deny"` | No exec allowed |

### 7. File Permissions

```bash
# Set correct permissions
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json

# Verify with doctor
openclaw doctor
```

---

## 🛡️ Hardened Baseline Config

Copy this as a starting point:

```json
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "REPLACE_WITH_LONG_RANDOM_TOKEN"
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "messaging",
    "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"],
    "fs": { "workspaceOnly": true },
    "exec": { "security": "deny", "ask": "always" },
    "elevated": { "enabled": false }
  },
  "channels": {
    "discord": {
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

---

## 🔐 Sensitive Files Location

| File | Contains |
|------|----------|
| `~/.openclaw/openclaw.json` | Config, may include tokens |
| `~/.openclaw/credentials/` | Channel credentials |
| `~/.openclaw/agents/*/auth-profiles.json` | API keys, OAuth tokens |
| `~/.openclaw/agents/*/sessions/` | Conversation transcripts |

**Keep these private!** Never commit to git.

---

## 🚫 What NOT to Do

❌ Expose gateway without auth  
❌ Use `dmPolicy: "open"` + tools enabled  
❌ Use `groupPolicy: "open"` in public servers  
❌ Set `elevated.enabled: true` without allowFrom  
❌ Store API keys in plain text in config  
❌ Share `~/.openclaw` directory publicly  

---

## 🆘 If Something Goes Wrong

### 1. Stop Immediately

```bash
openclaw gateway stop
```

### 2. Contain Exposure

```json
{
  "gateway": { "bind": "loopback" },
  "channels": { "discord": { "dmPolicy": "disabled" } }
}
```

### 3. Rotate Credentials

- Gateway token
- API keys
- Discord bot token
- Any exposed secrets

### 4. Audit

```bash
# Check logs
tail -f ~/.openclaw/logs/openclaw.log

# Run security audit
openclaw security audit --deep
```

---

## 📚 References

- [OpenClaw Security Documentation](https://docs.openclaw.ai/gateway/security)
- [Sandboxing Guide](https://docs.openclaw.ai/gateway/sandboxing)
- [Exec Approvals](https://docs.openclaw.ai/tools/exec-approvals)
