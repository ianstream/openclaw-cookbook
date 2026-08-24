# 다중 에이전트 가이드

> 용도별로 다른 모델의 에이전트 설정

[English](multi-agent-guide.md) | [한국어](multi-agent-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 왜 다중 에이전트인가?

| 에이전트 | 모델 | 용도 | 비용 |
|---------|------|------|------|
| **code** | Opus | 복잡한 코딩, 아키텍처 | 높음 |
| **analyst** | Sonnet | 분석, 리서치 | 중간 |
| **lite** | Haiku | 간단한 대화, Q&A | 낮음 |

## 📝 기본 설정

```json
{
  "agents": {
    "defaults": {
      "workspace": "/Users/yourname/.openclaw/workspace",
      "model": "anthropic/claude-sonnet-4-6"
    },
    "list": [
      {
        "id": "code",
        "name": "Code Agent",
        "model": "anthropic/claude-opus-4-8"
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
      },
      {
        "id": "main",
        "name": "Main Agent",
        "default": true
      }
    ]
  }
}
```

## ⚙️ 에이전트 옵션

| 옵션 | 설명 |
|------|------|
| `id` | 고유 식별자 (바인딩에서 사용) |
| `name` | 표시 이름 |
| `model` | 사용할 모델 |
| `default` | 기본 에이전트 여부 |
| `workspace` | 작업 디렉토리 |
| `heartbeat` | 에이전트별 하트비트 설정 |

## 📝 에이전트별 설정 오버라이드

```json
{
  "agents": {
    "list": [
      {
        "id": "code",
        "model": "anthropic/claude-opus-4-8",
        "compaction": {
          "reserveTokensFloor": 80000
        },
        "heartbeat": {
          "every": "2h"
        }
      },
      {
        "id": "lite",
        "model": "anthropic/claude-haiku-4-5",
        "compaction": {
          "reserveTokensFloor": 20000
        }
      }
    ]
  }
}
```

## 🔧 채널 바인딩과 함께 사용

```json
{
  "bindings": [
    {
      "agentId": "code",
      "match": { "channel": "discord", "peer": { "kind": "channel", "id": "TECH_CHANNEL" } }
    },
    {
      "agentId": "analyst",
      "match": { "channel": "discord", "peer": { "kind": "channel", "id": "QUANT_CHANNEL" } }
    },
    {
      "agentId": "lite",
      "match": { "channel": "discord", "peer": { "kind": "channel", "id": "CASUAL_CHANNEL" } }
    },
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ]
}
```

## 💡 비용 최적화 팁

1. **일상 대화**: Haiku (가장 저렴)
2. **분석/리서치**: Sonnet (균형)
3. **복잡한 코딩**: Opus (필요할 때만)
4. **서브에이전트**: 더 저렴한 모델 지정

## 📚 참고 문서

- [OpenClaw Agents 문서](https://docs.openclaw.ai/concepts/agents)
