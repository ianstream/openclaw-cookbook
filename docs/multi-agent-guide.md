# 다중 에이전트 설정 가이드

> 용도별로 다른 모델/설정을 가진 에이전트 운영하기

## 개념

OpenClaw에서 여러 에이전트를 정의하고, 각각 다른 모델과 설정을 부여할 수 있습니다.

## 왜 필요한가?

| 용도 | 추천 모델 | 이유 |
|------|----------|------|
| 코딩/복잡한 추론 | Opus | 최고 성능, 긴 컨텍스트 |
| 분석/리서치 | Sonnet | 비용 대비 성능 |
| 간단한 대화 | Haiku/Sonnet 4.6 | 빠른 응답, 저비용 |

## 설정 방법

### 1. agents.list에 에이전트 정의

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-6",
        "fallbacks": ["google/gemini-2.5-flash"]
      },
      "workspace": "/path/to/workspace"
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
        "model": "anthropic/claude-sonnet-4-6"
      },
      {
        "id": "main",
        "name": "Main Agent",
        "default": true,
        "model": "anthropic/claude-opus-4-8"
      }
    ]
  }
}
```

### 2. 각 필드 설명

| 필드 | 필수 | 설명 |
|------|------|------|
| `id` | ✅ | 에이전트 식별자 (바인딩에서 참조) |
| `name` | ❌ | 표시 이름 |
| `model` | ❌ | 사용할 모델 (없으면 defaults 사용) |
| `default` | ❌ | 기본 에이전트 여부 |
| `workspace` | ❌ | 워크스페이스 경로 |

### 3. 바인딩과 함께 사용

에이전트 정의 후, `bindings`로 채널에 연결:

```json
{
  "bindings": [
    {
      "agentId": "code",
      "match": {
        "channel": "discord",
        "peer": { "kind": "channel", "id": "123456789" }
      }
    }
  ]
}
```

## 실전 예제

### 개발 팀 구성

```json
{
  "agents": {
    "list": [
      {
        "id": "architect",
        "name": "시스템 아키텍트",
        "model": "anthropic/claude-opus-4-8"
      },
      {
        "id": "reviewer",
        "name": "코드 리뷰어",
        "model": "anthropic/claude-sonnet-4-6"
      },
      {
        "id": "helper",
        "name": "빠른 도우미",
        "model": "anthropic/claude-haiku-3-5"
      }
    ]
  }
}
```

## 관련 문서

- [Channel Binding Guide](channel-binding-guide.md)
- [OpenClaw Agents Docs](https://docs.openclaw.ai/concepts/agents)
