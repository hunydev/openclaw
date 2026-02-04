---
read_when:
  - DM 접근 제어를 설정할 때
  - 새로운 iOS/Android 노드를 페어링할 때
  - OpenClaw 보안 상태를 검토할 때
summary: "페어링 개요: DM 허용 대상 및 Gateway 참여 노드 승인"
title: 페어링
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: c46a5c39f289c8fd0783baacd927f550c3d3ae8889a7bc7de133b795f16fa08a
  source_path: start/pairing.md
  workflow: 15
---

# 페어링

"페어링"은 OpenClaw의 명시적인 **소유자 승인** 단계입니다.
다음 두 가지 상황에서 사용됩니다:

1. **DM 페어링** (봇과 대화할 수 있는 사용자 지정)
2. **노드 페어링** (Gateway 네트워크에 참여할 수 있는 디바이스/노드 지정)

보안 관련 내용: [보안](/gateway/security)

## 1) DM 페어링 (인바운드 채팅 접근)

채널이 DM 정책 `pairing`으로 설정된 경우, 알 수 없는 발신자에게 짧은 코드가 발송되며 승인하기 전까지 해당 메시지는 **처리되지 않습니다**.

기본 DM 정책은 다음 문서에서 확인할 수 있습니다: [보안](/gateway/security)

페어링 코드:

- 8자리, 대문자, 혼동하기 쉬운 문자 제외 (`0O1I`).
- **1시간 후 만료**. 봇은 새 요청이 생성될 때만 페어링 메시지를 발송합니다 (발신자당 대략 1시간에 한 번).
- 대기 중인 DM 페어링 요청은 기본적으로 **채널당 최대 3개**로 제한됩니다. 추가 요청은 기존 요청이 만료되거나 승인될 때까지 무시됩니다.

### 발신자 승인

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

지원 채널: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### 상태 저장 위치

`~/.openclaw/credentials/` 아래에 저장됩니다:

- 대기 중인 요청: `<channel>-pairing.json`
- 승인된 허용 목록: `<channel>-allowFrom.json`

이 파일들은 어시스턴트에 대한 접근 권한을 제어하므로 민감하게 취급하세요.

## 2) 노드 디바이스 페어링 (iOS/Android/macOS/헤드리스 노드)

노드는 `role: node`로 **디바이스**로서 Gateway에 연결됩니다. Gateway는 승인이 필요한 디바이스 페어링 요청을 생성합니다.

### 노드 디바이스 승인

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### 상태 저장 위치

`~/.openclaw/devices/` 아래에 저장됩니다:

- `pending.json` (단기 유효; 대기 중인 요청은 만료됨)
- `paired.json` (페어링된 디바이스 + 토큰)

### 참고 사항

- 레거시 `node.pair.*` API (CLI: `openclaw nodes pending/approve`)는 Gateway가 소유한 별도의 페어링 저장소입니다. WebSocket 노드는 여전히 디바이스 페어링이 필요합니다.

## 관련 문서

- 보안 모델 + 프롬프트 인젝션: [보안](/gateway/security)
- 안전하게 업데이트하기 (doctor 실행): [업데이트](/install/updating)
- 채널 설정:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - iMessage: [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)
