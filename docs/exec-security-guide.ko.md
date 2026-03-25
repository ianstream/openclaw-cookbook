# Exec 보안 가이드

> 명령어 실행 권한 및 샌드박싱 제어

[English](exec-security-guide.md) | [한국어](exec-security-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 기본 설정

```json
{
  "tools": {
    "exec": {
      "security": "full"
    }
  }
}
```

## ⚙️ 보안 모드

| 모드 | 설명 |
|------|------|
| `"full"` | 모든 명령어 허용 |
| `"allowlist"` | 화이트리스트 명령어만 |
| `"deny"` | exec 비허용 |

## 🔒 Elevated 명령어

특정 사용자만 호스트 명령 허용:

```json
{
  "tools": {
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "discord": ["user:YOUR_USER_ID"],
        "telegram": ["+821012345678"]
      }
    }
  }
}
```

## 🐳 Docker 샌드박싱

격리된 컨테이너에서 명령 실행:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "agent",
        "workspaceAccess": "rw",
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "network": "none",
          "memory": "1g",
          "cpus": 1
        }
      }
    }
  }
}
```

### 샌드박스 모드

| 모드 | 설명 |
|------|------|
| `"off"` | 샌드박싱 없음 |
| `"non-main"` | 서브에이전트만 샌드박스 |
| `"all"` | 모든 실행 격리 |

## 📝 예제 설정

### 프로덕션 (엄격)

```json
{
  "tools": {
    "exec": {
      "security": "allowlist",
      "backgroundMs": 10000,
      "timeoutSec": 300
    },
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "discord": ["user:YOUR_ADMIN_ID"]
      }
    }
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "docker": {
          "network": "none"
        }
      }
    }
  }
}
```

### 개발 (느슨)

```json
{
  "tools": {
    "exec": {
      "security": "full",
      "timeoutSec": 1800
    }
  }
}
```

## 💡 팁

1. **프로덕션**: 안전을 위해 `security: "allowlist"` 사용
2. **네트워크 격리**: 샌드박스에서 `network: "none"` 설정
3. **메모리 제한**: `memory: "1g"`로 리소스 남용 방지
4. **Elevated 사용자**: 호스트 명령은 신뢰할 수 있는 사용자만

## 📚 참고 문서

- [OpenClaw Exec 문서](https://docs.openclaw.ai/concepts/exec)
- [OpenClaw Sandbox 문서](https://docs.openclaw.ai/concepts/sandbox)
