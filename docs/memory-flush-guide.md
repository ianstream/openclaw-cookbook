# 메모리 플러시 설정 가이드

> 컴팩션 전 중요 내용을 자동으로 파일에 저장하기

## 개념

세션이 길어져서 컴팩션(대화 요약)이 발생하기 전에, 에이전트에게 중요한 내용을 파일로 저장하도록 지시합니다.

## 왜 필요한가?

컴팩션은 대화를 요약하지만, 모든 세부사항을 보존하지 않습니다:

- ❌ 아키텍처 결정의 구체적 이유
- ❌ 버그 픽스의 상세한 맥락
- ❌ 새로 발견한 패턴/인사이트

메모리 플러시로 이런 중요한 정보를 영구 저장!

## 설정 방법

### 기본 설정

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
          "prompt": "Review the session for any architectural decisions, bug fixes, or new patterns. Write notes to memory/YYYY-MM-DD.md. Reply NO_REPLY if nothing to store.",
          "systemPrompt": "Session nearing compaction. Store durable memories now."
        }
      }
    }
  }
}
```

### 각 필드 설명

| 필드 | 기본값 | 설명 |
|------|-------|------|
| `enabled` | `false` | 메모리 플러시 활성화 |
| `softThresholdTokens` | - | 남은 토큰이 이 값 이하면 플러시 트리거 |
| `prompt` | - | 에이전트에게 전달할 지시문 |
| `systemPrompt` | - | 시스템 프롬프트로 추가될 메시지 |

## 실전 예제

### 개발 작업용

```json
{
  "memoryFlush": {
    "enabled": true,
    "softThresholdTokens": 4000,
    "prompt": "세션을 검토하고 다음을 memory/YYYY-MM-DD.md에 저장:\n- 아키텍처 결정과 그 이유\n- 버그 픽스와 근본 원인\n- 새로 발견한 패턴\n- TODO 항목\n\n저장할 내용 없으면 NO_REPLY.",
    "systemPrompt": "컴팩션 임박. 중요 내용을 저장하세요."
  }
}
```

### brv (context tree) 연동

```json
{
  "memoryFlush": {
    "enabled": true,
    "softThresholdTokens": 4000,
    "prompt": "Review the session for any architectural decisions, bug fixes, or new patterns. If found, run 'brv curate \"<summary of change>\"' to update the context tree. Also write personal notes to memory/YYYY-MM-DD.md. Reply NO_REPLY if nothing to store.",
    "systemPrompt": "Session nearing compaction. Store durable memories now."
  }
}
```

### 최소 설정

```json
{
  "memoryFlush": {
    "enabled": true,
    "softThresholdTokens": 5000,
    "prompt": "Save important notes to memory/YYYY-MM-DD.md. Reply NO_REPLY if nothing."
  }
}
```

## 워크스페이스 구조

메모리 플러시가 잘 동작하려면 워크스페이스에 `memory/` 디렉토리가 필요합니다:

```
~/.openclaw/workspace/
├── AGENTS.md           # 에이전트 지침
├── MEMORY.md           # 장기 기억 (수동 관리)
├── memory/             # 일일 메모 (자동 생성)
│   ├── 2026-03-24.md
│   └── 2026-03-25.md
└── ...
```

## 팁

### softThresholdTokens 설정

- 너무 낮으면: 플러시 턴에서 토큰 부족
- 너무 높으면: 불필요하게 자주 플러시
- **권장**: 3000-5000 토큰

### prompt 작성 팁

1. **구체적으로**: "중요한 거 저장" ❌ → "아키텍처 결정, 버그 픽스 저장" ✅
2. **파일 경로 명시**: `memory/YYYY-MM-DD.md` 형식 권장
3. **NO_REPLY 지시**: 저장할 게 없으면 빈 응답 방지

### 확인 방법

```bash
# 메모리 파일 확인
ls ~/.openclaw/workspace/memory/

# 오늘 메모 확인
cat ~/.openclaw/workspace/memory/$(date +%Y-%m-%d).md
```

## 관련 문서

- [Token Optimization Guide](https://github.com/ianstream/openclaw-token-optimization)
- [OpenClaw Memory Docs](https://docs.openclaw.ai/concepts/memory)
- [OpenClaw Compaction Docs](https://docs.openclaw.ai/concepts/compaction)
