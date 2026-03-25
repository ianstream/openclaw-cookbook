# OpenClaw Installation Guide

> Complete setup guide for OpenClaw

[English](installation.md) | [한국어](installation.ko.md) | [← Back to Cookbook](../README.md)

---

## 📋 Prerequisites

- **Node.js**: v20.0.0 or higher
- **npm**: v10.0.0 or higher (comes with Node.js)
- **Operating System**: macOS, Linux, or Windows (WSL2)

### Check versions

```bash
node --version   # v20.0.0+
npm --version    # v10.0.0+
```

## 🚀 Quick Install

### 1. Install OpenClaw

```bash
npm install -g openclaw
```

### 2. Initial Setup

```bash
openclaw setup
```

This will:
- Create `~/.openclaw/` directory
- Generate default `openclaw.json` config
- Guide you through authentication

### 3. Authenticate with Anthropic

```bash
# Option A: Claude Max subscription (OAuth)
openclaw auth login anthropic

# Option B: API Key
openclaw auth add anthropic --api-key YOUR_API_KEY
```

### 4. Start Gateway

```bash
openclaw gateway start
```

## 📱 Discord Bot Setup

### 1. Create Discord Application

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click "New Application"
3. Name your bot (e.g., "OpenClaw Bot")

### 2. Get Bot Token

1. Go to "Bot" section
2. Click "Reset Token"
3. Copy the token

### 3. Enable Intents

In the "Bot" section, enable:
- ✅ Presence Intent
- ✅ Server Members Intent
- ✅ Message Content Intent

### 4. Invite Bot to Server

1. Go to "OAuth2" → "URL Generator"
2. Select scopes: `bot`, `applications.commands`
3. Select permissions: `Send Messages`, `Read Message History`, `Add Reactions`
4. Copy and open the generated URL
5. Select your server and authorize

### 5. Configure OpenClaw

```bash
# Edit config
nano ~/.openclaw/openclaw.json
```

Add Discord configuration:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "groupPolicy": "allowlist",
      "allowFrom": ["YOUR_DISCORD_USER_ID"],
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": false,
          "channels": {
            "*": { "allow": true }
          }
        }
      }
    }
  }
}
```

### 6. Get Your IDs

```bash
# Enable Developer Mode in Discord:
# User Settings → App Settings → Advanced → Developer Mode

# Right-click your server → Copy Server ID (Guild ID)
# Right-click yourself → Copy User ID
```

### 7. Restart Gateway

```bash
openclaw gateway restart
```

## 🔧 Common Commands

| Command | Description |
|---------|-------------|
| `openclaw status` | Check gateway status |
| `openclaw gateway start` | Start gateway |
| `openclaw gateway stop` | Stop gateway |
| `openclaw gateway restart` | Restart gateway |
| `openclaw auth list` | List auth profiles |
| `openclaw doctor` | Diagnose issues |

## 🐛 Troubleshooting

### Gateway won't start

```bash
# Check logs
tail -f ~/.openclaw/logs/openclaw.log

# Run doctor
openclaw doctor
```

### Discord bot not responding

1. Check bot is online in Discord
2. Verify token is correct
3. Check `allowFrom` includes your user ID
4. Ensure `requireMention` matches your expectation

### Authentication errors

```bash
# Re-authenticate
openclaw auth login anthropic

# Or check auth status
openclaw auth list
```

## 📚 Next Steps

After installation:

1. [Token Optimization](token-optimization.md) - Save costs
2. [Multi-Agent Setup](multi-agent-guide.md) - Multiple personas
3. [Channel Binding](channel-binding-guide.md) - Per-channel config

## 📚 References

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
