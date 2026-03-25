# 메모리 플러시 가이드

> 컴팩션 전 중요 내용 자동 저장

[English](memory-flush-guide.md) | [한국어](memory-flush-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 왜 메모리 플러시인가?

컴팩션이 발생하면 오래된 대화가 요약됩니다.
메모리 플러시는 컴팩션 **직전에** 중요한 정보를 파일로 저장합니다:

- 아키텍처 결정
- 버그 수정 내용
- 새로운 패턴/관례

## 📝 기본 설정

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000,
          "prompt": "세션을 검토하고 아키텍처 결정, 버그 수정, 새 패턴이 있으면 memory/YYYY-MM-DD.md에 저장해. 저장할 게 없으면 NO_REPLY.",
          "systemPrompt": "세션이 컴팩션에 가까워졌습니다. 지금 영구 메모리를 저장하세요."
        }
      }
    }
  }
}
```

## ⚙️ 옵션

| 옵션 | 설명 |
|------|------|
| `enabled` | 메모리 플러시 활성화 |
| `softThresholdTokens` | 남은 토큰이 이 값 이하면 플러시 트리거 |
| `prompt` | 플러시 시 에이전트에게 보내는 프롬프트 |
| `systemPrompt` | 시스템 레벨 안내 메시지 |

## 📁 메모리 파일 구조

```
~/.openclaw/workspace/
├── memory/
│   ├── 2026-03-24.md
│   ├── 2026-03-25.md
│   └── ...
├── MEMORY.md          # 장기 기억 (수동 큐레이션)
└── ...
```

## 📝 예제: 저장될 내용

```markdown
# 2026-03-25

## 아키텍처 결정
- BM25-only 검색으로 전환 (하이브리드 제거)
- 이유: 코드 검색은 90%가 키워드 매칭

## 버그 수정
- 메모리 누수 수정: 캐시 TTL 추가
- PR #123 참조

## 새 패턴
- 서브에이전트에 저렴한 모델 사용하는 패턴 도입
```

## 💡 팁

1. **softThresholdTokens**: 4000-8000 권장 (플러시할 여유 확보)
2. **일일 파일**: `memory/YYYY-MM-DD.md` 형식으로 날짜별 정리
3. **MEMORY.md**: 중요한 것만 수동으로 장기 기억에 옮기기
4. **NO_REPLY**: 저장할 게 없으면 조용히 패스

## 🔧 전체 예제

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "compaction": {
        "mode": "default",
        "reserveTokensFloor": 50000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 5000,
          "prompt": "Review the session for architectural decisions, bug fixes, or new patterns. If found, write to memory/YYYY-MM-DD.md. Reply NO_REPLY if nothing to store.",
          "systemPrompt": "Session nearing compaction. Store durable memories now."
        }
      }
    }
  }
}
```

## 📚 참고 문서

- [OpenClaw Compaction 문서](https://docs.openclaw.ai/concepts/compaction)
