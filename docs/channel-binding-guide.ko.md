# 채널 바인딩 가이드

> 채널별로 다른 에이전트와 시스템 프롬프트 설정

[English](channel-binding-guide.md) | [한국어](channel-binding-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 기본 개념

채널 바인딩은 Discord/Slack/Telegram 채널마다:
- 다른 **에이전트** 할당 (다른 모델 사용)
- 다른 **시스템 프롬프트** 설정 (다른 페르소나)

## 📝 기본 설정

### 에이전트 정의

```json
{
  "agents": {
    "list": [
      {
        "id": "code",
        "name": "Code Agent",
        "model": "anthropic/claude-opus-4-6"
      },
      {
        "id": "analyst",
        "name": "Analyst Agent",
        "model": "anthropic/claude-sonnet-4-6"
      },
      {
        "id": "lite",
        "name": "Lite Agent",
        "model": "anthropic/claude-haiku-4-5"
      }
    ]
  }
}
```

### 채널 바인딩

```json
{
  "bindings": [
    {
      "agentId": "code",
      "match": {
        "channel": "discord",
        "peer": {
          "kind": "channel",
          "id": "TECH_CHANNEL_ID"
        }
      }
    },
    {
      "agentId": "analyst",
      "match": {
        "channel": "discord",
        "peer": {
          "kind": "channel",
          "id": "QUANT_CHANNEL_ID"
        }
      }
    }
  ]
}
```

### 채널별 시스템 프롬프트

```json
{
  "channels": {
    "discord": {
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": {
            "TECH_CHANNEL_ID": {
              "allow": true,
              "systemPrompt": "🧙 20년차 시니어 소프트웨어 엔지니어.\n패키지 설치, 의존성 관리, 트러블슈팅 전문."
            },
            "QUANT_CHANNEL_ID": {
              "allow": true,
              "systemPrompt": "📊 월스트리트 20년차 퀀트 투자 구루.\n숫자와 데이터 기반 냉철한 투자 판단."
            }
          }
        }
      }
    }
  }
}
```

## 🔧 전체 예제

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-6"
    },
    "list": [
      {
        "id": "code",
        "model": "anthropic/claude-opus-4-6"
      },
      {
        "id": "analyst",
        "model": "anthropic/claude-sonnet-4-6"
      },
      {
        "id": "lite",
        "model": "anthropic/claude-haiku-4-5"
      },
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "bindings": [
    {
      "agentId": "code",
      "match": { "channel": "discord", "peer": { "kind": "channel", "id": "123456789" } }
    },
    {
      "agentId": "analyst",
      "match": { "channel": "discord", "peer": { "kind": "channel", "id": "987654321" } }
    },
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ]
}
```

## 💡 팁

1. **바인딩 순서**: 위에서 아래로 매칭, 먼저 매칭되면 적용
2. **기본 에이전트**: `"default": true`로 지정
3. **와일드카드**: `"*"` 채널 ID로 모든 채널 매칭
4. **DM 바인딩**: `"peer": { "kind": "direct" }`

## 📚 참고 문서

- [OpenClaw Bindings 문서](https://docs.openclaw.ai/concepts/bindings)
