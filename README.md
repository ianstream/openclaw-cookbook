# OpenClaw Cookbook 📖

> Production-tested OpenClaw configuration patterns and best practices

[English](README.md) | [한국어](README.ko.md)

---

## What is this?

A collection of **real-world configuration patterns** for [OpenClaw](https://github.com/openclaw/openclaw) — the open-source AI assistant framework.

**Who is this for?**
- OpenClaw users who want to go beyond basic setup
- Developers looking for production-ready configurations
- Anyone tired of reading docs and wanting copy-paste examples

**What's included?**
- 12 detailed guides with working examples
- Security hardening configurations
- Multi-agent orchestration patterns
- Cost optimization strategies
- Full English/Korean documentation

**Why this exists:**
OpenClaw docs are comprehensive but scattered. This cookbook distills months of production usage into patterns you can use immediately.

---

## 📁 Repository Structure

```
openclaw-cookbook/
├── README.md / README.ko.md
├── docs/
│   ├── token-optimization.md     # Token & context management
│   ├── multi-agent-guide.md      # Multiple agents setup
│   ├── channel-binding-guide.md  # Channel-specific config
│   ├── memory-flush-guide.md     # Pre-compaction memory
│   ├── tts-guide.md              # Text-to-Speech
│   ├── heartbeat-guide.md        # Periodic check-ins
│   ├── cron-guide.md             # Scheduled tasks
│   ├── subagent-guide.md         # Parallel background tasks
│   ├── exec-security-guide.md    # Sandboxing & permissions
│   ├── security-guide.md         # Full security hardening
│   └── auto-mode.md              # Claude Code Auto Mode
└── examples/
    └── ...
```

## 🚀 Installation

**[📖 Installation Guide](docs/installation.md)** | **[📖 설치 가이드](docs/installation.ko.md)**

```bash
# Quick install
npm install -g openclaw
openclaw setup
openclaw auth login anthropic
openclaw gateway start
```

---

## 🎯 Configuration Patterns

### 1. Token Optimization

Reduce costs and manage context window efficiently.

- **Context Pruning**: Remove old tool results
- **Compaction**: Summarize old conversations

→ [Detailed Guide](docs/token-optimization.md)

### 2. Multi-Agent

Run different agents with different models per use case.

- **code**: Coding tasks (Opus)
- **analyst**: Analysis/research (Sonnet)
- **lite**: Simple conversations (Haiku)

→ [Detailed Guide](docs/multi-agent-guide.md)

### 3. Channel Binding

Different agents + system prompts per channel.

→ [Detailed Guide](docs/channel-binding-guide.md)

### 4. Memory Flush

Auto-save important content before compaction.

→ [Detailed Guide](docs/memory-flush-guide.md)

### 5. TTS (Text-to-Speech)

Convert agent responses to voice.

- **Edge TTS**: Free, no API key
- **ElevenLabs**: Premium quality
- **OpenAI TTS**: GPT-4o voice

→ [Detailed Guide](docs/tts-guide.md)

### 6. Heartbeat

Periodic agent check-ins and background tasks.

- **Active Hours**: Only run during specific times
- **Light Context**: Token-saving mode
- **Custom Channel**: Send alerts to specific channel

→ [Detailed Guide](docs/heartbeat-guide.md)

### 7. Cron Jobs

Schedule tasks at specific times.

- **Morning briefs**: Daily summaries
- **Reminders**: One-time alerts
- **Reports**: Weekly/monthly automation

→ [Detailed Guide](docs/cron-guide.md)

### 8. Subagents

Run parallel background tasks with child agents.

- **Orchestrator Pattern**: Main → Workers
- **Cost Optimization**: Cheaper models for subtasks
- **Tool Restrictions**: Limit subagent permissions

→ [Detailed Guide](docs/subagent-guide.md)

### 9. Exec Security

Control command execution and sandboxing.

- **Security Modes**: full / allowlist / deny
- **Elevated Users**: Host command permissions
- **Docker Sandbox**: Isolated execution

→ [Detailed Guide](docs/exec-security-guide.md)

### 10. Security Hardening

Protect your deployment from unauthorized access.

- **Gateway Auth**: Token/password authentication
- **Network Binding**: Loopback vs LAN vs Tailnet
- **DM/Group Policies**: Pairing, allowlists
- **Sensitive Files**: Permissions, secrets management

→ [Detailed Guide](docs/security-guide.md)

### 11. Claude Code Auto Mode

Skip permission prompts safely with model-based classifiers.

- **Two-Layer Defense**: Input probe + Output classifier
- **Three-Tier Permissions**: Safe auto-allow, project edits, classified actions
- **Real Threat Protection**: Overeager behavior, credential exploration, data exfiltration

→ [Detailed Guide](docs/auto-mode.md)

## 🚀 Quick Start (After Installation)

### 1. Copy example config

```bash
cp examples/full-config/openclaw.example.json ~/.openclaw/openclaw.json
```

### 2. Edit your settings

Replace placeholders like `YOUR_DISCORD_BOT_TOKEN`, `YOUR_GUILD_ID`, etc.

### 3. Restart OpenClaw

```bash
openclaw gateway restart
```

## 📚 References

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Discord](https://discord.com/invite/clawd)

## 🌐 Community

Found this useful? Share it!

- **OpenClaw Discord** — Best place for feedback and discussion
- **Twitter/X** — Tag #OpenClaw
- **Reddit** — r/LocalLLaMA, r/ChatGPTCoding
- **Hacker News** — Show HN
- **GeekNews** — 한국 개발자 커뮤니티

## 🤝 Contributing

PRs welcome! If you have useful patterns, feel free to contribute.

## License

MIT
