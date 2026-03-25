# 채널 바인딩 설정 가이드

> Discord/Slack 채널별로 다른 에이전트와 페르소나 설정하기

## 개념

하나의 OpenClaw 인스턴스에서 채널마다 다른 에이전트와 시스템 프롬프트를 적용합니다.

## 왜 필요한가?

- `#tech` 채널에서는 개발자로 동작
- `#quant` 채널에서는 투자 전문가로 동작
- `#casual` 채널에서는 친근한 대화 상대로 동작

각 채널의 목적에 맞는 페르소나로 대화 품질 향상!

## 설정 방법

### 1. bindings 설정

어떤 채널에 어떤 에이전트를 매칭할지 정의:

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
    },
    {
      "agentId": "main",
      "match": {
        "channel": "discord"
      }
    }
  ]
}
```

**주의**: 바인딩은 순서대로 매칭됩니다. 구체적인 것을 먼저, 일반적인 것(fallback)을 나중에!

### 2. 채널별 시스템 프롬프트

`channels.discord.guilds.[GUILD_ID].channels`에서 채널별 프롬프트 정의:

```json
{
  "channels": {
    "discord": {
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": false,
          "channels": {
            "TECH_CHANNEL_ID": {
              "allow": true,
              "systemPrompt": "🧙 20년차 시니어 소프트웨어 엔지니어.\n\n## 핵심 역량\n- 풀스택 개발, 아키텍처 설계\n- 트러블슈팅, 성능 최적화"
            },
            "QUANT_CHANNEL_ID": {
              "allow": true,
              "systemPrompt": "📊 월스트리트 퀀트 투자 전문가.\n\n## 핵심 역량\n- 데이터 기반 투자 분석\n- 백테스팅, 리스크 관리"
            },
            "*": {
              "allow": true
            }
          }
        }
      }
    }
  }
}
```

### 3. 각 필드 설명

#### bindings

| 필드 | 설명 |
|------|------|
| `agentId` | agents.list에 정의된 에이전트 id |
| `match.channel` | 채널 타입 (discord, slack, telegram 등) |
| `match.peer.kind` | `channel` 또는 `user` |
| `match.peer.id` | 채널/유저 ID |

#### channels.[provider].guilds

| 필드 | 설명 |
|------|------|
| `requireMention` | true면 @멘션 필요, false면 모든 메시지에 응답 |
| `channels.[ID].allow` | 채널 허용 여부 |
| `channels.[ID].systemPrompt` | 해당 채널에서 사용할 시스템 프롬프트 |

## 실전 예제

### 다목적 Discord 서버

```json
{
  "bindings": [
    { "agentId": "code", "match": { "channel": "discord", "peer": { "kind": "channel", "id": "111" } } },
    { "agentId": "analyst", "match": { "channel": "discord", "peer": { "kind": "channel", "id": "222" } } },
    { "agentId": "coach", "match": { "channel": "discord", "peer": { "kind": "channel", "id": "333" } } },
    { "agentId": "main", "match": { "channel": "discord" } }
  ],
  "channels": {
    "discord": {
      "guilds": {
        "GUILD_ID": {
          "channels": {
            "111": {
              "allow": true,
              "systemPrompt": "🧙 시니어 개발자. 코드 품질, 아키텍처, 트러블슈팅 전문."
            },
            "222": {
              "allow": true,
              "systemPrompt": "📊 퀀트 전문가. 데이터 기반 투자, 백테스팅, 리스크 관리."
            },
            "333": {
              "allow": true,
              "systemPrompt": "💪 라이프 코치. 목표 설정, 습관 형성, 동기부여."
            },
            "*": { "allow": true }
          }
        }
      }
    }
  }
}
```

## 팁

### 채널 ID 찾기 (Discord)

1. Discord 설정 → 고급 → 개발자 모드 활성화
2. 채널 우클릭 → "ID 복사"

### 시스템 프롬프트 작성 팁

```markdown
🧙 [역할/페르소나]

## 핵심 역량
- 능력 1
- 능력 2

## 대화 스타일
- 스타일 1
- 스타일 2

## 참고 파일 (선택)
📖 상세: path/to/README.md
```

## 관련 문서

- [Multi-Agent Guide](multi-agent-guide.md)
- [OpenClaw Bindings Docs](https://docs.openclaw.ai/concepts/bindings)
