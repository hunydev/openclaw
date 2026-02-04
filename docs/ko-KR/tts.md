---
read_when:
  - 응답에 텍스트 음성 변환을 활성화할 때
  - TTS 제공자 또는 제한을 구성할 때
  - /tts 명령을 사용할 때
summary: 아웃바운드 응답용 텍스트 음성 변환(TTS)
title: 텍스트 음성 변환
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 텍스트 음성 변환(TTS)

OpenClaw는 ElevenLabs, OpenAI 또는 Edge TTS를 사용하여 아웃바운드 응답을 오디오로 변환할 수 있습니다.
OpenClaw가 오디오를 보낼 수 있는 곳이면 어디서나 작동합니다; Telegram은 원형 음성 메시지 버블을 표시합니다.

## 지원 서비스

- **ElevenLabs**(기본 또는 대체 제공자)
- **OpenAI**(기본 또는 대체 제공자; 요약에도 사용)
- **Edge TTS**(기본 또는 대체 제공자; `node-edge-tts` 사용, API 키 없을 때 기본 옵션)

### Edge TTS 참고

Edge TTS는 `node-edge-tts` 라이브러리를 통해 Microsoft Edge의 온라인 신경망 TTS 서비스를 사용합니다. 로컬이 아닌 호스팅 서비스이며 Microsoft의 엔드포인트를 사용하고 API 키가 필요 없습니다.

Edge TTS는 공개 SLA나 할당량이 없는 공용 웹 서비스이므로 최선의 노력으로 취급하세요. 보장된 제한과 지원이 필요하면 OpenAI 또는 ElevenLabs를 사용하세요.

## 선택적 키

OpenAI 또는 ElevenLabs를 사용해야 하는 경우:

- `ELEVENLABS_API_KEY`(또는 `XI_API_KEY`)
- `OPENAI_API_KEY`

Edge TTS는 API 키가 **필요 없습니다**. API 키가 없으면 OpenClaw는 기본적으로 Edge TTS를 사용합니다(`messages.tts.edge.enabled=false`로 비활성화하지 않는 한).

여러 제공자가 구성된 경우 선택한 제공자가 우선이고 나머지는 대체 옵션입니다.
자동 요약은 구성된 `summaryModel`(또는 `agents.defaults.model.primary`)을 사용하므로 요약을 활성화하면 해당 제공자도 인증되어야 합니다.

## 기본적으로 활성화되어 있나요?

아니요. 자동 TTS는 기본적으로 **꺼져** 있습니다. 구성의 `messages.tts.auto` 또는 각 세션에서 `/tts always`(별칭: `/tts on`)로 활성화합니다.

TTS가 켜지면 Edge TTS는 기본적으로 **활성화**되어 있으며 OpenAI나 ElevenLabs API 키가 없을 때 자동으로 사용됩니다.

## 구성

TTS 구성은 `openclaw.json`의 `messages.tts` 아래에 있습니다.
전체 스키마는 [Gateway 구성](/gateway/configuration)을 참조하세요.

### 최소 구성(활성화 + 제공자)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI 기본, ElevenLabs 대체

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS 기본(API 키 불필요)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### Edge TTS 비활성화

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### 사용자 정의 제한 + 환경 설정 경로

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### 음성 메시지를 받은 후에만 오디오로 응답

```json5
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### 필드 설명

- `auto`: 자동 TTS 모드(`off`, `always`, `inbound`, `tagged`).
  - `inbound`는 음성 메시지를 받은 후에만 오디오를 보냅니다.
  - `tagged`는 응답에 `[[tts]]` 태그가 포함된 경우에만 오디오를 보냅니다.
- `enabled`: 레거시 토글(doctor가 `auto`로 마이그레이션).
- `mode`: `"final"`(기본값) 또는 `"all"`(도구/블록 응답 포함).
- `provider`: `"elevenlabs"`, `"openai"` 또는 `"edge"`(대체는 자동).
- `provider`가 **설정되지 않으면** OpenClaw는 `openai`(키가 있는 경우), 그 다음 `elevenlabs`(키가 있는 경우), 그렇지 않으면 `edge`를 우선합니다.
- `summaryModel`: 자동 요약을 위한 선택적 저비용 모델; 기본값은 `agents.defaults.model.primary`.
- `modelOverrides`: 모델이 TTS 지시를 내릴 수 있게 합니다(기본값 켜짐).
- `maxTextLength`: TTS 입력의 하드 제한(문자).
- `timeoutMs`: 요청 타임아웃(밀리초).
- `prefsPath`: 로컬 환경 설정 JSON 경로 재정의.

## 모델 기반 재정의(기본값 켜짐)

기본적으로 모델은 단일 응답에 대해 TTS 지시를 **내릴 수 있습니다**.
`messages.tts.auto`가 `tagged`일 때 이러한 지시가 오디오를 트리거하는 데 필요합니다.

활성화되면 모델은 `[[tts:...]]` 지시를 내려 단일 응답의 음성을 재정의할 수 있고, 선택적 `[[tts:text]]...[[/tts:text]]` 블록으로 오디오에만 나타나는 표현 태그(웃음, 노래 큐 등)를 제공할 수 있습니다.

예시 응답:

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

모든 모델 재정의 비활성화:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

## 사용자 환경 설정

슬래시 명령은 로컬 재정의를 `prefsPath`에 씁니다(기본값: `~/.openclaw/settings/tts.json`).

저장되는 필드:

- `enabled`
- `provider`
- `maxLength`(요약 임계값; 기본값 1500자)
- `summarize`(기본값 `true`)

이러한 설정은 해당 호스트의 `messages.tts.*`를 재정의합니다.

## 출력 형식(고정)

- **Telegram**: Opus 음성 메시지(ElevenLabs는 `opus_48000_64`, OpenAI는 `opus`).
- **기타 채널**: MP3(ElevenLabs는 `mp3_44100_128`, OpenAI는 `mp3`).
- **Edge TTS**: `edge.outputFormat` 사용(기본값 `audio-24khz-48kbitrate-mono-mp3`).

## 자동 TTS 동작

활성화되면 OpenClaw:

- 응답에 이미 미디어나 `MEDIA:` 지시가 포함되어 있으면 TTS를 건너뜁니다.
- 매우 짧은 응답(< 10자)을 건너뜁니다.
- 활성화된 경우 `agents.defaults.model.primary`(또는 `summaryModel`)를 사용하여 긴 응답을 요약합니다.
- 생성된 오디오를 응답에 첨부합니다.

응답이 `maxLength`를 초과하고 요약이 꺼져 있으면(또는 요약 모델에 API 키가 없으면) 오디오를 건너뛰고 일반 텍스트 응답을 보냅니다.

## 슬래시 명령 사용법

명령은 하나입니다: `/tts`.
활성화 세부 정보는 [슬래시 명령](/tools/slash-commands)을 참조하세요.

Discord 참고: `/tts`는 Discord의 내장 명령이므로 OpenClaw는 해당 플랫폼에서 `/voice`를 네이티브 명령으로 등록합니다. 텍스트 명령 `/tts ...`는 여전히 작동합니다.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

참고:

- 명령에는 승인된 발신자가 필요합니다(허용 목록/소유자 규칙이 여전히 적용).
- `commands.text` 또는 네이티브 명령 등록이 활성화되어야 합니다.
- `off|always|inbound|tagged`는 세션별 토글입니다(`/tts on`은 `/tts always`의 별칭).
- `limit`와 `summary`는 메인 구성이 아닌 로컬 환경 설정에 저장됩니다.
- `/tts audio`는 일회성 오디오 응답을 생성합니다(TTS 토글을 전환하지 않음).

## 에이전트 도구

`tts` 도구는 텍스트를 음성으로 변환하고 `MEDIA:` 경로를 반환합니다. 결과가 Telegram과 호환되면 도구에 `[[audio_as_voice]]`가 포함되어 Telegram이 음성 버블을 보냅니다.

## Gateway RPC

Gateway 메서드:

- `tts.status`
- `tts.enable`
- `tts.disable`
- `tts.convert`
- `tts.setProvider`
- `tts.providers`
