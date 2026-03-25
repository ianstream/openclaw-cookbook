# Subagent 가이드

> 병렬 백그라운드 작업을 자식 에이전트로 실행

[English](subagent-guide.md) | [한국어](subagent-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 기본 설정

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxConcurrent": 4,
        "runTimeoutSeconds": 900
      }
    }
  }
}
```

## ⚙️ 옵션

| 옵션 | 설명 |
|------|------|
| `maxConcurrent` | 최대 병렬 서브에이전트 수 |
| `runTimeoutSeconds` | 실행당 타임아웃 (기본: 900) |
| `model` | 저렴한 모델 사용 |
| `maxSpawnDepth` | 최대 중첩 깊이 (기본: 2) |
| `maxChildrenPerAgent` | 부모당 최대 자식 수 |
| `archiveAfterMinutes` | 완료된 세션 아카이브 시간 |

## 📝 예제 설정

### 비용 최적화

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "model": "openai/gpt-4.1-mini",
        "maxConcurrent": 4,
        "runTimeoutSeconds": 600
      }
    }
  }
}
```

### 오케스트레이터 패턴

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxSpawnDepth": 2,
        "maxChildrenPerAgent": 5,
        "maxConcurrent": 8
      }
    }
  }
}
```

### 서브에이전트 도구 제한

```json
{
  "tools": {
    "subagents": {
      "tools": {
        "deny": ["gateway", "cron", "message"]
      }
    }
  }
}
```

## 🏗️ 아키텍처

```
메인 에이전트 (Opus)
    ├── 서브에이전트 1 (Sonnet) - 리서치 작업
    ├── 서브에이전트 2 (Sonnet) - 코드 리뷰
    └── 서브에이전트 3 (Haiku) - 단순 조회
```

## 💡 팁

1. **비용 절감**: 루틴 작업에 저렴한 모델 사용
2. **병렬 작업**: 무거운 작업엔 `maxConcurrent: 8`
3. **보안**: 민감한 도구 (`gateway`, `cron`) 서브에이전트에서 차단
4. **정리**: `cleanup: "delete"`로 완료된 세션 자동 삭제
5. **깊이 제한**: `maxSpawnDepth: 2`로 무한 스폰 방지

## 📚 참고 문서

- [OpenClaw Subagent 문서](https://docs.openclaw.ai/concepts/subagents)
