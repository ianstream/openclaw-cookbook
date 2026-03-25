# OpenClaw Claude 관련 설정 옵션 레퍼런스

> 출처: https://docs.openclaw.ai/gateway/configuration-reference, https://docs.openclaw.ai/providers/anthropic, https://docs.openclaw.ai/concepts/retry, https://docs.openclaw.ai/concepts/model-failover
> 조사일: 2026-03-25

---

## `agents.defaults.models`

### 설명
- 모델 카탈로그 및 `/model` 명령의 허용 목록(allowlist)
- 각 모델 항목에 별칭(alias)과 파라미터(params) 설정 가능
- `params` 에 `temperature`, `maxTokens`, `cacheRetention`, `context1m` 등 프로바이더별 파라미터 지정

### 타입
- object (키: `provider/model` 문자열)

### 기본값
- 미설정 시 allowlist 없음 (모든 모델 허용)

### 예제
```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "alias": "opus",
          "params": {
            "cacheRetention": "long",
            "context1m": true,
            "thinking": "adaptive"
          }
        },
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet",
          "params": {
            "cacheRetention": "short",
            "fastMode": true
          }
        }
      }
    }
  }
}
```

### 주의사항
- `agents.defaults.models` 를 설정하면 allowlist로 동작 → 목록에 없는 모델은 사용 불가
- `params` 병합 순서: `agents.defaults.models[model].params` (기본) → `agents.list[].params` (오버라이드)

---

## `agents.defaults.models["anthropic/<model>"].params.cacheRetention`

### 설명
- Anthropic API의 프롬프트 캐싱 TTL 설정
- API Key 인증에서만 동작 (setup-token/OAuth 미지원)

### 타입
- string: `"none"` | `"short"` | `"long"`

### 기본값
- API Key 인증 시: `"short"` (5분 캐시 자동 적용)
- subscription/OAuth 인증 시: 캐시 설정 무시됨

### 값 설명
| 값 | 캐시 시간 | 설명 |
|----|-----------|------|
| `"none"` | 없음 | 캐싱 비활성화 |
| `"short"` | 5분 | API Key 인증 기본값 |
| `"long"` | 1시간 | Extended cache (beta flag 필요) |

### 예제
```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": { "cacheRetention": "long" }
        }
      }
    }
  }
}
```

### 레거시
- 구버전 `cacheControlTtl` 파라미터도 지원:
  - `"5m"` → `"short"`
  - `"1h"` → `"long"`
- 마이그레이션 권장

### Bedrock 참고
- `amazon-bedrock/*anthropic.claude*` 모델은 `cacheRetention` pass-through 지원
- 비-Anthropic Bedrock 모델은 강제로 `"none"` 적용

---

## `agents.defaults.models["anthropic/<model>"].params.thinking`

### 설명
- Anthropic Claude 모델의 thinking(추론) 모드 설정
- Claude 4.6 모델은 기본값이 `"adaptive"`

### 타입
- string: `"off"` | `"low"` | `"medium"` | `"high"` | `"adaptive"`

### 기본값
- Claude 4.6 계열: `"adaptive"` (명시적 설정 없으면 adaptive로 동작)
- 다른 모델: 없음 (thinking 미지원)

### 예제
```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "params": { "thinking": "low" }
        }
      }
    }
  }
}
```

### 관련
- 채팅 중 `/think:<level>` 으로 per-message 오버라이드 가능
- Anthropic 공식 docs: [Adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking), [Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

---

## `agents.defaults.thinkingDefault`

### 설명
- 모든 모델의 기본 thinking 레벨 설정

### 타입
- string: `"off"` | `"low"` | `"medium"` | `"high"` | `"adaptive"`

### 기본값
- 미설정 (모델별 defaults 따름)

### 예제
```json
{
  "agents": {
    "defaults": {
      "thinkingDefault": "low"
    }
  }
}
```

---

## `agents.defaults.models["anthropic/<model>"].params.fastMode`

### 설명
- Anthropic API의 Priority Tier (서비스 티어) 설정
- `/fast` 채팅 명령에 대응
- API Key 전용 (OAuth/subscription 미지원)

### 타입
- boolean

### 기본값
- `false`

### 값 매핑
- `true` → `service_tier: "auto"` (Priority Tier 활성화 시도)
- `false` → `service_tier: "standard_only"`

### 예제
```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "params": { "fastMode": true }
        }
      }
    }
  }
}
```

### 주의사항
- 직접 `api.anthropic.com` 요청에만 적용 (프록시/게이트웨이 경유 시 무시)
- Priority Tier 없는 계정에서 `"auto"`는 `"standard"`로 fallback될 수 있음

---

## `agents.defaults.models["anthropic/<model>"].params.context1m`

### 설명
- Anthropic 1M 컨텍스트 윈도우 베타 기능 활성화
- 지원 모델: Opus, Sonnet (일부)

### 타입
- boolean

### 기본값
- `false` (명시적으로 `true` 설정 필요)

### 예제
```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": { "context1m": true }
        }
      }
    }
  }
}
```

### 주의사항
- `anthropic-beta: context-1m-2025-08-07` 헤더 자동 추가
- OAuth/subscription(`sk-ant-oat-*`) 인증에서는 자동으로 스킵됨
- 계정에 Extra Usage 권한 필요 (없으면 HTTP 429 반환)

---

## `agents.defaults.model`

### 설명
- 기본 사용 모델 및 fallback 모델 목록 설정

### 타입
- string (`"provider/model"`) 또는 object (`{ primary, fallbacks }`)

### 기본값
- 없음 (설정 필수)

### 예제
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-6", "openai/gpt-5.2"]
      }
    }
  }
}
```

### 모델 선택 순서
1. `model.primary` 설정 모델
2. `model.fallbacks` 순서대로 시도
3. 프로바이더 내 auth profile rotation

---

## `agents.defaults.timeoutSeconds`

### 설명
- 에이전트 실행 최대 대기 시간 (초)

### 타입
- number

### 기본값
- `600` (10분)

### 예제
```json
{
  "agents": {
    "defaults": {
      "timeoutSeconds": 600
    }
  }
}
```

---

## `agents.defaults.contextTokens`

### 설명
- 컨텍스트 윈도우 최대 토큰 수

### 타입
- number

### 기본값
- `200000`

### 예제
```json
{
  "agents": {
    "defaults": {
      "contextTokens": 200000
    }
  }
}
```

---

## `agents.defaults.maxConcurrent`

### 설명
- 세션 간 최대 병렬 에이전트 실행 수 (각 세션은 직렬화됨)

### 타입
- number

### 기본값
- `1`

### 예제
```json
{
  "agents": {
    "defaults": {
      "maxConcurrent": 3
    }
  }
}
```

---

## `models.providers.anthropic`

### 설명
- Anthropic 프로바이더 커스텀 설정 (baseUrl, apiKey, 헤더 등)
- `models.json` 에 저장 (에이전트 디렉토리)

### 타입
- object

### 기본값
- 없음 (표준 Anthropic API 사용)

### 예제 (커스텀 base URL / 프록시)
```json
{
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://my-anthropic-proxy.example.com/v1",
        "apiKey": "${ANTHROPIC_API_KEY}",
        "headers": {
          "x-custom-header": "value"
        }
      }
    }
  }
}
```

### 예제 (환경변수로 API Key)
```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-..."
  },
  "agents": {
    "defaults": {
      "model": { "primary": "anthropic/claude-opus-4-6" }
    }
  }
}
```

### SecretRef 사용
```json
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": { "source": "env", "provider": "default", "id": "ANTHROPIC_API_KEY" }
      }
    }
  }
}
```

---

## `channels.<channel>.retry` (Rate Limiting / Retry)

### 설명
- 채널별 HTTP 요청 재시도 정책 (메시지 전송, 미디어 업로드, 반응 등)
- Discord: HTTP 429 (rate-limit)에만 재시도
- Telegram: 429, 타임아웃, 연결 오류에 재시도

### 타입
- object

### 기본값
```json
{
  "attempts": 3,
  "minDelayMs": 500,
  "maxDelayMs": 30000,
  "jitter": 0.1
}
```
- Discord minDelayMs: `500`
- Telegram minDelayMs: `400`

### 예제
```json
{
  "channels": {
    "telegram": {
      "retry": {
        "attempts": 3,
        "minDelayMs": 400,
        "maxDelayMs": 30000,
        "jitter": 0.1
      }
    },
    "discord": {
      "retry": {
        "attempts": 3,
        "minDelayMs": 500,
        "maxDelayMs": 30000,
        "jitter": 0.1
      }
    }
  }
}
```

### 동작 방식
- 지수 백오프(exponential backoff) + 지터(jitter)
- `retry_after` 헤더 있으면 우선 사용
- 복합 플로우(multi-step)에서 완료된 단계는 재시도 안 함

---

## `auth.cooldowns` (Auth Profile Cooldown / Backoff)

### 설명
- 인증 프로파일 실패 시 cooldown 및 billing 비활성화 설정
- 429/타임아웃 → 지수 백오프로 cooldown 후 다음 프로파일로 rotation

### 타입
- object

### 기본값
- Cooldown 지수 백오프: 1분 → 5분 → 25분 → 1시간 (상한)
- Billing backoff: 5시간 시작, 2배씩 증가, 24시간 상한

### 예제
```json
{
  "auth": {
    "cooldowns": {
      "billingBackoffHours": 5,
      "billingBackoffHoursByProvider": {
        "anthropic": 6
      },
      "billingMaxHours": 24,
      "failureWindowHours": 24
    }
  }
}
```

### 관련 필드
- `auth.profiles` / `auth.order` - 프로파일 메타데이터 & 라우팅
- `auth.cooldowns.billingBackoffHours` - 과금 실패 초기 backoff (기본: 5h)
- `auth.cooldowns.billingBackoffHoursByProvider` - 프로바이더별 billing backoff 오버라이드
- `auth.cooldowns.billingMaxHours` - billing backoff 최대값 (기본: 24h)
- `auth.cooldowns.failureWindowHours` - 실패 카운터 리셋 윈도우 (기본: 24h)

### Cooldown 상태 저장 위치
`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`의 `usageStats`

---

## `agents.defaults.model.fallbacks` (Model Failover)

### 설명
- 기본 모델 실패 시 순서대로 시도할 대체 모델 목록
- auth 실패, rate limit, 타임아웃으로 모든 프로파일 소진 시 fallback 진행

### 타입
- array of string

### 기본값
- `[]` (없음)

### 예제
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": [
          "anthropic/claude-sonnet-4-6",
          "openai/gpt-5.2",
          "openrouter/anthropic/claude-sonnet-4-6"
        ]
      }
    }
  }
}
```

### Failover 트리거 조건
- 인증 오류 (401)
- Rate limit (429)
- 타임아웃 (profile rotation 소진 후)
- Format/invalid-request 오류 (일부)

### 세션 스티키니스 (Cache 최적화)
- 세션 내에서는 동일 auth profile 유지 (캐시 warm 유지)
- `/new` / `/reset`, compaction 완료, cooldown/disabled 시 rotation

---

## 전체 Claude 관련 설정 요약

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-6"]
      },
      "models": {
        "anthropic/claude-opus-4-6": {
          "alias": "opus",
          "params": {
            "cacheRetention": "long",
            "thinking": "adaptive",
            "context1m": false,
            "fastMode": false
          }
        },
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet",
          "params": {
            "cacheRetention": "short",
            "thinking": "low",
            "fastMode": true
          }
        }
      },
      "thinkingDefault": "adaptive",
      "timeoutSeconds": 600,
      "contextTokens": 200000,
      "maxConcurrent": 1
    }
  },
  "auth": {
    "cooldowns": {
      "billingBackoffHours": 5,
      "billingMaxHours": 24,
      "failureWindowHours": 24
    }
  },
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-..."
  }
}
```

---

## 참고 링크

- [Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference)
- [Anthropic Provider](https://docs.openclaw.ai/providers/anthropic)
- [Models CLI](https://docs.openclaw.ai/concepts/models)
- [Model Failover](https://docs.openclaw.ai/concepts/model-failover)
- [Retry Policy](https://docs.openclaw.ai/concepts/retry)
- [Anthropic Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- [Anthropic Extended Thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)
