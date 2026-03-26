# OpenClaw Cookbook 📖

> 실전 검증된 OpenClaw 설정 패턴과 베스트 프랙티스

[English](README.md) | [한국어](README.ko.md)

---

## 이게 뭔가요?

[OpenClaw](https://github.com/openclaw/openclaw) — 오픈소스 AI 어시스턴트 프레임워크를 위한 **실전 설정 패턴** 모음집.

**누구를 위한 건가요?**
- 기본 설정을 넘어서고 싶은 OpenClaw 사용자
- 프로덕션 레디 설정을 찾는 개발자
- 문서 읽기 지치고 복붙 예제가 필요한 사람

**뭐가 있나요?**
- 동작하는 예제와 함께 12개의 상세 가이드
- 보안 강화 설정
- 다중 에이전트 오케스트레이션 패턴
- 비용 최적화 전략
- 영어/한국어 완전 지원

**왜 만들었나요:**
OpenClaw 공식 문서는 포괄적이지만 흩어져 있음. 이 쿡북은 수개월의 프로덕션 사용 경험을 즉시 사용 가능한 패턴으로 정리함.

---

## 📁 저장소 구조

```
openclaw-cookbook/
├── README.md / README.ko.md
├── docs/
│   ├── token-optimization.ko.md  # 토큰 & 컨텍스트 관리
│   ├── multi-agent-guide.md      # 다중 에이전트 설정
│   ├── channel-binding-guide.md  # 채널별 설정
│   ├── memory-flush-guide.md     # 컴팩션 전 메모리
│   ├── tts-guide.ko.md           # 음성 합성
│   ├── heartbeat-guide.ko.md     # 주기적 체크인
│   ├── cron-guide.ko.md          # 예약 작업
│   ├── subagent-guide.ko.md      # 병렬 백그라운드
│   ├── exec-security-guide.ko.md # 샌드박싱 & 권한
│   ├── security-guide.ko.md      # 전체 보안 강화
│   └── auto-mode.ko.md           # Claude Code Auto Mode
└── examples/
    └── ...
```

## 🚀 설치

**[📖 Installation Guide](docs/installation.md)** | **[📖 설치 가이드](docs/installation.ko.md)**

```bash
# 빠른 설치
npm install -g openclaw
openclaw setup
openclaw auth login anthropic
openclaw gateway start
```

---

## 🎯 설정 패턴

### 1. 토큰 최적화

비용 절감 및 컨텍스트 윈도우 효율적 관리.

- **Context Pruning**: 오래된 tool 결과 제거
- **Compaction**: 오래된 대화 요약

→ [상세 가이드](docs/token-optimization.ko.md)

### 2. 다중 에이전트

용도별로 다른 모델의 에이전트 실행.

- **code**: 코딩 작업 (Opus)
- **analyst**: 분석/리서치 (Sonnet)
- **lite**: 간단한 대화 (Haiku)

→ [상세 가이드](docs/multi-agent-guide.md)

### 3. 채널 바인딩

채널별로 다른 에이전트 + 시스템 프롬프트.

→ [상세 가이드](docs/channel-binding-guide.md)

### 4. 메모리 플러시

컴팩션 전 중요 내용 자동 저장.

→ [상세 가이드](docs/memory-flush-guide.md)

### 5. TTS (음성 합성)

에이전트 응답을 음성으로 변환.

- **Edge TTS**: 무료, API 키 불필요
- **ElevenLabs**: 프리미엄 품질
- **OpenAI TTS**: GPT-4o 음성

→ [상세 가이드](docs/tts-guide.ko.md)

### 6. Heartbeat

주기적 에이전트 체크인 및 백그라운드 작업.

- **Active Hours**: 특정 시간대에만 실행
- **Light Context**: 토큰 절약 모드
- **Custom Channel**: 특정 채널로 알림

→ [상세 가이드](docs/heartbeat-guide.ko.md)

### 7. Cron Jobs

특정 시간에 작업 예약 실행.

- **아침 브리핑**: 매일 요약
- **리마인더**: 일회성 알림
- **리포트**: 주간/월간 자동화

→ [상세 가이드](docs/cron-guide.ko.md)

### 8. Subagents

병렬 백그라운드 작업을 자식 에이전트로 실행.

- **오케스트레이터 패턴**: 메인 → 워커
- **비용 최적화**: 서브태스크에 저렴한 모델
- **도구 제한**: 서브에이전트 권한 제한

→ [상세 가이드](docs/subagent-guide.ko.md)

### 9. Exec 보안

명령어 실행 및 샌드박싱 제어.

- **보안 모드**: full / allowlist / deny
- **Elevated 사용자**: 호스트 명령 권한
- **Docker Sandbox**: 격리된 실행

→ [상세 가이드](docs/exec-security-guide.ko.md)

### 10. 보안 강화

무단 접근으로부터 배포 보호.

- **게이트웨이 인증**: 토큰/비밀번호 인증
- **네트워크 바인딩**: Loopback vs LAN vs Tailnet
- **DM/그룹 정책**: 페어링, 허용 목록
- **민감한 파일**: 권한, 시크릿 관리

→ [상세 가이드](docs/security-guide.ko.md)

### 11. Claude Code Auto Mode

모델 기반 분류기로 권한 프롬프트를 안전하게 건너뛰기.

- **2중 방어**: 입력 탐지 + 출력 분류기
- **3단계 권한**: 안전 자동 허용, 프로젝트 편집, 분류 작업
- **실제 위협 방어**: 과잉 행동, 자격증명 탐색, 데이터 유출

→ [상세 가이드](docs/auto-mode.ko.md)

## 🚀 빠른 시작 (설치 후)

### 1. 예제 설정 복사

```bash
cp examples/full-config/openclaw.example.json ~/.openclaw/openclaw.json
```

### 2. 설정 수정

`YOUR_DISCORD_BOT_TOKEN`, `YOUR_GUILD_ID` 등 플레이스홀더 교체.

### 3. OpenClaw 재시작

```bash
openclaw gateway restart
```

## 📚 참고 문서

- [OpenClaw 공식 문서](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Discord](https://discord.com/invite/clawd)

## 🌐 커뮤니티

유용했다면 공유해주세요!

- **OpenClaw Discord** — 피드백과 토론하기 좋은 곳
- **Twitter/X** — #OpenClaw 태그
- **Reddit** — r/LocalLLaMA, r/ChatGPTCoding
- **Hacker News** — Show HN
- **GeekNews** — 한국 개발자 커뮤니티

## 🤝 기여

PR 환영! 유용한 패턴이 있으면 기여해주세요.

## 라이선스

MIT
