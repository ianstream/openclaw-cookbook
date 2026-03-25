# Anthropic Claude API 최적화 설정 가이드

> 작성일: 2026-03-25  
> 출처: [Anthropic 공식 문서](https://platform.claude.com/docs)

---

## 1. Prompt Caching

### 개념
- API 요청의 반복되는 prefix를 캐싱하여 재처리 없이 재사용
- KV cache 표현과 암호화 해시를 저장 (원본 텍스트는 저장 안 함)
- 2가지 방식: **Automatic caching** (자동) / **Explicit cache breakpoints** (수동)

### 왜 필요한가
- 처리 시간 대폭 감소 (캐시 히트 시 CPU 처리 불필요)
- 비용 절감: 캐시 히트된 입력 토큰은 일반 입력보다 저렴
- 반복 작업 / 긴 시스템 프롬프트 / 멀티턴 대화에 특히 효과적

### 설정 방법

**방법 1: Automatic Caching (권장 - 멀티턴 대화)**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "cache_control": {"type": "ephemeral"},
  "system": "당신은 긴 시스템 프롬프트를 가진 어시스턴트입니다...",
  "messages": [
    {"role": "user", "content": "질문"}
  ]
}
```

**방법 2: Explicit Cache Breakpoints (세밀한 제어)**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "system": [
    {
      "type": "text",
      "text": "캐시할 긴 시스템 프롬프트...",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "긴 문서 내용...",
          "cache_control": {"type": "ephemeral"}
        },
        {"type": "text", "text": "이 문서를 요약해줘"}
      ]
    }
  ]
}
```

### 권장 값
- **캐시 TTL**: `ephemeral` (현재 유일한 타입) — 세션 내 유지
- **최소 캐시 가능 크기**: 일반적으로 1,024 tokens 이상
- **Automatic**: 멀티턴 대화, 성장하는 대화 히스토리 캐싱
- **Explicit**: 특정 문서, 코드베이스, 긴 컨텍스트 고정 캐싱

---

## 2. Extended Thinking (수동 모드)

### 개념
- Claude가 내부 추론 과정을 `thinking` 블록으로 출력
- 최종 응답 전 단계별 사고 과정을 거침
- `budget_tokens`으로 사고에 사용할 최대 토큰 수 제한

> ⚠️ **주의**: Opus 4.6, Sonnet 4.6에서는 `budget_tokens` 방식이 **deprecated**  
> → 대신 **Adaptive Thinking** 사용 권장

### 지원 모델
- `claude-sonnet-4-6` (manual + adaptive 모두)
- `claude-opus-4-5-20251101`, `claude-opus-4-1-20250805`
- `claude-sonnet-4-5-20250929`
- `claude-haiku-4-5-20251001`

### 설정 방법

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 16000,
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  },
  "messages": [
    {
      "role": "user",
      "content": "복잡한 수학 문제나 추론이 필요한 질문"
    }
  ]
}
```

**응답 구조:**
```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "단계별 추론 과정...",
      "signature": "WaUjzkypQ2m..."
    },
    {
      "type": "text",
      "text": "최종 답변"
    }
  ]
}
```

### 권장 값
- `budget_tokens`: 복잡도에 따라 1,000 ~ 50,000
  - 간단한 추론: 1,000 ~ 5,000
  - 복잡한 분석: 5,000 ~ 20,000
  - 매우 어려운 문제: 20,000 ~ 50,000
- `max_tokens`는 `budget_tokens` + 예상 응답 길이보다 커야 함

---

## 3. Adaptive Thinking (최신 권장)

### 개념
- Extended thinking의 최신 방식 (Opus 4.6, Sonnet 4.6용)
- Claude가 요청 복잡도에 따라 thinking 사용 여부와 양을 **자동 결정**
- `effort` 파라미터로 전체적인 노력 수준 조절
- 자동으로 **Interleaved thinking** 활성화 (도구 호출 사이에도 사고 가능)

### 왜 필요한가
- 고정 `budget_tokens` 대비 성능 향상 (특히 bimodal 작업)
- 에이전틱 워크플로우에 최적화
- 예측 가능한 레이턴시가 필요 없을 때 더 나은 성능

### 설정 방법

```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 16000,
  "thinking": {
    "type": "adaptive"
  },
  "messages": [
    {
      "role": "user",
      "content": "복잡한 에이전트 작업"
    }
  ]
}
```

**effort 파라미터 조절:**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 16000,
  "thinking": {
    "type": "adaptive"
  },
  "effort": "low",
  "messages": [...]
}
```

### 권장 값
| effort 레벨 | 설명 | 사용 시기 |
|------------|------|---------|
| `high` (기본) | 거의 항상 thinking 사용 | 복잡한 작업, 에이전트 |
| `medium` | 중간 복잡도 판단 | 일반 작업 |
| `low` | 단순 문제는 thinking 스킵 | 빠른 응답 필요 시 |

---

## 4. Streaming

### 개념
- `"stream": true`로 서버-사이드 이벤트(SSE) 방식으로 응답 스트리밍
- 전체 응답 완료를 기다리지 않고 토큰 단위로 수신

### 왜 필요한가
- 사용자 체감 응답 속도 대폭 향상
- 긴 응답 생성 시 타임아웃 방지
- 실시간 인터페이스 구현 필수

### 설정 방법

**Python SDK:**
```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    max_tokens=1024,
    messages=[{"role": "user", "content": "긴 응답 요청"}],
    model="claude-opus-4-6",
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

**Raw API:**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "stream": true,
  "messages": [{"role": "user", "content": "질문"}]
}
```

**SSE 이벤트 타입:**
- `message_start` — 메시지 시작
- `content_block_start` — 콘텐츠 블록 시작
- `content_block_delta` — 토큰 델타 (실제 텍스트)
- `content_block_stop` — 블록 종료
- `message_delta` — stop_reason, usage
- `message_stop` — 완료

### 권장 값
- **기본 사용**: Python/TypeScript SDK의 `.stream()` 메서드 사용 (에러 처리 자동화)
- **Streaming + Extended Thinking**: thinking 블록도 스트리밍 가능
- **에러 처리**: 200 응답 후에도 스트림 중 에러 발생 가능 → 스트림 에러 핸들러 필수

---

## 5. Rate Limits & 에러 처리

### 개념
- Token Bucket 알고리즘 기반 (고정 간격 리셋이 아닌 연속 보충)
- 두 가지 한도: **Spend Limits** (월 지출) + **Rate Limits** (요청 수/초)

### HTTP 에러 코드
| 코드 | 타입 | 설명 |
|-----|------|------|
| 429 | `rate_limit_error` | Rate limit 초과 |
| 529 | `overloaded_error` | API 과부하 (일시적) |
| 500 | `api_error` | 내부 오류 |

### 설정 방법 (Python 지수 백오프)

```python
import anthropic
import time

def call_with_retry(client, **kwargs, max_retries=5):
    for attempt in range(max_retries):
        try:
            return client.messages.create(**kwargs)
        except anthropic.RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = (2 ** attempt) + 0.5  # 지수 백오프 + 지터
            print(f"Rate limit hit. Waiting {wait_time}s...")
            time.sleep(wait_time)
        except anthropic.APIStatusError as e:
            if e.status_code == 529:  # Overloaded
                time.sleep(30)
            else:
                raise
```

```json
// Retry-After 헤더 확인
// 429 응답의 retry-after 헤더 값(초) 대기 후 재시도
```

### 권장 값
- **기본 백오프**: 2^n 초 (1, 2, 4, 8, 16초)
- **최대 재시도**: 5회
- **지터 추가**: ±0.5초 (동시 다중 요청 충돌 방지)
- **트래픽 램프업**: 급격한 증가 대신 점진적으로 (acceleration limits 방지)

### Usage Tier별 Rate Limits
| Tier | Credit 구매 | 월 지출 한도 |
|-----|------------|------------|
| Tier 1 | $5 | $100 |
| Tier 2 | $40 | $500 |
| Tier 3 | $200 | $1,000 |
| Tier 4 | $400 | $200,000 |

---

## 6. Context Window

### 개념
- 모델이 응답 생성 시 참조할 수 있는 전체 텍스트 (입력 + 출력 포함)
- 대화 히스토리가 쌓이면 선형적으로 증가
- "Context rot": 토큰이 많아질수록 정확도/회상률 저하

### 모델별 Context Window
| 모델 | Context Window | Max Output |
|-----|---------------|------------|
| Claude Opus 4.6 | **1M tokens** | 128k tokens |
| Claude Sonnet 4.6 | **1M tokens** | 64k tokens |
| Claude Haiku 4.5 | 200k tokens | 64k tokens |

### 효율적 사용법

**1. Server-side Compaction (주요 전략)**
```json
// 긴 대화 → 서버가 자동으로 이전 컨텍스트 압축
// /docs/en/build-with-claude/compaction 참조
```

**2. Context Editing**
- Tool result 삭제
- Thinking block 삭제
- 오래된 메시지 제거

**3. 컨텍스트 최적화 팁**
```python
# 토큰 카운트 미리 확인
response = client.messages.count_tokens(
    model="claude-opus-4-6",
    system="시스템 프롬프트",
    messages=messages
)
print(f"입력 토큰: {response.input_tokens}")
```

### 권장 값
- **실용 한도**: 전체 context window의 70% 이하 유지 (context rot 방지)
- **멀티턴 대화**: Prompt Caching + Compaction 병행 사용
- **에이전트 워크플로우**: 불필요한 tool result는 압축/제거

---

## 7. Model Selection

### 모델별 특성 (2026-03-25 기준)

| 모델 | 가격 (입력/출력) | 레이턴시 | Context | Max Output | 특징 |
|-----|----------------|---------|---------|-----------|------|
| **Claude Opus 4.6** | $5/$25 per MTok | 보통 | 1M | 128k | 가장 지능적, 에이전트/코딩 최적 |
| **Claude Sonnet 4.6** | $3/$15 per MTok | 빠름 | 1M | 64k | 속도+지능 균형 |
| **Claude Haiku 4.5** | $1/$5 per MTok | 가장 빠름 | 200k | 64k | 경량 작업, 비용 절감 |

### 선택 가이드

```
복잡한 추론/코딩/에이전트 → Opus 4.6 (Adaptive thinking)
일반 작업 + 빠른 응답 → Sonnet 4.6
단순 분류/요약/빠른 응답 → Haiku 4.5
비용 최우선 + 대량 처리 → Haiku 4.5 + Batch API
```

### 설정 방법
```json
{
  "model": "claude-opus-4-6",
  // 또는 alias 사용:
  // "model": "claude-sonnet-4-6"
  // "model": "claude-haiku-4-5"
}
```

### 권장 값
- **개발/테스트**: `claude-haiku-4-5` (빠르고 저렴)
- **프로덕션 일반**: `claude-sonnet-4-6`
- **복잡한 작업/에이전트**: `claude-opus-4-6`
- **스냅샷 고정**: 정확한 버전 ID 사용 (e.g., `claude-sonnet-4-5-20250929`)

---

## 8. Temperature / Top-p

### 개념
- **Temperature**: 출력의 무작위성 조절 (0=결정론적, 1=창의적)
- **Top-p**: 누적 확률 기준 토큰 샘플링 범위 (0-1)
- **Top-k**: 상위 k개 토큰만 고려

### 설정 방법

```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "temperature": 0.7,
  "top_p": 0.9,
  "messages": [{"role": "user", "content": "질문"}]
}
```

### 권장 값
| 용도 | temperature | top_p | 설명 |
|-----|------------|-------|------|
| 코드 생성 | 0.0 ~ 0.2 | 0.95 | 결정론적, 정확한 답 |
| 분류/추출 | 0.0 | 1.0 | 완전 결정론적 |
| 일반 대화 | 0.7 | 0.9 | 균형 잡힌 응답 |
| 창의적 글쓰기 | 0.9 ~ 1.0 | 0.95 | 다양하고 창의적 |
| 브레인스토밍 | 1.0 | 0.95 | 최대 다양성 |

> **주의**: `temperature`와 `top_p`를 동시에 조절하면 예측하기 어려울 수 있음  
> → 일반적으로 하나만 조절 권장

---

## 9. System Prompts

### 개념
- 모델의 역할, 지침, 컨텍스트를 정의하는 특별 프롬프트
- `messages` 배열 외부에 별도로 설정
- 모든 대화 턴에 적용되는 "기본 설정"

### 왜 필요한가
- 일관된 페르소나/행동 방식 정의
- 출력 형식 지정 (JSON, Markdown 등)
- 안전 가이드라인, 도메인 제한 설정
- **Prompt Caching과 결합 시 비용 절감 극대화**

### 설정 방법

**기본 설정:**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "system": "당신은 전문 소프트웨어 엔지니어입니다. 코드 리뷰 시 보안, 성능, 가독성 순으로 우선순위를 두세요.",
  "messages": [...]
}
```

**Prompt Caching + System Prompt (최적화):**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 1024,
  "system": [
    {
      "type": "text",
      "text": "매우 긴 시스템 프롬프트 (수천 토큰)...\n컨텍스트, 역할, 지침 등",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "messages": [...]
}
```

### 권장 값
- **명확한 역할 정의**: 모델이 무엇을 해야 하는지 명시
- **출력 형식 지정**: JSON 스키마, Markdown 구조 등 예시 포함
- **긴 시스템 프롬프트**: 반드시 Prompt Caching 적용 (1,024+ 토큰)
- **지침 우선순위**: 중요한 규칙을 앞에, 세부사항을 뒤에
- **부정문 최소화**: "하지 마세요" 대신 "~해주세요" 형식

---

## 10. Batch API (대량 처리)

### 개념
- 비동기 대량 요청 처리 (최대 256MB, 100,000 요청/배치)
- **50% 비용 절감** (실시간 API 대비)
- 24시간 내 처리 완료 보장

### 설정 방법

```json
// POST /v1/messages/batches
{
  "requests": [
    {
      "custom_id": "request-1",
      "params": {
        "model": "claude-haiku-4-5",
        "max_tokens": 1024,
        "messages": [{"role": "user", "content": "배치 작업 1"}]
      }
    },
    {
      "custom_id": "request-2",
      "params": {
        "model": "claude-haiku-4-5",
        "max_tokens": 1024,
        "messages": [{"role": "user", "content": "배치 작업 2"}]
      }
    }
  ]
}
```

### 권장 값
- **사용 시기**: 실시간 응답 불필요, 대량 데이터 처리, 분석 파이프라인
- **모델**: `claude-haiku-4-5` (저렴 + 빠름)
- **배치 크기**: 가능한 크게 묶어서 요청 (API 오버헤드 최소화)

---

## 종합 최적화 체크리스트

```
□ 반복 요청 → Prompt Caching 적용 (비용 절감 + 속도 향상)
□ 복잡한 추론 → Adaptive Thinking on Opus/Sonnet 4.6
□ 긴 응답 → Streaming 활성화
□ 429 에러 → 지수 백오프 + 지터 구현
□ 긴 대화 → Compaction 전략 사용
□ 비용 최적화 → 작업별 적합한 모델 선택
□ 창의적 작업 → temperature 높게 (0.8-1.0)
□ 코드/분석 → temperature 낮게 (0.0-0.3)
□ 시스템 프롬프트 길면 → cache_control 추가
□ 대량 처리 → Batch API (50% 할인)
```

---

*참고: [Anthropic 공식 문서](https://platform.claude.com/docs) | 최신 모델 정보는 [Models Overview](https://platform.claude.com/docs/en/about-claude/models/overview) 확인*
