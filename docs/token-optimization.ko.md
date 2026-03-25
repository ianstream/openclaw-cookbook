# OpenClaw 토큰 최적화 가이드

> 장기 세션에서 컨텍스트 관리 및 비용 최적화 설정

[English](token-optimization.md) | [한국어](token-optimization.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🔧 두 가지 최적화 메커니즘

| 기능 | Context Pruning | Compaction |
|------|-----------------|------------|
| **대상** | 오래된 tool 결과 | 전체 대화 이력 |
| **방식** | in-memory 제거 (요청당) | 요약 후 압축 |
| **저장** | 세션 파일 변경 없음 | JSONL에 요약 저장 |
| **목적** | 캐시 비용 절감 | 컨텍스트 윈도우 관리 |

## 1️⃣ Context Pruning (세션 프루닝)

### 개념
- LLM 호출 직전에 **오래된 tool 결과만** 메모리에서 제거
- 세션 파일(*.jsonl)은 그대로 유지
- Anthropic 캐시 TTL과 연동하여 비용 절감

### 왜 필요한가?
- Anthropic 프롬프트 캐싱은 TTL 내에서만 적용
- 세션이 TTL 지나면 다음 요청에서 전체 프롬프트 재캐싱
- **프루닝 = 재캐싱 비용 절감**

### 기본 설정

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "5m"
      }
    }
  }
}
```

### 상세 설정 (고급)

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "45m",
        "keepLastAssistants": 3,
        "softTrimRatio": 0.3,
        "hardClearRatio": 0.5,
        "minPrunableToolChars": 50000,
        "softTrim": {
          "maxChars": 4000,
          "headChars": 1500,
          "tailChars": 1500
        },
        "hardClear": {
          "enabled": true,
          "placeholder": "[Old tool result content cleared]"
        }
      }
    }
  }
}
```

### 파라미터 설명

| 파라미터 | 기본값 | 설명 |
|---------|-------|------|
| `mode` | `"off"` | `"cache-ttl"` = TTL 기반 프루닝 활성화 |
| `ttl` | `"5m"` | 마지막 API 호출 후 이 시간 지나면 프루닝 |
| `keepLastAssistants` | `3` | 최근 어시스턴트 응답 N개 보호 |
| `softTrimRatio` | `0.3` | soft-trim 적용 임계치 |
| `hardClearRatio` | `0.5` | hard-clear 적용 임계치 |
| `minPrunableToolChars` | `50000` | 이 크기 이상인 tool 결과만 프루닝 대상 |

### Soft vs Hard 프루닝

| 타입 | 동작 |
|------|------|
| **Soft-trim** | 앞뒤만 유지, 중간에 `...` 삽입 |
| **Hard-clear** | 전체 삭제, placeholder로 대체 |

### 프루닝 제외 대상
- ❌ 사용자 메시지
- ❌ 어시스턴트 메시지  
- ❌ 이미지 포함 tool 결과
- ✅ toolResult 메시지만 대상

### 특정 도구만 프루닝

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "tools": {
          "allow": ["exec", "read"],
          "deny": ["*image*"]
        }
      }
    }
  }
}
```

## 2️⃣ Compaction (컴팩션)

### 개념
- 컨텍스트 윈도우가 꽉 차면 **오래된 대화를 요약**
- 요약본이 세션 JSONL에 저장됨
- 이후 요청은 요약 + 최근 메시지만 사용

### 기본 설정

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "default"
      }
    }
  }
}
```

### 상세 설정

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "model": "openrouter/anthropic/claude-sonnet-4-6",
        "identifierPolicy": "strict",
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000,
          "prompt": "세션 요약 전 중요 내용을 memory/YYYY-MM-DD.md에 저장하세요.",
          "systemPrompt": "컴팩션 임박. 중요 기억을 저장하세요."
        }
      }
    }
  }
}
```

### 파라미터 설명

| 파라미터 | 설명 |
|---------|------|
| `mode` | `"default"` = 자동 컴팩션 |
| `reserveTokensFloor` | 최소 예약 토큰 (컴팩션 후에도 유지) |
| `model` | 컴팩션용 별도 모델 지정 (선택) |
| `identifierPolicy` | `"strict"` = 식별자 보존, `"off"` = 무시 |
| `memoryFlush.enabled` | 컴팩션 전 메모리 저장 턴 실행 |
| `memoryFlush.softThresholdTokens` | 이 토큰 남았을 때 flush 트리거 |

### 수동 컴팩션

```
/compact Focus on decisions and open questions
```

## 🎯 권장 설정

### Claude Code Max 사용자 (비용 무관, 품질 우선)

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "6h",
        "keepLastAssistants": 3
      },
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000
        }
      }
    }
  }
}
```

### API 키 사용자 (비용 절감 우선)

```json
{
  "agents": {
    "defaults": {
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "30m",
        "keepLastAssistants": 2,
        "minPrunableToolChars": 30000,
        "hardClear": {
          "enabled": true
        }
      },
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 30000,
        "model": "anthropic/claude-sonnet-4-6"
      }
    }
  }
}
```

## 🔍 상태 확인

```bash
# 현재 세션 상태
/status

# 컴팩션 횟수 확인 (🧹 Compactions: N)
```

## 📚 참고 문서

- [OpenClaw Compaction 공식 문서](https://github.com/openclaw/openclaw/blob/main/docs/concepts/compaction.md)
- [Session Pruning 공식 문서](https://github.com/openclaw/openclaw/blob/main/docs/concepts/session-pruning.md)
- [Gateway Configuration](https://docs.openclaw.ai/gateway/configuration)

## License

MIT
