# Claude Code Auto Mode Guide

> A safer way to skip permission prompts using model-based classifiers.

**Source**: [Anthropic Engineering Blog](https://www.anthropic.com/engineering/claude-code-auto-mode)

## Overview

Auto mode delegates permission decisions to Sonnet 4.6-based classifiers instead of manual approval. It catches dangerous actions while letting safe operations run without prompts.

### The Problem

| Mode | Safety | Friction |
|------|--------|----------|
| Manual prompts | ✅ Safe | ❌ 93% are just approved anyway |
| `--dangerously-skip-permissions` | ❌ Unsafe | ✅ Zero friction |
| Sandbox | ✅ Safe | ❌ High maintenance |
| **Auto mode** | ✅ Safe | ✅ Low friction |

## How It Works

### Two-Layer Defense

1. **Input Layer**: Prompt injection probe scans tool outputs before they enter context
2. **Output Layer**: Transcript classifier (Sonnet 4.6) evaluates each action
   - Stage 1: Fast yes/no filter (cheap)
   - Stage 2: Chain-of-thought reasoning (only if Stage 1 flags)

### Three-Tier Permission System

| Tier | Actions | Classifier? |
|------|---------|-------------|
| 1 | Safe tools allowlist (reads, searches) | ❌ Auto-allow |
| 2 | In-project file edits | ❌ Auto-allow |
| 3 | Shell, external tools, out-of-project | ✅ Classified |

## Threat Model

Auto mode defends against:

- **Overeager behavior**: Agent takes initiative beyond authorization
- **Honest mistakes**: Misunderstands blast radius
- **Prompt injection**: Malicious instructions in content
- **Misalignment**: (Rare) Agent pursues own goals

### Real Incident Examples (Blocked)

| Incident | Why Blocked |
|----------|-------------|
| "Clean up branches" → batch delete remote | Vague request ≠ destructive authorization |
| Auth error → grep for other API tokens | Credential exploration |
| "Cancel my job" → delete closest match | Agent-inferred target |
| Debug script → GitHub Gist | Data exfiltration |
| Deploy failed → retry with `--skip-verification` | Safety bypass |

## Quick Start

```bash
# Run Claude Code in auto mode
claude --permission-mode auto

# Or switch during session
/permission-mode auto
```

## Configuration

### View Current Config

```bash
# See effective config (your settings + defaults)
claude auto-mode config

# See default rules only
claude auto-mode defaults

# Get AI feedback on custom rules
claude auto-mode critique
```

### Customize Environment

Add to `~/.claude/settings.json`:

```json
{
  "autoMode": {
    "environment": [
      "**Source control orgs**: github.com/your-org, github.com/your-username",
      "**Trusted internal domains**: internal.company.com, api.company.com",
      "**Trusted cloud buckets**: s3://your-bucket, gs://your-bucket"
    ]
  }
}
```

### Custom Allow Rules

```json
{
  "autoMode": {
    "allow": [
      "Custom Rule: Description of what to allow and why"
    ]
  }
}
```

### Custom Deny Rules

```json
{
  "autoMode": {
    "soft_deny": [
      "Custom Block: Description of what to block"
    ]
  }
}
```

## Default Rules Summary

### Auto-Allowed (ALLOW)

- Test artifacts (placeholder credentials)
- Local operations within project scope
- Read-only operations (GET, queries)
- Declared dependencies (`npm install`, `pip install -r`)
- Toolchain bootstrap (rustup, pyenv, brew)
- Standard credential flows
- Git push to working branch (not default)

### Auto-Blocked (DENY)

| Category | Examples |
|----------|----------|
| **Destructive Git** | Force push, delete remote branches |
| **External Code** | `curl \| bash`, untrusted repos |
| **Cloud Mass Delete** | S3/GCS bulk operations |
| **Production** | Deploy, migrations |
| **Security Weaken** | Disable TLS, logging, firewalls |
| **Credential Abuse** | Exploration, leakage |
| **Data Exfiltration** | POST to external, public gists |
| **Persistence** | SSH keys, cron jobs, systemd |
| **Self-Modification** | Edit own settings/permissions |

## Performance

| Metric | Real Traffic (n=10K) | Overeager (n=52) | Synthetic Exfil (n=1K) |
|--------|---------------------|------------------|------------------------|
| **FPR** (false blocks) | 0.4% | - | - |
| **FNR** (missed danger) | - | 17% | 5.7% |

> 17% FNR on overeager actions is the honest number. The agent was genuinely trying to help but went past authorization.

## Best Practices

1. **Start with defaults** — They're conservative and battle-tested
2. **Add trusted sources incrementally** — Only add what you need
3. **Use `claude auto-mode critique`** — Get AI feedback on custom rules
4. **Review blocked actions** — Learn what the classifier catches
5. **Combine with git** — In-project edits are reviewable via version control

## Limitations

- 17% of overeager actions may slip through
- Classifier adds latency for Tier 3 actions
- Custom rules require careful wording
- Some legitimate workflows may need exceptions

## See Also

- [Permission Modes Documentation](https://code.claude.com/docs/en/permission-modes)
- [Anthropic Blog Post](https://www.anthropic.com/engineering/claude-code-auto-mode)
- [Claude Opus 4.6 System Card §6.2](https://www-cdn.anthropic.com/14e4fb01875d2a69f646fa5e574dea2b1c0ff7b5.pdf)
