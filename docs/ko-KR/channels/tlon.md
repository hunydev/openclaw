---
read_when:
  - Tlon/Urbit 채널 기능 개발 시
summary: Tlon/Urbit 지원 상태, 기능 및 설정
title: Tlon
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 19d7ffe23e82239fd2a2e35913e0d52c809b2c2b939dd39184e6c27a539ed97d
  source_path: channels/tlon.md
  workflow: 14
---

# Tlon(플러그인)

Tlon은 Urbit 위에 구축된 탈중앙화 메신저입니다. OpenClaw는 Urbit ship에 연결하여 DM과 그룹 채팅 메시지에 응답할 수 있습니다. 그룹 채팅 응답은 기본적으로 @ 멘션이 필요하며 허용 목록으로 추가 제한할 수 있습니다.

상태: 플러그인을 통해 지원. DM, 그룹 멘션, 스레드 응답 및 일반 텍스트 미디어 폴백(URL이 캡션에 추가됨) 지원. 반응, 투표, 네이티브 미디어 업로드는 지원하지 않습니다.

## 플러그인 필요

Tlon은 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

CLI를 통해 설치(npm 레지스트리):

```bash
openclaw plugins install @openclaw/tlon
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/tlon
```

자세한 내용: [플러그인](/plugin)

## 설정

1. Tlon 플러그인 설치.
2. ship URL과 로그인 코드 획득.
3. `channels.tlon` 설정.
4. Gateway 재시작.
5. 봇에게 DM을 보내거나 그룹 채널에서 멘션.

최소 설정(단일 계정):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

## 그룹 채널

자동 검색이 기본적으로 활성화됩니다. 수동으로 채널을 고정할 수도 있습니다:

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

자동 검색 비활성화:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## 접근 제어

DM 허용 목록(비어 있으면 = 모두 허용):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

그룹 권한(기본값은 제한 모드):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## 전달 대상(CLI/크론)

`openclaw message send` 또는 크론 전달과 함께 사용:

- DM: `~sampel-palnet` 또는 `dm/~sampel-palnet`
- 그룹: `chat/~host-ship/channel` 또는 `group:~host-ship/channel`

## 참고

- 그룹 채팅 응답은 멘션이 필요합니다(예: `~your-bot-ship`).
- 스레드 응답: 받은 메시지가 스레드에 있으면 OpenClaw는 스레드 내에서 응답합니다.
- 미디어: `sendMedia`는 텍스트 + URL로 폴백합니다(네이티브 업로드 미지원).
