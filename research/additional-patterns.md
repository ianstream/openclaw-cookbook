# OpenClaw 추가 설정 패턴 연구

> 출처: `https://docs.openclaw.ai` + `/opt/homebrew/lib/node_modules/openclaw/docs/`  
> 조사일: 2026-03-25

현재 Cookbook에 없는 유용한 설정 패턴 목록.

---

## TTS (Text-to-Speech) 설정

### 용도
- 에이전트 응답을 음성으로 자동 변환
- ElevenLabs 또는 OpenAI TTS 사용
- `/tts off|always|inbound|tagged` 로 세션별 제어 가능

### 설정 예제
```json5
{
  "messages": {
    "tts": {
      "auto": "always",
      "mode": "final",
      "provider": "elevenlabs",
      "summaryModel": "openai/gpt-4.1-mini",
      "maxTextLength": 4000,
      "elevenlabs": {
        "apiKey": "${ELEVENLABS_API_KEY}",
        "voiceId": "voice_id",
        "modelId": "eleven_multilingual_v2",
        "voiceSettings": {
          "stability": 0.5,
          "similarityBoost": 0.75,
          "speed": 1.0
        }
      },
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "model": "gpt-4o-mini-tts",
        "voice": "alloy"
      }
    }
  }
}
```

### 권장 사항
- `auto: "always"` — 모든 응답 자동 음성 변환
- `auto: "inbound"` — 음성 메시지 받을 때만
- `auto: "tagged"` — `/tts` 명령어로 태그한 것만
- Discord 보이스 채널에서도 TTS 연동 가능 (`channels.discord.voice.tts`)

---

## Talk Mode (Voice Call) 설정

### 용도
- macOS/iOS/Android 앱에서 실시간 음성 대화 모드
- Talk 버튼으로 음성 입력 → TTS 응답

### 설정 예제
```json5
{
  "talk": {
    "voiceId": "EXAVITQu4vr4xnSDxMaL",
    "voiceAliases": {
      "Clawd": "EXAVITQu4vr4xnSDxMaL",
      "Roger": "CwhRBWXzGAHq8TQ4Fs17"
    },
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "${ELEVENLABS_API_KEY}",
    "silenceTimeoutMs": 1500,
    "interruptOnSpeech": true
  }
}
```

### 권장 사항
- `silenceTimeoutMs` — 말이 끊긴 후 몇 ms 후 전송할지 (macOS: 700ms 기본)
- `interruptOnSpeech: true` — 에이전트 말하는 중에도 끼어들기 가능
- ElevenLabs API 키는 env 변수 사용 권장

---

## Heartbeat 설정

### 용도
- 주기적으로 에이전트 자동 실행 (배경 작업, 알림, 체크인)
- HEARTBEAT.md 파일로 할 일 목록 관리
- 특정 채널로 알림 전달 가능

### 설정 예제
```json5
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
        },
        "model": "openai/gpt-4.1-mini",
        "ackMaxChars": 300
      }
    }
  }
}
```

### 활성 시간 + 채널 지정 예제
```json5
{
  "agents": {
    "list": [
      {
        "id": "ops",
        "heartbeat": {
          "every": "1h",
          "target": "discord",
          "to": "channel:123456789012345678",
          "prompt": "HEARTBEAT.md를 읽고 긴급한 사항이 있으면 알려줘. 없으면 HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

### 권장 사항
- `lightContext: true` — HEARTBEAT.md만 사용, 토큰 절약
- `target: "none"` — 외부 전달 없이 내부 상태만 업데이트
- `target: "last"` — 마지막 사용 채널로 전달
- Heartbeat 비용 줄이려면 더 저렴한 모델 지정

---

## Cron Job 설정

### 용도
- 정확한 시간에 예약 실행 (아침 브리핑, 리마인더, 정기 보고서)
- 독립 세션(isolated) 또는 메인 세션으로 실행
- 특정 채널에 결과 자동 전달

### 설정 예제
```json5
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 2,
    "sessionRetention": "24h",
    "runLog": {
      "maxBytes": "2mb",
      "keepLines": 2000
    }
  }
}
```

### CLI로 Job 추가 (실전 예제)
```bash
# 매일 아침 7시 Discord에 브리핑
openclaw cron add \
  --name "Morning Brief" \
  --cron "0 7 * * *" \
  --tz "Asia/Seoul" \
  --session isolated \
  --message "오늘 할 일 요약해줘. 캘린더 + inbox 확인." \
  --announce \
  --channel discord \
  --to "channel:1467422131355910197"

# 20분 후 리마인더
openclaw cron add \
  --name "Reminder" \
  --at "20m" \
  --session main \
  --system-event "리마인더: 미팅 준비해." \
  --wake now \
  --delete-after-run
```

### JSON 형식 (도구 호출 시)
```json
{
  "name": "Morning Brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "Asia/Seoul" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "오늘 할 일 요약해줘.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:1467422131355910197"
  }
}
```

### 권장 사항
- `sessionTarget: "isolated"` — 메인 세션 오염 방지
- `lightContext: true` — 불필요한 workspace 파일 로드 방지
- `--exact` 옵션 — top-of-hour 자동 stagger 비활성화
- 비용 줄이려면 `--model openai/gpt-4.1-mini` 지정

---

## Audio/STT (Speech-to-Text) 설정

### 용도
- 음성 메시지 자동 텍스트 변환
- OpenAI Whisper 또는 로컬 CLI 지원
- 채널별로 허용/차단 범위 설정

### 설정 예제
```json5
{
  "tools": {
    "media": {
      "concurrency": 2,
      "audio": {
        "enabled": true,
        "maxBytes": 20971520,
        "scope": {
          "default": "deny",
          "rules": [
            { "action": "allow", "match": { "chatType": "direct" } }
          ]
        },
        "models": [
          { "provider": "openai", "model": "gpt-4o-mini-transcribe" }
        ]
      },
      "video": {
        "enabled": true,
        "maxBytes": 52428800,
        "models": [
          { "provider": "google", "model": "gemini-3-flash-preview" }
        ]
      }
    }
  }
}
```

### 로컬 Whisper CLI 사용
```json5
{
  "tools": {
    "media": {
      "audio": {
        "enabled": true,
        "models": [
          {
            "type": "cli",
            "command": "whisper",
            "args": ["--model", "base", "{{MediaPath}}"]
          }
        ]
      }
    }
  }
}
```

### 권장 사항
- DM에서만 음성 허용하려면 `scope.rules` 로 제한
- 비용 절감: `gpt-4o-mini-transcribe` 사용
- 오프라인: `whisper-cpp` CLI 연동 (brew install whisper-cpp)

---

## Subagent 설정

### 용도
- 백그라운드에서 병렬 작업 실행
- 오케스트레이터 → 워커 패턴 구성
- 완료 시 부모 세션에 결과 자동 보고

### 설정 예제
```json5
{
  "agents": {
    "defaults": {
      "subagents": {
        "model": "openai/gpt-4.1-mini",
        "maxConcurrent": 4,
        "runTimeoutSeconds": 900,
        "archiveAfterMinutes": 60,
        "maxSpawnDepth": 2,
        "maxChildrenPerAgent": 5
      }
    }
  }
}
```

### 오케스트레이터 패턴 활성화
```json5
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxSpawnDepth": 2,
        "maxChildrenPerAgent": 5,
        "maxConcurrent": 8
      }
    }
  },
  "tools": {
    "subagents": {
      "tools": {
        "deny": ["gateway", "cron"]
      }
    }
  }
}
```

### 권장 사항
- `model` — 서브에이전트에 저렴한 모델 지정해 비용 절감
- `maxSpawnDepth: 2` — main → orchestrator → worker 3단계
- `runTimeoutSeconds: 900` — 15분 타임아웃 권장
- `cleanup: "delete"` — 완료 후 즉시 세션 정리

---

## Browser 설정

### 용도
- 에이전트 전용 브라우저 프로필 관리
- 개인 Chrome 세션과 완전 분리
- 원격 CDP (Browserless, Browserbase) 연동 가능

### 설정 예제
```json5
{
  "browser": {
    "enabled": true,
    "defaultProfile": "openclaw",
    "evaluateEnabled": true,
    "ssrfPolicy": {
      "dangerouslyAllowPrivateNetwork": true
    },
    "profiles": {
      "openclaw": { "cdpPort": 18800, "color": "#FF4500" },
      "work": { "cdpPort": 18801, "color": "#0066CC" }
    }
  }
}
```

### 보안 강화 (외부 접근 차단)
```json5
{
  "browser": {
    "ssrfPolicy": {
      "dangerouslyAllowPrivateNetwork": false,
      "hostnameAllowlist": ["*.example.com"],
      "allowedHostnames": ["localhost"]
    },
    "evaluateEnabled": false
  }
}
```

### 원격 브라우저 (Browserless)
```json5
{
  "browser": {
    "enabled": true,
    "defaultProfile": "browserless",
    "profiles": {
      "browserless": {
        "cdpUrl": "https://production-sfo.browserless.io?token=${BROWSERLESS_API_KEY}",
        "color": "#00AA00"
      }
    }
  }
}
```

### 권장 사항
- 기본: `profile="openclaw"` (격리된 에이전트 전용 브라우저)
- 로그인 필요: `profile="user"` (실제 Chrome 세션 연동)
- `evaluateEnabled: false` — JS 인젝션 방지 (보안 강화)
- `executablePath` — Brave/Edge 등 다른 브라우저 지정 가능

---

## Exec 보안 설정

### 용도
- exec 도구 실행 권한 세밀 제어
- gateway/node 호스트 실행 승인 관리
- safeBins으로 안전한 명령어 화이트리스트

### 설정 예제
```json5
{
  "tools": {
    "exec": {
      "backgroundMs": 10000,
      "timeoutSec": 1800,
      "notifyOnExit": true,
      "pathPrepend": ["~/bin", "/opt/oss/bin"]
    },
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "discord": ["user:987654321098765432"],
        "whatsapp": ["+821012345678"]
      }
    }
  }
}
```

### 샌드박싱 활성화 (Docker)
```json5
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

### 권장 사항
- `mode: "non-main"` — 서브에이전트만 샌드박스 적용
- `mode: "all"` — 모든 실행 격리 (최고 보안)
- `elevated.enabled: true` + `allowFrom` — 특정 사용자만 호스트 명령 허용
- Docker sandbox는 `network: "none"` 기본 (네트워크 차단)

---

## Plugin 설정

### 용도
- 채널 플러그인 (Mattermost, Voice Call 등) 관리
- 플러그인별 설정 및 활성화/비활성화
- 보안 제어 (promptInjection 차단)

### 설정 예제
```json5
{
  "plugins": {
    "enabled": true,
    "allow": ["voice-call", "mattermost"],
    "deny": [],
    "entries": {
      "voice-call": {
        "enabled": true,
        "hooks": {
          "allowPromptInjection": false
        },
        "config": { "provider": "twilio" }
      }
    }
  }
}
```

### Skills 설정
```json5
{
  "skills": {
    "allowBundled": ["gemini", "peekaboo"],
    "entries": {
      "nano-banana-pro": {
        "apiKey": { "source": "env", "provider": "default", "id": "GEMINI_API_KEY" }
      },
      "sag": { "enabled": false }
    }
  }
}
```

### 권장 사항
- `allowPromptInjection: false` — 플러그인이 프롬프트 조작 못하게 차단
- `allow` 화이트리스트로 필요한 플러그인만 로드
- `plugins.slots.memory` — 커스텀 메모리 플러그인 연결

---

## mDNS Discovery 설정

### 용도
- 로컬 네트워크에서 OpenClaw 게이트웨이 자동 검색
- 모바일 앱과 자동 페어링 지원
- 와이드 에어리어 DNS-SD (Tailscale 연동)

### 설정 예제
```json5
{
  "discovery": {
    "mdns": {
      "mode": "minimal"
    }
  }
}
```

### 와이드 에어리어 DNS-SD (크로스 네트워크)
```json5
{
  "discovery": {
    "mdns": {
      "mode": "full"
    },
    "wideArea": {
      "enabled": true
    }
  }
}
```

### 권장 사항
- `minimal` (기본) — cliPath, sshPort 노출 안 함 (보안)
- `full` — 모든 정보 노출 (내부 네트워크에서만)
- `OPENCLAW_MDNS_HOSTNAME` env로 호스트명 커스터마이징
- 와이드 에어리어: CoreDNS + Tailscale split DNS 조합

---

## 채널별 모델 오버라이드

### 용도
- 특정 채널/그룹에 다른 모델 할당
- 비용 절감 (일반 채널은 저렴한 모델, 중요 채널은 고성능 모델)

### 설정 예제
```json5
{
  "channels": {
    "modelByChannel": {
      "discord": {
        "123456789012345678": "anthropic/claude-opus-4-8"
      },
      "telegram": {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6"
      }
    }
  }
}
```

### 권장 사항
- 중요 채널(개인 DM): 강력한 모델
- 그룹 채팅: 저렴한 모델로 비용 절감
- Telegram 토픽별 모델 지정 가능

---

## 인바운드 디바운스 (메시지 배치)

### 용도
- 빠르게 연속 입력되는 메시지를 묶어서 처리
- 불필요한 API 호출 감소

### 설정 예제
```json5
{
  "messages": {
    "inbound": {
      "debounceMs": 2000,
      "byChannel": {
        "whatsapp": 5000,
        "slack": 1500,
        "discord": 1000
      }
    }
  }
}
```

### 권장 사항
- WhatsApp: 5000ms (메시지 여러 개 한 번에 보내는 경우 많음)
- Discord: 1000ms
- `0` — 디바운스 비활성화

---

## 세션 리셋 설정

### 용도
- 오래된 세션 자동 초기화
- 일별/유휴 기반 리셋 정책 설정

### 설정 예제
```json5
{
  "session": {
    "reset": {
      "mode": "idle",
      "idleMinutes": 120
    },
    "resetByType": {
      "thread": { "mode": "daily", "atHour": 4 },
      "direct": { "mode": "idle", "idleMinutes": 240 },
      "group": { "mode": "idle", "idleMinutes": 60 }
    }
  }
}
```

### 권장 사항
- `daily` — 매일 새벽 4시 리셋 (깔끔한 컨텍스트)
- `idle` — 유휴 시간 기반 리셋 (자연스러운 대화 흐름)
- 그룹 채팅은 더 짧은 idle 시간 권장

---

## 루프 감지 설정

### 용도
- 에이전트가 같은 도구를 반복 호출하는 루프 방지
- 무한 루프로 인한 비용 폭주 차단

### 설정 예제
```json5
{
  "tools": {
    "loopDetection": {
      "enabled": true,
      "historySize": 30,
      "warningThreshold": 10,
      "criticalThreshold": 20,
      "globalCircuitBreakerThreshold": 30,
      "detectors": {
        "genericRepeat": true,
        "knownPollNoProgress": true,
        "pingPong": true
      }
    }
  }
}
```

### 권장 사항
- 기본적으로 비활성화 — 코딩 에이전트 사용 시 활성화 권장
- `warningThreshold: 10` — 10회 반복 시 경고
- `criticalThreshold: 20` — 20회 반복 시 차단
- `pingPong` — 두 도구가 번갈아 호출되는 패턴 감지

---

## 모델 Failover 설정

### 용도
- 주 모델 실패 시 자동으로 대체 모델 사용
- API 장애나 속도 제한 대응

### 설정 예제
```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-8",
        "fallbacks": [
          "openai/gpt-5.4",
          "openai/gpt-4.1-mini"
        ]
      }
    }
  }
}
```

### 권장 사항
- Anthropic → OpenAI → 무료 모델 순서로 fallback
- 비용 고려: fallback은 주 모델보다 저렴한 것 권장
- `fallbacks: []` — fallback 완전 비활성화

---

## Secrets 관리

### 용도
- API 키를 config 파일에 평문 저장 방지
- 환경 변수, 파일, 외부 vault에서 시크릿 로드

### 설정 예제
```json5
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-...",
    "OPENAI_API_KEY": "sk-..."
  },
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

### SecretRef 방식 (파일 기반)
```json5
{
  "secrets": {
    "providers": {
      "filemain": {
        "source": "file",
        "path": "~/.openclaw/secrets.json",
        "mode": "json"
      }
    }
  },
  "channels": {
    "telegram": {
      "botToken": { "source": "file", "provider": "filemain", "id": "/telegram/botToken" }
    }
  }
}
```

### 권장 사항
- `${VAR_NAME}` 형식으로 env 변수 참조
- SecretRef로 키를 파일이나 vault에서 로드
- `.env` 파일: `~/.openclaw/.env` 사용 (config 파일에 평문 금지)

---

## 웹 검색 도구 설정

### 용도
- Brave Search API 연동
- 검색 결과 캐싱으로 비용 절감

### 설정 예제
```json5
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "apiKey": "${BRAVE_API_KEY}",
        "maxResults": 5,
        "timeoutSeconds": 30,
        "cacheTtlMinutes": 15
      },
      "fetch": {
        "enabled": true,
        "maxChars": 50000,
        "timeoutSeconds": 30,
        "cacheTtlMinutes": 15
      }
    }
  }
}
```

### 권장 사항
- `cacheTtlMinutes: 15` — 같은 검색어 반복 시 캐시 활용
- `maxResults: 5` — 적은 결과로 토큰 절약
- `maxChars: 50000` — 페이지 내용 최대 크기 제한

---

## Canvas Host 설정

### 용도
- 에이전트가 HTML/CSS/JS 파일 편집 + 브라우저 미리보기
- A2UI (Agent-to-UI) 인터페이스

### 설정 예제
```json5
{
  "canvasHost": {
    "root": "~/.openclaw/workspace/canvas",
    "liveReload": true
  }
}
```

### 권장 사항
- `liveReload: true` — 파일 변경 시 브라우저 자동 새로고침
- 대용량 디렉토리나 `EMFILE` 오류 시 `liveReload: false`
- URL: `http://127.0.0.1:18789/__openclaw__/canvas/`

---

## 응답 prefix 커스터마이징

### 용도
- 채널별 응답 앞에 식별자 추가
- 모델명, 에이전트명 등 메타데이터 표시

### 설정 예제
```json5
{
  "messages": {
    "responsePrefix": "🦞"
  },
  "channels": {
    "discord": {
      "responsePrefix": "[{model}]"
    },
    "telegram": {
      "responsePrefix": "[{identity.name}]"
    }
  }
}
```

### 템플릿 변수
| 변수 | 설명 | 예시 |
|------|------|------|
| `{model}` | 짧은 모델명 | `claude-opus-4-8` |
| `{modelFull}` | 전체 모델 ID | `anthropic/claude-opus-4-8` |
| `{provider}` | 프로바이더명 | `anthropic` |
| `{thinkingLevel}` | 현재 사고 레벨 | `high`, `low`, `off` |
| `{identity.name}` | 에이전트 이름 | `베일리` |

### 권장 사항
- `""` — prefix 완전 비활성화
- `"auto"` — `[{identity.name}]` 자동 설정
- 채널별 다른 prefix 설정 가능

---

## 큐 모드 설정 (메시지 처리 방식)

### 용도
- 여러 메시지 동시 수신 시 처리 방식 결정
- 메시지 드랍, 요약, 스티어링 정책

### 설정 예제
```json5
{
  "messages": {
    "queue": {
      "mode": "collect",
      "debounceMs": 1000,
      "cap": 20,
      "drop": "summarize",
      "byChannel": {
        "whatsapp": "collect",
        "discord": "steer"
      }
    }
  }
}
```

### 큐 모드 종류
| 모드 | 동작 |
|------|------|
| `collect` | 메시지 모아서 한 번에 처리 |
| `steer` | 새 메시지가 현재 진행 중인 응답 방향 조정 |
| `followup` | 응답 후 다음 메시지 처리 |
| `interrupt` | 새 메시지가 현재 응답 중단 |
| `queue` | 순서대로 큐에 넣어 처리 |

### 권장 사항
- WhatsApp: `collect` (여러 메시지 묶음 처리)
- Discord: `steer` (빠른 대화 흐름)
- `drop: "summarize"` — 큐 초과 시 요약해서 처리

---

*조사 완료: 2026-03-25*
