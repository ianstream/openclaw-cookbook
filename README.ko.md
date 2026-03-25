# OpenClaw Cookbook 📖

> 실전에서 검증된 OpenClaw 설정 패턴과 베스트 프랙티스

[English](README.md) | [한국어](README.ko.md)

---

## 📁 레포 구조

```
openclaw-cookbook/
├── README.md                    # English
├── README.ko.md                 # 한국어
├── docs/
│   ├── token-optimization.md    # Token optimization (EN)
│   ├── token-optimization.ko.md # 토큰 최적화 (KO)
│   ├── multi-agent-guide.md     # 다중 에이전트 설정
│   ├── channel-binding-guide.md # 채널 바인딩
│   └── memory-flush-guide.md    # 메모리 플러시
└── examples/
    ├── token-optimization/      # 컨텍스트 프루닝 & 컴팩션
    ├── multi-agent/             # 에이전트 정의
    ├── channel-binding/         # 채널별 바인딩
    ├── memory-flush/            # 컴팩션 전 메모리 저장
    └── full-config/             # 전체 설정 예제
```

## 🎯 설정 패턴

### 1. 토큰 최적화

비용 절감 및 컨텍스트 윈도우 관리.

- **Context Pruning**: LLM 호출 전 오래된 tool 결과 제거
- **Compaction**: 컨텍스트 꽉 차면 대화 요약

→ [상세 가이드](docs/token-optimization.ko.md)

### 2. 다중 에이전트

용도별로 다른 모델/설정을 가진 에이전트 운영.

- **code**: 코딩 작업용 (Opus)
- **analyst**: 분석/리서치용 (Sonnet)
- **lite**: 간단한 대화용 (Haiku)

→ [상세 가이드](docs/multi-agent-guide.md)

### 3. 채널 바인딩

Discord/Slack 채널별로 다른 에이전트와 시스템 프롬프트.

- `#tech` → code 에이전트 + 개발자 페르소나
- `#quant` → analyst 에이전트 + 투자 전문가 페르소나

→ [상세 가이드](docs/channel-binding-guide.md)

### 4. 메모리 플러시

컴팩션 전 중요 내용 자동 저장.

- 세션 요약 전 `memory/YYYY-MM-DD.md`에 기록
- 아키텍처 결정, 버그 픽스, 패턴 등 보존

→ [상세 가이드](docs/memory-flush-guide.md)

## 🚀 빠른 시작

### 1. 예제 설정 복사

```bash
cp examples/full-config/openclaw.example.json ~/.openclaw/openclaw.json
```

### 2. 설정 수정

```bash
# 플레이스홀더 교체
YOUR_DISCORD_BOT_TOKEN
YOUR_GUILD_ID
YOUR_DISCORD_USER_ID
TECH_CHANNEL_ID
```

### 3. OpenClaw 재시작

```bash
openclaw gateway restart
```

## 📚 참고 문서

- [OpenClaw 공식 문서](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Compaction 공식 문서](https://github.com/openclaw/openclaw/blob/main/docs/concepts/compaction.md)
- [Session Pruning 공식 문서](https://github.com/openclaw/openclaw/blob/main/docs/concepts/session-pruning.md)

## 🤝 기여하기

유용한 패턴이 있다면 PR 환영합니다!

## License

MIT
