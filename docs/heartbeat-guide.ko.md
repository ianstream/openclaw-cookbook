# Heartbeat 가이드

> 주기적 에이전트 체크인 및 백그라운드 작업

[English](heartbeat-guide.md) | [한국어](heartbeat-guide.ko.md) | [← Cookbook으로](../README.ko.md)

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
      "heartbeat": {
        "every": "1h"
      }
    }
  }
}
```

## ⚙️ 옵션

| 옵션 | 설명 |
|------|------|
| `every` | 주기 (`"30m"`, `"1h"`, `"2h"`) |
| `target` | 전송 대상 (`"last"`, `"none"`, `"discord"`) |
| `lightContext` | HEARTBEAT.md만 로드 (토큰 절약) |
| `activeHours` | 특정 시간대에만 실행 |
| `model` | 저렴한 모델 사용 |

## 📝 예제 설정

### 활성 시간 설정

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",
        "target": "last",
        "lightContext": true,
        "activeHours": {
          "start": "09:00",
          "end": "23:00",
          "timezone": "Asia/Seoul"
        }
      }
    }
  }
}
```

### 비용 최적화 (저렴한 모델)

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "1h",
        "model": "openai/gpt-4.1-mini",
        "lightContext": true
      }
    }
  }
}
```

### 특정 채널로 전송

```json
{
  "agents": {
    "list": [
      {
        "id": "ops",
        "heartbeat": {
          "every": "1h",
          "target": "discord",
          "to": "channel:123456789012345678",
          "prompt": "HEARTBEAT.md 확인. 긴급한 것만 보고. 없으면 HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

## 📄 HEARTBEAT.md 예제

`~/.openclaw/workspace/HEARTBEAT.md` 생성:

```markdown
# Heartbeat 체크리스트

## 매번 체크
- [ ] 긴급 이메일 확인
- [ ] 다가오는 일정 확인

## 하루 한 번
- [ ] 어제 활동 요약
- [ ] 메모리 파일 업데이트
```

## 💡 팁

1. **토큰 절약**: `lightContext: true`로 HEARTBEAT.md만 로드
2. **비용 절약**: 저렴한 모델 사용
3. **야간 방지**: `activeHours`로 밤 알림 차단
4. **응답**: 할 일 없으면 `HEARTBEAT_OK` 응답

## 📚 참고 문서

- [OpenClaw Heartbeat 문서](https://docs.openclaw.ai/concepts/heartbeat)
