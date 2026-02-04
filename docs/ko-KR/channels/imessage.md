---
read_when:
  - iMessage 지원 설정 시
  - iMessage 송수신 디버깅 시
summary: imsg(stdio 기반 JSON-RPC)를 통한 iMessage 지원, 설정 및 chat_id 라우팅
title: iMessage
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# iMessage(imsg)

상태: 외부 CLI 통합. Gateway가 `imsg rpc`(stdio 기반 JSON-RPC)를 시작합니다.

## 빠른 설정(초보자)

1. 이 Mac에서 메시지 앱이 로그인되어 있는지 확인.
2. `imsg` 설치:
   - `brew install steipete/tap/imsg`
3. OpenClaw의 `channels.imessage.cliPath`와 `channels.imessage.dbPath` 구성.
4. Gateway를 시작하고 모든 macOS 프롬프트 승인(자동화 + 전체 디스크 접근 권한).

최소 구성:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

## 이것이 무엇인가

- macOS에서 `imsg`로 지원되는 iMessage 채널.
- 결정론적 라우팅: 응답은 항상 iMessage로 돌아갑니다.
- DM은 에이전트의 기본 세션을 공유하며, 그룹은 격리됩니다(`agent:<agentId>:imessage:group:<chat_id>`).
- 다중 참여자 스레드가 `is_group=false`로 도착하는 경우에도 `chat_id`를 사용하여 `channels.imessage.groups`로 격리할 수 있습니다(아래 "그룹과 유사한 스레드" 참조).

## 구성 쓰기

기본적으로 iMessage는 `/config set|unset`으로 트리거되는 구성 업데이트 쓰기를 허용합니다(`commands.config: true` 필요).

비활성화 방법:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## 요구 사항

- macOS이고 메시지 앱이 로그인되어 있어야 함.
- OpenClaw + `imsg`는 전체 디스크 접근 권한이 필요합니다(Messages 데이터베이스 접근).
- 전송 시 자동화 권한이 필요합니다.
- `channels.imessage.cliPath`는 stdin/stdout을 프록시하는 모든 명령을 가리킬 수 있습니다(예: SSH를 통해 다른 Mac에 연결하고 `imsg rpc`를 실행하는 래퍼 스크립트).

## 설정(빠른 경로)

1. 이 Mac에서 메시지 앱이 로그인되어 있는지 확인.
2. iMessage를 구성하고 Gateway 시작.

### 전용 봇 macOS 사용자(격리된 신원용)

봇이 **별도의 iMessage 신원**에서 메시지를 보내도록 하려면(개인 메시지 앱을 깔끔하게 유지), 전용 Apple ID + 전용 macOS 사용자를 사용하세요.

1. 전용 Apple ID 생성(예: `my-cool-bot@icloud.com`).
   - Apple은 인증/2FA를 위해 전화번호가 필요할 수 있습니다.
2. macOS 사용자 생성(예: `openclawhome`)하고 로그인.
3. 해당 macOS 사용자에서 메시지 앱을 열고 봇 Apple ID로 iMessage 로그인.
4. 원격 로그인 활성화(시스템 설정 → 일반 → 공유 → 원격 로그인).
5. `imsg` 설치:
   - `brew install steipete/tap/imsg`
6. SSH가 `ssh <bot-macos-user>@localhost true`를 비밀번호 없이 작동하도록 설정.
7. `channels.imessage.accounts.bot.cliPath`를 봇 사용자로 `imsg`를 실행하는 SSH 래퍼 스크립트로 지정.

첫 실행 참고: 전송/수신에는 *봇 macOS 사용자*에서 GUI 승인이 필요할 수 있습니다(자동화 + 전체 디스크 접근 권한). `imsg rpc`가 멈추거나 종료되면, 해당 사용자로 로그인(화면 공유가 유용함)하고, `imsg chats --limit 1` / `imsg send ...`를 한 번 실행하고, 프롬프트를 승인한 후 다시 시도하세요.

예제 래퍼 스크립트(`chmod +x`). `<bot-macos-user>`를 실제 macOS 사용자 이름으로 교체:

```bash
#!/usr/bin/env bash
set -euo pipefail

# 먼저 대화형 SSH를 한 번 실행하여 호스트 키 수락:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

예제 구성:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
        },
      },
    },
  },
}
```

단일 계정 설정의 경우 `accounts` 맵 대신 플랫 옵션(`channels.imessage.cliPath`, `channels.imessage.dbPath`) 사용.

### 원격/SSH 변형(선택)

다른 Mac에서 iMessage를 사용하려면 `channels.imessage.cliPath`를 SSH를 통해 원격 macOS 호스트에서 `imsg`를 실행하는 래퍼 스크립트로 설정하세요. OpenClaw는 stdio만 필요합니다.

예제 래퍼 스크립트:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**원격 첨부 파일:** `cliPath`가 SSH를 통해 원격 호스트를 가리킬 때, Messages 데이터베이스의 첨부 파일 경로는 원격 머신의 파일을 참조합니다. OpenClaw는 `channels.imessage.remoteHost`를 설정하여 SCP를 통해 이러한 파일을 자동으로 가져올 수 있습니다:

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // 원격 Mac으로의 SSH 래퍼 스크립트
      remoteHost: "user@gateway-host", // SCP 파일 전송용
      includeAttachments: true,
    },
  },
}
```

`remoteHost`가 설정되지 않으면 OpenClaw는 래퍼 스크립트의 SSH 명령을 파싱하여 자동 감지를 시도합니다. 안정성을 위해 명시적 구성을 권장합니다.

#### Tailscale을 통한 원격 Mac 연결(예제)

Gateway가 Linux 호스트/VM에서 실행되지만 iMessage가 Mac에서 실행되어야 하는 경우, Tailscale이 가장 쉬운 브릿지 솔루션입니다: Gateway가 tailnet을 통해 Mac과 통신하고, SSH를 통해 `imsg`를 실행하고, SCP를 통해 첨부 파일을 다시 전송합니다.

아키텍처:

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway 호스트(Linux/VM)      │──────────────────────────────────▶│ Messages + imsg가 있는 Mac │
│ - openclaw gateway           │          SCP(첨부 파일)           │ - Messages 로그인됨        │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - 원격 로그인 활성화       │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet(호스트명 또는 100.x.y.z)
              ▼
        user@gateway-host
```

구체적 구성 예제(Tailscale 호스트명):

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

예제 래퍼 스크립트(`~/.openclaw/scripts/imsg-ssh`):

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

참고:

- Mac이 메시지 앱에 로그인되어 있고 원격 로그인이 활성화되어 있는지 확인.
- SSH 키를 사용하여 `ssh bot@mac-mini.tailnet-1234.ts.net`이 프롬프트 없이 작동하도록 설정.
- `remoteHost`는 SSH 대상과 일치해야 SCP가 첨부 파일을 가져올 수 있습니다.

다중 계정 지원: `channels.imessage.accounts`를 사용하고, 각 계정에 독립적인 옵션과 선택적 `name` 구성. 공유 모드는 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 참조. `~/.openclaw/openclaw.json`을 커밋하지 마세요(보통 토큰 포함).

## 접근 제어(DM + 그룹)

DM:

- 기본값: `channels.imessage.dmPolicy = "pairing"`.
- 알 수 없는 발신자는 페어링 코드를 받으며, 승인 전까지 메시지는 무시됩니다(페어링 코드는 1시간 후 만료).
- 승인 방법:
  - `openclaw pairing list imessage`
  - `openclaw pairing approve imessage <CODE>`
- 페어링은 iMessage DM의 기본 토큰 교환입니다. 자세한 내용: [페어링](/start/pairing)

그룹:

- `channels.imessage.groupPolicy = open | allowlist | disabled`.
- `allowlist`로 설정하면 `channels.imessage.groupAllowFrom`이 그룹에서 트리거할 수 있는 사람을 제어합니다.
- 멘션 게이트는 iMessage에 네이티브 멘션 메타데이터가 없으므로 `agents.list[].groupChat.mentionPatterns`(또는 `messages.groupChat.mentionPatterns`)를 사용합니다.
- 다중 에이전트 재정의: `agents.list[].groupChat.mentionPatterns`에 에이전트별 패턴 설정.

## 작동 방식(동작)

- `imsg`가 메시지 이벤트를 스트리밍하고, Gateway가 이를 공유 채널 엔벨로프로 정규화합니다.
- 응답은 항상 동일한 chat id 또는 핸들로 라우팅됩니다.

## 그룹과 유사한 스레드(`is_group=false`)

일부 iMessage 스레드는 여러 참여자가 있지만 메시지 앱이 채팅 식별자를 저장하는 방식 때문에 `is_group=false`로 도착할 수 있습니다.

`channels.imessage.groups` 아래에 `chat_id`를 명시적으로 구성하면 OpenClaw는 해당 스레드를 다음 목적으로 "그룹"으로 처리합니다:

- 세션 격리(별도의 `agent:<agentId>:imessage:group:<chat_id>` 세션 키)
- 그룹 허용 목록/멘션 게이트 동작

예제:

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { requireMention: false },
      },
    },
  },
}
```

특정 스레드에 격리된 성격/모델을 사용하려는 경우 유용합니다([다중 에이전트 라우팅](/concepts/multi-agent) 참조). 파일 시스템 격리는 [샌드박싱](/gateway/sandboxing) 참조.

## 미디어 + 제한

- `channels.imessage.includeAttachments`를 통해 첨부 파일 수신 선택 가능.
- 미디어 제한은 `channels.imessage.mediaMaxMb`로 설정.

## 제한

- 아웃바운드 텍스트는 `channels.imessage.textChunkLimit`로 청크 분할(기본값 4000).
- 선택적 줄바꿈 청크: `channels.imessage.chunkMode="newline"`을 설정하면 길이별 청크 전에 빈 줄(단락 경계)로 분할.
- 미디어 업로드 제한: `channels.imessage.mediaMaxMb`(기본값 16).

## 주소 지정 / 배달 대상

안정적인 라우팅을 위해 `chat_id` 사용 권장:

- `chat_id:123`(권장)
- `chat_guid:...`
- `chat_identifier:...`
- 직접 핸들: `imessage:+1555` / `sms:+1555` / `user@example.com`

채팅 목록:

```
imsg chats --limit 20
```

## 구성 참조(iMessage)

전체 구성: [구성](/gateway/configuration)

제공자 옵션:

- `channels.imessage.enabled`: 채널 시작 활성화/비활성화.
- `channels.imessage.cliPath`: `imsg` 경로.
- `channels.imessage.dbPath`: Messages 데이터베이스 경로.
- `channels.imessage.remoteHost`: `cliPath`가 원격 Mac을 가리킬 때 SCP 첨부 파일 전송을 위한 SSH 호스트(예: `user@gateway-host`). 설정되지 않으면 SSH 래퍼 스크립트에서 자동 감지.
- `channels.imessage.service`: `imessage | sms | auto`.
- `channels.imessage.region`: SMS 지역.
- `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled`(기본값: pairing).
- `channels.imessage.allowFrom`: DM 허용 목록(핸들, 이메일, E.164 번호 또는 `chat_id:*`). `open`은 `"*"` 필요. iMessage에는 사용자 이름이 없으므로 핸들 또는 채팅 대상 사용.
- `channels.imessage.groupPolicy`: `open | allowlist | disabled`(기본값: allowlist).
- `channels.imessage.groupAllowFrom`: 그룹 발신자 허용 목록.
- `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: 컨텍스트로 포함할 최대 그룹 메시지 수(0이면 비활성화).
- `channels.imessage.dmHistoryLimit`: DM 기록 제한(사용자 턴 수). 사용자별 재정의: `channels.imessage.dms["<handle>"].historyLimit`.
- `channels.imessage.groups`: 그룹별 기본값 + 허용 목록(`"*"`로 전역 기본값 설정).
- `channels.imessage.includeAttachments`: 컨텍스트에 첨부 파일 수신.
- `channels.imessage.mediaMaxMb`: 인바운드/아웃바운드 미디어 제한(MB).
- `channels.imessage.textChunkLimit`: 아웃바운드 청크 크기(문자).
- `channels.imessage.chunkMode`: `length`(기본값) 또는 `newline`, 길이별 청크 전에 빈 줄(단락 경계)로 분할.

관련 전역 옵션:

- `agents.list[].groupChat.mentionPatterns`(또는 `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
