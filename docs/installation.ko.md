# OpenClaw 설치 가이드

> OpenClaw 완전 설치 안내

[English](installation.md) | [한국어](installation.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📋 사전 요구사항

- **Node.js**: v20.0.0 이상
- **npm**: v10.0.0 이상 (Node.js에 포함)
- **운영체제**: macOS, Linux, Windows (WSL2)

### 버전 확인

```bash
node --version   # v20.0.0+
npm --version    # v10.0.0+
```

## 🚀 빠른 설치

### 1. OpenClaw 설치

```bash
npm install -g openclaw
```

### 2. 초기 설정

```bash
openclaw setup
```

이 명령어가 수행하는 작업:
- `~/.openclaw/` 디렉토리 생성
- 기본 `openclaw.json` 설정 파일 생성
- 인증 안내

### 3. Anthropic 인증

```bash
# 옵션 A: Claude Max 구독 (OAuth)
openclaw auth login anthropic

# 옵션 B: API Key
openclaw auth add anthropic --api-key YOUR_API_KEY
```

### 4. 게이트웨이 시작

```bash
openclaw gateway start
```

## 📱 Discord 봇 설정

### 1. Discord 애플리케이션 생성

1. [Discord Developer Portal](https://discord.com/developers/applications) 접속
2. "New Application" 클릭
3. 봇 이름 입력 (예: "OpenClaw Bot")

### 2. 봇 토큰 받기

1. "Bot" 섹션으로 이동
2. "Reset Token" 클릭
3. 토큰 복사

### 3. 인텐트 활성화

"Bot" 섹션에서 활성화:
- ✅ Presence Intent
- ✅ Server Members Intent
- ✅ Message Content Intent

### 4. 서버에 봇 초대

1. "OAuth2" → "URL Generator" 이동
2. 스코프 선택: `bot`, `applications.commands`
3. 권한 선택: `Send Messages`, `Read Message History`, `Add Reactions`
4. 생성된 URL 복사 후 열기
5. 서버 선택 후 승인

### 5. OpenClaw 설정

```bash
# 설정 파일 편집
nano ~/.openclaw/openclaw.json
```

Discord 설정 추가:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "groupPolicy": "allowlist",
      "allowFrom": ["YOUR_DISCORD_USER_ID"],
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": false,
          "channels": {
            "*": { "allow": true }
          }
        }
      }
    }
  }
}
```

### 6. ID 확인 방법

```bash
# Discord에서 개발자 모드 활성화:
# 사용자 설정 → 앱 설정 → 고급 → 개발자 모드

# 서버 우클릭 → 서버 ID 복사 (Guild ID)
# 자신 우클릭 → 사용자 ID 복사
```

### 7. 게이트웨이 재시작

```bash
openclaw gateway restart
```

## 🔧 주요 명령어

| 명령어 | 설명 |
|--------|------|
| `openclaw status` | 게이트웨이 상태 확인 |
| `openclaw gateway start` | 게이트웨이 시작 |
| `openclaw gateway stop` | 게이트웨이 중지 |
| `openclaw gateway restart` | 게이트웨이 재시작 |
| `openclaw auth list` | 인증 프로필 목록 |
| `openclaw doctor` | 문제 진단 |

## 🐛 문제 해결

### 게이트웨이가 시작되지 않음

```bash
# 로그 확인
tail -f ~/.openclaw/logs/openclaw.log

# 진단 실행
openclaw doctor
```

### Discord 봇이 응답하지 않음

1. Discord에서 봇이 온라인인지 확인
2. 토큰이 올바른지 확인
3. `allowFrom`에 자신의 사용자 ID 포함 확인
4. `requireMention` 설정 확인

### 인증 오류

```bash
# 재인증
openclaw auth login anthropic

# 또는 인증 상태 확인
openclaw auth list
```

## 📚 다음 단계

설치 후:

1. [토큰 최적화](token-optimization.ko.md) - 비용 절감
2. [다중 에이전트 설정](multi-agent-guide.ko.md) - 여러 페르소나
3. [채널 바인딩](channel-binding-guide.ko.md) - 채널별 설정

## 📚 참고 문서

- [OpenClaw 공식 문서](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
