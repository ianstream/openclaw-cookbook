# TTS (음성 합성) 가이드

> 에이전트 응답을 자동으로 음성으로 변환

[English](tts-guide.md) | [한국어](tts-guide.ko.md) | [← Cookbook으로](../README.ko.md)

---

## 📍 설정 파일 위치

```
~/.openclaw/openclaw.json
```

## 🎯 기본 설정

### Edge TTS (무료, API 키 불필요)

```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "edge",
      "edge": {
        "voice": "ko-KR-SunHiNeural",
        "lang": "ko-KR"
      }
    }
  }
}
```

### ElevenLabs (프리미엄 품질)

```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "elevenlabs",
      "elevenlabs": {
        "apiKey": "${ELEVENLABS_API_KEY}",
        "voiceId": "YOUR_VOICE_ID",
        "modelId": "eleven_multilingual_v2"
      }
    }
  }
}
```

### OpenAI TTS

```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "openai",
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "model": "gpt-4o-mini-tts",
        "voice": "alloy"
      }
    }
  }
}
```

## ⚙️ 자동 모드

| 모드 | 설명 |
|------|------|
| `"always"` | 모든 응답 음성 변환 |
| `"inbound"` | 음성 메시지 받을 때만 |
| `"tagged"` | `/tts` 명령어 태그된 것만 |
| `"off"` | TTS 비활성화 |

## 🎙️ 음성 옵션

### Edge TTS 음성 (무료)

| 음성 | 언어 |
|------|------|
| `ko-KR-SunHiNeural` | 한국어 (여성) |
| `ko-KR-InJoonNeural` | 한국어 (남성) |
| `en-US-JennyNeural` | 영어 (여성) |
| `en-US-GuyNeural` | 영어 (남성) |

### ElevenLabs 세부 설정

```json
{
  "elevenlabs": {
    "voiceSettings": {
      "stability": 0.5,
      "similarityBoost": 0.75,
      "speed": 1.0
    }
  }
}
```

## 📝 예제 설정

### 한국어 음성 (무료)

```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "edge",
      "edge": {
        "voice": "ko-KR-SunHiNeural",
        "lang": "ko-KR"
      }
    }
  }
}
```

### 긴 응답 처리

```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "maxTextLength": 4000,
      "summaryModel": "openai/gpt-4.1-mini"
    }
  }
}
```

## 📚 참고 문서

- [OpenClaw TTS 문서](https://docs.openclaw.ai/concepts/tts)
