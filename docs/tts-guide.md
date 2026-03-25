# TTS (Text-to-Speech) Guide

> Convert agent responses to voice automatically

[English](tts-guide.md) | [한국어](tts-guide.ko.md) | [← Back to Cookbook](../README.md)

---

## 📍 Config Location

```
~/.openclaw/openclaw.json
```

## 🎯 Basic Setup

### Edge TTS (Free, No API Key)

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

### ElevenLabs (Premium Quality)

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

## ⚙️ Auto Modes

| Mode | Description |
|------|-------------|
| `"always"` | All responses converted to voice |
| `"inbound"` | Only when receiving voice messages |
| `"tagged"` | Only `/tts` command tagged responses |
| `"off"` | TTS disabled |

## 🎙️ Voice Options

### Edge TTS Voices (Free)

| Voice | Language |
|-------|----------|
| `ko-KR-SunHiNeural` | Korean (Female) |
| `ko-KR-InJoonNeural` | Korean (Male) |
| `en-US-JennyNeural` | English (Female) |
| `en-US-GuyNeural` | English (Male) |

### ElevenLabs Settings

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

## 📝 Example Configs

### Korean Voice (Free)

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

### Long Response Handling

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

## 📚 References

- [OpenClaw TTS Docs](https://docs.openclaw.ai/concepts/tts)
