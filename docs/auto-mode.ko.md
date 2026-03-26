# Claude Code Auto Mode 가이드

> 모델 기반 분류기를 사용해 권한 프롬프트를 안전하게 건너뛰는 방법

**출처**: [Anthropic Engineering Blog](https://www.anthropic.com/engineering/claude-code-auto-mode)

## 개요

Auto mode는 수동 승인 대신 Sonnet 4.6 기반 분류기에 권한 결정을 위임합니다. 위험한 작업은 차단하면서 안전한 작업은 프롬프트 없이 실행합니다.

### 문제점

| 모드 | 안전성 | 마찰 |
|------|--------|------|
| 수동 승인 | ✅ 안전 | ❌ 93%는 그냥 승인함 |
| `--dangerously-skip-permissions` | ❌ 위험 | ✅ 마찰 없음 |
| 샌드박스 | ✅ 안전 | ❌ 유지보수 높음 |
| **Auto mode** | ✅ 안전 | ✅ 마찰 낮음 |

## 작동 원리

### 2중 방어

1. **입력 레이어**: 도구 출력을 컨텍스트에 넣기 전 프롬프트 인젝션 탐지
2. **출력 레이어**: Transcript 분류기 (Sonnet 4.6)가 각 작업 평가
   - Stage 1: 빠른 yes/no 필터 (저렴)
   - Stage 2: Chain-of-thought 추론 (Stage 1에서 플래그된 경우만)

### 3단계 권한 체계

| Tier | 작업 | 분류기 호출? |
|------|------|-------------|
| 1 | 안전 도구 allowlist (읽기, 검색) | ❌ 자동 허용 |
| 2 | 프로젝트 내 파일 수정 | ❌ 자동 허용 |
| 3 | 쉘, 외부 도구, 프로젝트 외부 | ✅ 분류 |

## 위협 모델

Auto mode가 방어하는 대상:

- **과잉 행동(Overeager)**: 에이전트가 권한 범위 초과
- **실수**: 영향 범위 오해
- **프롬프트 인젝션**: 콘텐츠에 악성 지시 삽입
- **미정렬(Misalignment)**: (드묾) 에이전트가 자체 목표 추구

### 실제 사고 사례 (차단됨)

| 사고 | 차단 이유 |
|------|----------|
| "브랜치 정리해줘" → 원격 일괄 삭제 | 모호한 요청 ≠ 파괴적 권한 |
| 인증 오류 → 다른 API 토큰 탐색 | 자격증명 탐색 |
| "내 작업 취소해줘" → 유사 이름 삭제 | 에이전트가 추론한 대상 |
| 디버그 스크립트 → GitHub Gist | 데이터 유출 |
| 배포 실패 → `--skip-verification`으로 재시도 | 안전 우회 |

## 빠른 시작

```bash
# Auto mode로 Claude Code 실행
claude --permission-mode auto

# 또는 세션 중 전환
/permission-mode auto
```

## 설정

### 현재 설정 확인

```bash
# 현재 적용된 설정 (사용자 설정 + 기본값)
claude auto-mode config

# 기본 규칙만 확인
claude auto-mode defaults

# 커스텀 규칙에 AI 피드백 받기
claude auto-mode critique
```

### 환경 커스터마이징

`~/.claude/settings.json`에 추가:

```json
{
  "autoMode": {
    "environment": [
      "**Source control orgs**: github.com/your-org, github.com/your-username",
      "**Trusted internal domains**: internal.company.com, api.company.com",
      "**Trusted cloud buckets**: s3://your-bucket, gs://your-bucket"
    ]
  }
}
```

### 커스텀 허용 규칙

```json
{
  "autoMode": {
    "allow": [
      "Custom Rule: 허용할 작업과 이유 설명"
    ]
  }
}
```

### 커스텀 차단 규칙

```json
{
  "autoMode": {
    "soft_deny": [
      "Custom Block: 차단할 작업 설명"
    ]
  }
}
```

## 기본 규칙 요약

### 자동 허용 (ALLOW)

- 테스트 아티팩트 (placeholder 자격증명)
- 프로젝트 범위 내 로컬 작업
- 읽기 전용 작업 (GET, 쿼리)
- 선언된 의존성 (`npm install`, `pip install -r`)
- 툴체인 부트스트랩 (rustup, pyenv, brew)
- 표준 자격증명 흐름
- 작업 브랜치로 git push (기본 브랜치 제외)

### 자동 차단 (DENY)

| 카테고리 | 예시 |
|----------|------|
| **파괴적 Git** | Force push, 원격 브랜치 삭제 |
| **외부 코드** | `curl \| bash`, 신뢰할 수 없는 레포 |
| **클라우드 대량 삭제** | S3/GCS 일괄 작업 |
| **프로덕션** | 배포, 마이그레이션 |
| **보안 약화** | TLS, 로깅, 방화벽 비활성화 |
| **자격증명 남용** | 탐색, 유출 |
| **데이터 유출** | 외부 POST, public gist |
| **지속성** | SSH 키, cron, systemd |
| **자기 수정** | 자신의 설정/권한 편집 |

## 성능

| 지표 | 실제 트래픽 (n=10K) | 과잉 행동 (n=52) | 합성 유출 (n=1K) |
|------|---------------------|------------------|------------------|
| **FPR** (오탐) | 0.4% | - | - |
| **FNR** (미탐) | - | 17% | 5.7% |

> 과잉 행동에 대한 17% FNR이 솔직한 수치입니다. 에이전트가 진심으로 도우려 했지만 권한을 초과한 경우입니다.

## 모범 사례

1. **기본값으로 시작** — 보수적이고 실전 검증됨
2. **신뢰 소스 점진적 추가** — 필요한 것만 추가
3. **`claude auto-mode critique` 사용** — 커스텀 규칙에 AI 피드백
4. **차단된 작업 검토** — 분류기가 잡는 것 학습
5. **git과 결합** — 프로젝트 내 편집은 버전 관리로 검토 가능

## 제한사항

- 과잉 행동의 17%는 통과할 수 있음
- Tier 3 작업에 분류기 지연 추가
- 커스텀 규칙은 신중한 문구 필요
- 일부 정당한 워크플로우에 예외 필요할 수 있음

## 참고

- [Permission Modes 문서](https://code.claude.com/docs/en/permission-modes)
- [Anthropic 블로그 포스트](https://www.anthropic.com/engineering/claude-code-auto-mode)
- [Claude Opus 4.6 System Card §6.2](https://www-cdn.anthropic.com/14e4fb01875d2a69f646fa5e574dea2b1c0ff7b5.pdf)
