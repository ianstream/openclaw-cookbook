# Claude 모델 최적화 가이드

> OpenClaw에서 Anthropic Claude API 최적화 설정

[English](claude-optimization.md) | [한국어](claude-optimization.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 주요 최적화 설정

### 1. 프롬프트 캐싱 (`cacheRetention`)

반복되는 프롬프트를 캐싱하여 비용과 레이턴시 절감.

| 값 | TTL | 설명 |
|----|-----|------|
| `"none"` | - | 캐싱 비활성화 |
| `"short"` | 5분 | API Key 기본값 |
| `"long"` | 1시간 | 확장 캐시 (베타) |

⚠️ **API Key 전용** — OAuth/setup-token에서는 무시됨

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "cacheRetention": "long"
          }
        }
      }
    }
  }
}
```

### 2. Adaptive Thinking (`thinking`)

Claude가 작업 복잡도에 따라 "사고" 여부와 양을 자동 결정.

| 값 | 설명 |
|----|------|
| `"off"` | 사고 없음 |
| `"low"` | 최소 사고 |
| `"medium"` | 중간 수준 |
| `"high"` | 항상 깊이 사고 |
| `"adaptive"` | 자동 결정 (4.6 기본값) |

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "thinking": "adaptive"
          }
        }
      }
    }
  }
}
```

### 3. Priority Tier (`fastMode`)

Anthropic 우선 처리 티어 활성화.

⚠️ **API Key 전용**

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "params": {
            "fastMode": true
          }
        }
      }
    }
  }
}
```

### 4. 1M 컨텍스트 윈도우 (`context1m`)

100만 토큰 컨텍스트 윈도우 활성화 (베타).

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "params": {
            "context1m": true
          }
        }
      }
    }
  }
}
```

### 5. 모델 Fallback

기본 모델 실패 시 자동 대체.

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": [
          "anthropic/claude-sonnet-4-6",
          "google/gemini-2.5-flash"
        ]
      }
    }
  }
}
```

### 6. 모델 별칭 (Alias)

빠른 모델 전환을 위한 단축 이름.

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-6": {
          "alias": "opus"
        },
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet"
        }
      }
    }
  }
}
```

## 🎯 권장 설정

### 고성능 (Max 플랜)

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
            "thinking": "adaptive"
          }
        }
      }
    }
  }
}
```

### 비용 최적화 (API Key)

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-6",
        "fallbacks": ["anthropic/claude-haiku-4-5"]
      },
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "alias": "sonnet",
          "params": {
            "cacheRetention": "long",
            "thinking": "low"
          }
        }
      }
    }
  }
}
```

## 📊 모델 비교

| 모델 | 가격 (입력/출력) | 컨텍스트 | 적합한 용도 |
|-----|-----------------|---------|------------|
| **Opus 4.6** | $5/$25 per MTok | 1M | 복잡한 추론, 에이전트 |
| **Sonnet 4.6** | $3/$15 per MTok | 1M | 속도+품질 균형 |
| **Haiku 4.5** | $1/$5 per MTok | 200k | 빠른 단순 작업 |

## 📚 참고 문서

- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [Anthropic Adaptive Thinking](https://docs.anthropic.com/en/docs/build-with-claude/adaptive-thinking)
- [OpenClaw Anthropic Provider](https://docs.openclaw.ai/providers/anthropic)
