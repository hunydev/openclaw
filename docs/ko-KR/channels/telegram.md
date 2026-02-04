---
read_when:
  - Telegram 채널을 구성하거나 디버깅할 때
  - 운영자가 Telegram 통합에 대해 질문할 때
summary: Telegram에 연결하기 위한 설치 및 구성 가이드
title: Telegram
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Telegram

OpenClaw는 Telegram 봇 통합을 지원합니다:

- 폴링 모드(권장, 퍼블릭 IP 불필요)
- 웹훅 모드(인바운드 URL 필요)

Telegram 메시지는 Pi 에이전트로 전달됩니다.

## 빠른 시작

1. [BotFather](https://t.me/botfather)에서 봇 생성:
   - `/newbot` 명령 사용
   - 봇 이름과 사용자 이름 설정
   - API 토큰 복사
2. `config.json`에 토큰 추가:

```json5
{
  channels: {
    telegram: {
      token: "123456:ABC-DEF...",
    },
  },
}
```

3. Gateway 재시작.

## 봇 설정(BotFather)

BotFather에서 권장 설정:

1. `/mybots` → 봇 선택 → **Bot Settings**
2. **Group Privacy** → **Turn off** (그룹 메시지 읽기 허용)
3. **Allow Groups?** → **Turn on** (그룹 참가 허용)
4. 선택: **/setdescription**, **/setabouttext**, **/setuserpic**

**중요**: 그룹에서 봇이 메시지를 받으려면 **Group Privacy를 끄거나** 봇을 관리자로 설정해야 합니다.

## DM 정책

기본 모드는 `allowlist`입니다. `dmPolicy`로 변경:

```json5
{
  channels: {
    telegram: {
      dmPolicy: "open", // 모든 DM 허용
      // 또는 "disabled" 로 모든 DM 무시
      // 또는 "allowlist" (기본값, allowFrom 필요)
      // 또는 "pairing" 으로 페어링 흐름 활성화
    },
  },
}
```

`allowlist` 모드에서는 `allowFrom`이 필요합니다:

```json5
{
  channels: {
    telegram: {
      dmPolicy: "allowlist",
      allowFrom: [123456789], // Telegram 사용자 ID (숫자)
    },
  },
}
```

**사용자 ID 찾기**: [@userinfobot](https://t.me/userinfobot)에게 메시지를 보내면 ID를 알려줍니다.

## 페어링 모드

`dmPolicy: "pairing"`을 사용하면 알 수 없는 사용자가 페어링 코드를 받습니다:

```json5
{
  channels: {
    telegram: {
      dmPolicy: "pairing",
    },
  },
}
```

페어링 흐름:

1. 새 사용자가 봇에게 메시지 전송
2. 봇이 6자리 페어링 코드 반환
3. 관리자가 승인: `openclaw pairing approve telegram <code>`
4. 승인 후 사용자가 봇 사용 가능

페어링 관리:

```bash
openclaw pairing list telegram     # 대기 중인 요청 목록
openclaw pairing approve telegram <code>  # 승인
openclaw pairing reject telegram <code>   # 거부
```

## 그룹 채팅

Telegram 그룹/슈퍼그룹에서 봇 사용:

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist", // allowlist, open, disabled
      groups: [-1001234567890], // 허용 그룹 ID (음수)
      groupChat: {
        activationMode: "mention", // mention (기본값), always
        mentionPatterns: ["claude", "ai"], // 추가 트리거 단어
        historyLimit: 50, // 컨텍스트에 포함할 메시지 수
      },
    },
  },
}
```

**그룹 ID 찾기**: 그룹에 봇을 추가하고 메시지를 보내면 로그에서 그룹 ID를 확인할 수 있습니다.

### 활성화 모드

- `mention`: @봇이름 또는 `mentionPatterns`의 단어가 포함된 메시지에만 응답
- `always`: 그룹의 모든 메시지에 응답

### 인라인 명령(그룹용)

```
/activation mention   # 멘션 모드로 전환
/activation always    # 항상 응답 모드로 전환
```

## 스레딩

Telegram은 답장(Reply) 기반 스레딩을 지원합니다:

```json5
{
  channels: {
    telegram: {
      threading: "reply", // 항상 원본 메시지에 답장 (기본값)
      // 또는 "none" 으로 스레딩 비활성화
    },
  },
}
```

**토픽(포럼) 지원**: 슈퍼그룹의 토픽은 자동으로 인식되며, 응답은 같은 토픽에 전송됩니다.

## 미디어

Telegram은 다양한 미디어 유형을 지원합니다.

**인바운드(수신):**

- 사진, 동영상, 오디오, 문서, 음성 메시지
- 스티커, GIF, 비디오 노트
- `mediaMaxMb`로 크기 제한(기본값 50MB)

**아웃바운드(전송):**

- 이미지, 동영상, 오디오, 문서
- `agents.defaults.mediaMaxMb`로 제한(기본값 5MB)

```json5
{
  channels: {
    telegram: {
      mediaMaxMb: 50, // 인바운드 제한
    },
  },
  agents: {
    defaults: {
      mediaMaxMb: 5, // 아웃바운드 제한
    },
  },
}
```

## 스티커

스티커 수신 및 전송 지원:

```json5
{
  channels: {
    telegram: {
      stickers: {
        enabled: true, // 스티커 지원 활성화
        sendAsImage: false, // true면 스티커를 이미지로 변환
      },
    },
  },
}
```

## 음성 메시지

음성 메시지 수신 시 자동으로 텍스트 변환(Whisper 사용):

```json5
{
  channels: {
    telegram: {
      voice: {
        transcribe: true, // 음성을 텍스트로 변환
        // transcriptionModel: "whisper-1" // 선택: 모델 지정
      },
    },
  },
}
```

## 반응

메시지에 이모지 반응 추가:

```json5
{
  channels: {
    telegram: {
      actions: {
        reactions: true, // 기본값: 활성화
      },
    },
  },
}
```

에이전트 도구를 통해 반응 추가/제거 가능.

## 확인 반응

메시지 수신 시 자동 이모지 반응:

```json5
{
  channels: {
    telegram: {
      ackReaction: {
        emoji: "👀", // 확인 이모지
        direct: true, // DM에서 반응
        group: "mentions", // 그룹에서: "always", "mentions", "never"
      },
    },
  },
}
```

**참고**: Telegram 반응은 표준 이모지 또는 커스텀 이모지 ID를 사용합니다.

## 타이핑 표시기

봇이 응답을 생성하는 동안 "typing..." 표시:

```json5
{
  channels: {
    telegram: {
      typingIndicator: true, // 기본값: 활성화
    },
  },
}
```

## 스트리밍

Telegram은 메시지 편집을 통한 스트리밍을 지원합니다:

```json5
{
  channels: {
    telegram: {
      streaming: true, // 스트리밍 응답 활성화
      streamingThrottle: 500, // 편집 간격(ms), 기본값 500
    },
  },
}
```

**동작**: 응답이 생성되면서 메시지가 점진적으로 업데이트됩니다.

**제한**: Telegram API 속도 제한으로 인해 편집 간격이 너무 짧으면 제한될 수 있습니다.

## 인라인 모드(선택)

Telegram 인라인 쿼리 지원:

1. BotFather에서 인라인 모드 활성화: `/setinline`
2. 구성:

```json5
{
  channels: {
    telegram: {
      inline: {
        enabled: true,
        placeholder: "Ask me anything...", // 입력 플레이스홀더
      },
    },
  },
}
```

사용: 아무 채팅에서 `@봇이름 질문`을 입력하면 인라인 결과 표시.

## 명령 메뉴

BotFather에서 명령 설정:

```
/setcommands
help - 도움말 표시
clear - 대화 초기화
settings - 설정 보기
```

OpenClaw 내장 명령:

- `/help` - 도움말
- `/clear` - 세션 초기화
- `/config` - 구성 보기/변경

## 웹훅 모드(대안)

폴링 대신 웹훅 사용:

```json5
{
  channels: {
    telegram: {
      mode: "webhook",
      webhookUrl: "https://your-domain.com/telegram/webhook",
      webhookSecret: "your-secret", // 선택: 웹훅 검증
    },
  },
}
```

**요구 사항:**

- 퍼블릭 HTTPS URL 필요
- 유효한 SSL 인증서
- 포트 443, 80, 88, 또는 8443

**설정**: Gateway 시작 시 웹훅 자동 등록.

## 다중 봇

여러 Telegram 봇 지원:

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          token: "123456:ABC...",
          dmPolicy: "allowlist",
          allowFrom: [123456789],
        },
        support: {
          token: "789012:DEF...",
          dmPolicy: "open",
        },
      },
    },
  },
}
```

## 봇 명령 게이팅

특정 명령 비활성화:

```json5
{
  channels: {
    telegram: {
      commands: {
        config: false, // /config 비활성화
        clear: true, // /clear 활성화 (기본값)
      },
    },
  },
}
```

## 메시지 형식

Telegram은 Markdown 및 HTML을 지원합니다:

```json5
{
  channels: {
    telegram: {
      parseMode: "MarkdownV2", // 또는 "HTML", "Markdown"
    },
  },
}
```

**MarkdownV2**(권장):

- 볼드: `*text*`
- 이탤릭: `_text_`
- 코드: `` `code` ``
- 코드 블록: ` ```code``` `
- 링크: `[text](url)`

## 메시지 제한

긴 메시지 처리:

```json5
{
  channels: {
    telegram: {
      textChunkLimit: 4096, // 메시지당 최대 문자 수 (Telegram 제한)
      chunkMode: "length", // length (기본값) 또는 newline
    },
  },
}
```

긴 응답은 자동으로 여러 메시지로 분할됩니다.

## 구성 요약

| 키                                    | 설명                        | 기본값      |
| :------------------------------------ | :-------------------------- | :---------- |
| `channels.telegram.token`             | 봇 API 토큰                 | -           |
| `channels.telegram.mode`              | `polling` 또는 `webhook`    | `polling`   |
| `channels.telegram.webhookUrl`        | 웹훅 URL(웹훅 모드)         | -           |
| `channels.telegram.dmPolicy`          | DM 정책                     | `allowlist` |
| `channels.telegram.allowFrom`         | 허용 사용자 ID              | `[]`        |
| `channels.telegram.groupPolicy`       | 그룹 정책                   | `allowlist` |
| `channels.telegram.groups`            | 허용 그룹 ID                | `[]`        |
| `channels.telegram.threading`         | 스레딩 동작                 | `reply`     |
| `channels.telegram.mediaMaxMb`        | 인바운드 미디어 제한(MB)    | `50`        |
| `channels.telegram.typingIndicator`   | 타이핑 표시기               | `true`      |
| `channels.telegram.streaming`         | 스트리밍 응답               | `false`     |
| `channels.telegram.streamingThrottle` | 스트리밍 편집 간격(ms)      | `500`       |
| `channels.telegram.parseMode`         | 메시지 형식                 | `MarkdownV2`|
| `channels.telegram.textChunkLimit`    | 메시지당 최대 문자 수       | `4096`      |
| `channels.telegram.ackReaction`       | 확인 반응 설정              | -           |
| `channels.telegram.actions.reactions` | 반응 도구 활성화            | `true`      |

## 문제 해결

### 봇이 응답하지 않음

1. `channels status`로 연결 상태 확인
2. 토큰이 올바른지 확인
3. `dmPolicy`와 `allowFrom` 확인
4. 로그 확인: `openclaw logs --follow`

### 그룹에서 메시지를 받지 못함

1. BotFather에서 **Group Privacy** 끄기
2. 또는 봇을 그룹 관리자로 설정
3. `groupPolicy`와 `groups` 확인

### 웹훅 문제

1. URL이 HTTPS인지 확인
2. SSL 인증서가 유효한지 확인
3. 방화벽이 Telegram IP를 허용하는지 확인
4. `openclaw channels login`으로 웹훅 재등록

### 속도 제한

Telegram은 초당 메시지 수를 제한합니다:

- 동일 채팅: 초당 1개
- 전체: 초당 30개

스트리밍 사용 시 `streamingThrottle`을 높이세요.

### "Forbidden" 오류

- 사용자가 봇을 차단했거나
- 봇이 그룹에서 제거되었습니다

## 참고

- [Telegram Bot API](https://core.telegram.org/bots/api)
- [BotFather](https://t.me/botfather)
- [Telegram Bot 튜토리얼](https://core.telegram.org/bots/tutorial)
