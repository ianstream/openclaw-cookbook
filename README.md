# OpenClaw Cookbook 📖

> Production-tested OpenClaw configuration patterns and best practices

[English](README.md) | [한국어](README.ko.md)

---

## 📁 Repository Structure

```
openclaw-cookbook/
├── README.md                    # English
├── README.ko.md                 # 한국어
├── docs/
│   ├── token-optimization.md    # Token optimization (EN)
│   ├── token-optimization.ko.md # 토큰 최적화 (KO)
│   ├── multi-agent-guide.md     # Multi-agent setup
│   ├── channel-binding-guide.md # Channel binding
│   └── memory-flush-guide.md    # Memory flush
└── examples/
    ├── token-optimization/      # Context pruning & compaction
    ├── multi-agent/             # Agent definitions
    ├── channel-binding/         # Channel-specific bindings
    ├── memory-flush/            # Pre-compaction memory save
    └── full-config/             # Complete config example
```

## 🎯 Configuration Patterns

### 1. Token Optimization

Reduce costs and manage context window efficiently.

- **Context Pruning**: Remove old tool results before LLM calls
- **Compaction**: Summarize old conversations when context fills up

→ [Detailed Guide](docs/token-optimization.md)

### 2. Multi-Agent

Run different agents with different models/settings per use case.

- **code**: Coding tasks (Opus)
- **analyst**: Analysis/research (Sonnet)
- **lite**: Simple conversations (Haiku)

→ [Detailed Guide](docs/multi-agent-guide.md)

### 3. Channel Binding

Different agents + system prompts per Discord/Slack channel.

- `#tech` → code agent + developer persona
- `#quant` → analyst agent + investment expert persona

→ [Detailed Guide](docs/channel-binding-guide.md)

### 4. Memory Flush

Auto-save important content before compaction.

- Write to `memory/YYYY-MM-DD.md` before session summary
- Preserve architectural decisions, bug fixes, patterns

→ [Detailed Guide](docs/memory-flush-guide.md)

## 🚀 Quick Start

### 1. Copy example config

```bash
cp examples/full-config/openclaw.example.json ~/.openclaw/openclaw.json
```

### 2. Edit your settings

```bash
# Replace placeholders
YOUR_DISCORD_BOT_TOKEN
YOUR_GUILD_ID
YOUR_DISCORD_USER_ID
TECH_CHANNEL_ID
```

### 3. Restart OpenClaw

```bash
openclaw gateway restart
```

## 📚 References

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Compaction Docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/compaction.md)
- [Session Pruning Docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/session-pruning.md)

## 🤝 Contributing

PRs welcome! If you have useful patterns, feel free to contribute.

## License

MIT
