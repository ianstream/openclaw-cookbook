# 보안 가이드

> OpenClaw 배포를 무단 접근으로부터 보호

[English](security-guide.md) | [한국어](security-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## ⚠️ 중요: 개인 어시스턴트 신뢰 모델

OpenClaw은 **개인 어시스턴트** (단일 사용자/단일 신뢰 경계)로 설계됨.

- ✅ 게이트웨이당 한 명의 사용자
- ❌ 다중 테넌트 보안 경계가 아님
- 여러 신뢰할 수 없는 사용자가 필요하면 → **별도 게이트웨이** 실행

---

## 🔒 빠른 보안 감사

```bash
# 보안 감사 실행
openclaw security audit

# 심층 감사 (라이브 프로브 포함)
openclaw security audit --deep

# 일부 문제 자동 수정
openclaw security audit --fix
```

---

## 🎯 보안 체크리스트

### 1. 게이트웨이 인증 (필수!)

**항상 게이트웨이 인증 토큰 설정:**

```json
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "YOUR_LONG_RANDOM_TOKEN"
    }
  }
}
```

토큰 생성:
```bash
openclaw doctor --generate-gateway-token
```

### 2. 네트워크 바인딩

| 모드 | 노출 | 용도 |
|------|------|------|
| `"loopback"` | 로컬만 | **기본값, 가장 안전** |
| `"lan"` | 로컬 네트워크 | 방화벽 필요 |
| `"tailnet"` | Tailscale만 | 원격 접속 권장 |

```json
{
  "gateway": {
    "bind": "loopback"
  }
}
```

⚠️ **인증 없이 절대 노출하지 마세요!**

### 3. DM 접근 제어

```json
{
  "channels": {
    "discord": {
      "dmPolicy": "pairing",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

| 정책 | 설명 |
|------|------|
| `"pairing"` | 승인 코드 필요 (권장) |
| `"allowlist"` | 지정된 사용자만 |
| `"open"` | ⚠️ 누구나 DM 가능 (위험) |
| `"disabled"` | DM 비활성화 |

### 4. 그룹 채팅 보호

```json
{
  "channels": {
    "discord": {
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": true,
          "channels": {
            "*": { "allow": true }
          }
        }
      }
    }
  }
}
```

- 공개 그룹에서는 항상 `requireMention: true` 사용
- 필요하지 않으면 `groupPolicy: "open"` 피하기

### 5. 도구 제한

신뢰할 수 없는 컨텍스트에서 위험한 도구 차단:

```json
{
  "tools": {
    "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
  }
}
```

### 6. Exec 보안

```json
{
  "tools": {
    "exec": {
      "security": "allowlist"
    },
    "elevated": {
      "enabled": false
    }
  }
}
```

| 보안 모드 | 설명 |
|----------|------|
| `"full"` | 모든 명령어 허용 |
| `"allowlist"` | 화이트리스트 명령어만 |
| `"deny"` | exec 불허 |

### 7. 파일 권한

```bash
# 올바른 권한 설정
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json

# doctor로 확인
openclaw doctor
```

---

## 🛡️ 강화된 기본 설정

시작점으로 복사하세요:

```json
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "긴_랜덤_토큰으로_교체"
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "messaging",
    "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"],
    "fs": { "workspaceOnly": true },
    "exec": { "security": "deny", "ask": "always" },
    "elevated": { "enabled": false }
  },
  "channels": {
    "discord": {
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

---

## 🔐 민감한 파일 위치

| 파일 | 내용 |
|------|------|
| `~/.openclaw/openclaw.json` | 설정, 토큰 포함 가능 |
| `~/.openclaw/credentials/` | 채널 자격 증명 |
| `~/.openclaw/agents/*/auth-profiles.json` | API 키, OAuth 토큰 |
| `~/.openclaw/agents/*/sessions/` | 대화 기록 |

**이것들은 비공개로 유지하세요!** 절대 git에 커밋하지 마세요.

---

## 🚫 하지 말아야 할 것

❌ 인증 없이 게이트웨이 노출  
❌ `dmPolicy: "open"` + 도구 활성화  
❌ 공개 서버에서 `groupPolicy: "open"`  
❌ allowFrom 없이 `elevated.enabled: true`  
❌ 설정에 API 키 평문 저장  
❌ `~/.openclaw` 디렉토리 공개 공유  

---

## 🆘 문제 발생 시

### 1. 즉시 중지

```bash
openclaw gateway stop
```

### 2. 노출 차단

```json
{
  "gateway": { "bind": "loopback" },
  "channels": { "discord": { "dmPolicy": "disabled" } }
}
```

### 3. 자격 증명 교체

- 게이트웨이 토큰
- API 키
- Discord 봇 토큰
- 노출된 모든 시크릿

### 4. 감사

```bash
# 로그 확인
tail -f ~/.openclaw/logs/openclaw.log

# 보안 감사 실행
openclaw security audit --deep
```

---

## 📚 참고 문서

- [OpenClaw 보안 문서](https://docs.openclaw.ai/gateway/security)
- [샌드박싱 가이드](https://docs.openclaw.ai/gateway/sandboxing)
- [Exec 승인](https://docs.openclaw.ai/tools/exec-approvals)
