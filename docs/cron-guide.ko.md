# Cron Job 가이드

> 특정 시간에 작업 예약 실행

[English](cron-guide.md) | [한국어](cron-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 기본 설정

```json
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 2
  }
}
```

## 📝 CLI 예제

### 매일 아침 브리핑

```bash
openclaw cron add \
  --name "Morning Brief" \
  --cron "0 7 * * *" \
  --tz "Asia/Seoul" \
  --session isolated \
  --message "오늘 일정과 인박스 요약해줘." \
  --announce \
  --channel discord \
  --to "channel:YOUR_CHANNEL_ID"
```

### 일회성 리마인더

```bash
openclaw cron add \
  --name "Reminder" \
  --at "20m" \
  --session main \
  --system-event "리마인더: 10분 후 미팅" \
  --wake now \
  --delete-after-run
```

### 주간 리포트

```bash
openclaw cron add \
  --name "Weekly Report" \
  --cron "0 9 * * 1" \
  --tz "Asia/Seoul" \
  --session isolated \
  --message "주간 활동 리포트 생성해줘." \
  --announce
```

## ⚙️ 스케줄 타입

| 타입 | 예시 |
|------|------|
| `--cron` | `"0 7 * * *"` (매일 오전 7시) |
| `--at` | `"20m"` (20분 후) |
| `--every` | `"2h"` (2시간마다) |

## 🔧 옵션

| 옵션 | 설명 |
|------|------|
| `--session isolated` | 별도 세션에서 실행 (권장) |
| `--session main` | 메인 세션에서 실행 |
| `--announce` | 채널로 결과 전송 |
| `--delete-after-run` | 일회성 작업 |
| `--model` | 특정 모델 사용 |
| `--lightContext` | workspace 파일 스킵 |

## 📝 JSON 형식 (Tool Call)

```json
{
  "name": "Morning Brief",
  "schedule": {
    "kind": "cron",
    "expr": "0 7 * * *",
    "tz": "Asia/Seoul"
  },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "오늘 일정 요약해줘.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:123456789012345678"
  }
}
```

## 💡 팁

1. **Isolated 세션 사용**: 메인 세션 컨텍스트 오염 방지
2. **비용 절감**: `--model openai/gpt-4.1-mini` 루틴 작업에 사용
3. **가벼운 컨텍스트**: `--lightContext`로 workspace 로딩 스킵
4. **정확한 시간**: `--exact`로 정각 stagger 비활성화

## 🛠️ 관리 명령어

```bash
openclaw cron list              # 모든 job 목록
openclaw cron runs <jobId>      # 실행 이력 조회
openclaw cron remove <jobId>    # job 삭제
openclaw cron run <jobId>       # 즉시 실행
```

## 📚 참고 문서

- [OpenClaw Cron 문서](https://docs.openclaw.ai/concepts/cron)
