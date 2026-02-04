---
read_when:
  - BlueBubbles 채널 설정 시
  - webhook 페어링 문제 해결 시
  - macOS에서 iMessage 구성 시
summary: BlueBubbles macOS 서버를 통한 iMessage 통합(REST 송수신, 타이핑 상태, 반응, 페어링, 고급 작업).
title: BlueBubbles
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# BlueBubbles(macOS REST)

상태: 내장 플러그인으로, BlueBubbles macOS 서버와 HTTP를 통해 통신합니다. 기존 imsg 채널보다 API가 풍부하고 설정이 쉬워 **iMessage 통합에 권장**됩니다.

## 개요

- BlueBubbles 헬퍼 앱을 통해 macOS에서 실행([bluebubbles.app](https://bluebubbles.app)).
- 권장/테스트 완료: macOS Sequoia(15). macOS Tahoe(26)에서 작동하지만, 현재 Tahoe에서 편집 기능은 사용할 수 없으며 그룹 아이콘 업데이트는 성공을 보고하지만 동기화되지 않을 수 있습니다.
- OpenClaw는 REST API를 통해 통신(`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
- 수신 메시지는 webhook을 통해 도착하며, 발신 응답, 타이핑 표시기, 읽음 확인, tapback 반응은 모두 REST 호출입니다.
- 첨부 파일과 스티커는 인바운드 미디어로 수신됩니다(가능한 경우 에이전트에게 표시).
- 페어링/허용 목록은 다른 채널과 동일하게 작동(`/start/pairing` 등)하며, `channels.bluebubbles.allowFrom` + 페어링 코드를 사용합니다.
- 반응은 Slack/Telegram과 동일하게 시스템 이벤트로 표시되므로 에이전트가 응답 전에 "언급"할 수 있습니다.
- 고급 기능: 편집, 철회, 스레드 답장, 메시지 효과, 그룹 관리.

## 빠른 시작

1. Mac에 BlueBubbles 서버 설치([bluebubbles.app/install](https://bluebubbles.app/install) 지침 따르기).
2. BlueBubbles 구성에서 Web API를 활성화하고 비밀번호 설정.
3. `openclaw onboard`를 실행하고 BlueBubbles를 선택하거나 수동으로 구성:
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook",
       },
     },
   }
   ```
4. BlueBubbles webhook을 Gateway를 가리키도록 설정(예: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Gateway 시작; webhook 핸들러가 등록되고 페어링이 시작됩니다.

## 온보딩

BlueBubbles는 대화형 설정 마법사에서 사용 가능합니다:

```
openclaw onboard
```

마법사는 다음을 입력하라고 안내합니다:

- **서버 URL**(필수): BlueBubbles 서버 주소(예: `http://192.168.1.100:1234`)
- **비밀번호**(필수): BlueBubbles 서버 설정의 API 비밀번호
- **Webhook 경로**(선택): 기본값 `/bluebubbles-webhook`
- **DM 정책**: 페어링, 허용 목록, 개방 또는 비활성화
- **허용 목록**: 전화번호, 이메일 또는 채팅 대상

CLI를 통해 BlueBubbles 추가도 가능:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## 접근 제어(DM + 그룹)

DM:

- 기본값: `channels.bluebubbles.dmPolicy = "pairing"`.
- 알 수 없는 발신자는 페어링 코드를 받으며, 승인 전까지 메시지는 무시됩니다(페어링 코드는 1시간 후 만료).
- 승인 방법:
  - `openclaw pairing list bluebubbles`
  - `openclaw pairing approve bluebubbles <CODE>`
- 페어링은 기본 토큰 교환입니다. 자세한 내용: [페어링](/start/pairing)

그룹:

- `channels.bluebubbles.groupPolicy = open | allowlist | disabled`(기본값: `allowlist`).
- `allowlist`로 설정하면 `channels.bluebubbles.groupAllowFrom`이 그룹에서 트리거할 수 있는 사람을 제어합니다.

### 멘션 게이트(그룹)

BlueBubbles는 iMessage/WhatsApp 동작과 일치하는 그룹 채팅용 멘션 게이트를 지원합니다:

- `agents.list[].groupChat.mentionPatterns`(또는 `messages.groupChat.mentionPatterns`)를 사용하여 멘션 감지.
- 그룹에서 `requireMention`이 활성화되면 에이전트는 멘션될 때만 응답합니다.
- 승인된 발신자의 제어 명령은 멘션 게이트를 우회합니다.

그룹별 구성:

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // 모든 그룹의 기본값
        "iMessage;-;chat123": { requireMention: false }, // 특정 그룹 재정의
      },
    },
  },
}
```

### 명령 게이트

- 제어 명령(예: `/config`, `/model`)은 승인이 필요합니다.
- `allowFrom`과 `groupAllowFrom`을 사용하여 명령 승인을 결정합니다.
- 승인된 발신자는 그룹에서 멘션되지 않아도 제어 명령을 실행할 수 있습니다.

## 타이핑 상태 + 읽음 확인

- **타이핑 표시기**: 응답 생성 전과 생성 중에 자동 전송됩니다.
- **읽음 확인**: `channels.bluebubbles.sendReadReceipts`로 제어됩니다(기본값: `true`).
- **타이핑 표시기**: OpenClaw가 타이핑 시작 이벤트를 전송하며, BlueBubbles는 전송 또는 타임아웃 시 자동으로 타이핑 상태를 지웁니다(DELETE를 통한 수동 중지는 불안정).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // 읽음 확인 비활성화
    },
  },
}
```

## 고급 작업

구성에서 활성화하면 BlueBubbles는 고급 메시지 작업을 지원합니다:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapback 반응(기본값: true)
        edit: true, // 보낸 메시지 편집(macOS 13+, macOS 26 Tahoe에서는 사용 불가)
        unsend: true, // 메시지 철회(macOS 13+)
        reply: true, // 메시지 GUID로 스레드 답장
        sendWithEffect: true, // 메시지 효과(slam, loud 등)
        renameGroup: true, // 그룹 채팅 이름 변경
        setGroupIcon: true, // 그룹 채팅 아이콘/아바타 설정(macOS 26 Tahoe에서 불안정)
        addParticipant: true, // 그룹에 참여자 추가
        removeParticipant: true, // 그룹에서 참여자 제거
        leaveGroup: true, // 그룹 채팅 나가기
        sendAttachment: true, // 첨부 파일/미디어 전송
      },
    },
  },
}
```

사용 가능한 작업:

- **react**: tapback 반응 추가/제거(`messageId`, `emoji`, `remove`)
- **edit**: 보낸 메시지 편집(`messageId`, `text`)
- **unsend**: 메시지 철회(`messageId`)
- **reply**: 특정 메시지에 답장(`messageId`, `text`, `to`)
- **sendWithEffect**: iMessage 효과와 함께 전송(`text`, `to`, `effectId`)
- **renameGroup**: 그룹 채팅 이름 변경(`chatGuid`, `displayName`)
- **setGroupIcon**: 그룹 채팅 아이콘/아바타 설정(`chatGuid`, `media`)—macOS 26 Tahoe에서 불안정(API가 성공을 반환하지만 아이콘이 동기화되지 않을 수 있음).
- **addParticipant**: 그룹에 멤버 추가(`chatGuid`, `address`)
- **removeParticipant**: 그룹에서 멤버 제거(`chatGuid`, `address`)
- **leaveGroup**: 그룹 채팅 나가기(`chatGuid`)
- **sendAttachment**: 미디어/파일 전송(`to`, `buffer`, `filename`, `asVoice`)
  - 음성 메모: iMessage 음성 메시지로 전송하려면 `asVoice: true`를 설정하고 **MP3** 또는 **CAF** 오디오를 사용하세요. BlueBubbles는 음성 메모 전송 시 MP3를 CAF로 변환합니다.

### 메시지 ID(짧은 형식 vs 전체 형식)

OpenClaw는 토큰 절약을 위해 *짧은* 메시지 ID(예: `1`, `2`)를 표시할 수 있습니다.

- `MessageSid` / `ReplyToId`는 짧은 ID일 수 있습니다.
- `MessageSidFull` / `ReplyToIdFull`은 제공자의 전체 ID를 포함합니다.
- 짧은 ID는 메모리에 저장되며, 재시작 또는 캐시 정리 후 만료될 수 있습니다.
- 작업은 짧은 형식 또는 전체 형식의 `messageId`를 모두 허용하지만, 짧은 ID가 더 이상 사용 불가능하면 오류가 발생합니다.

영구 자동화 및 저장에는 전체 ID를 사용하세요:

- 템플릿: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
- 컨텍스트: 인바운드 페이로드의 `MessageSidFull` / `ReplyToIdFull`

템플릿 변수는 [구성](/gateway/configuration)을 참조하세요.

## 청크 스트리밍

응답을 단일 메시지로 보낼지 청크 스트리밍할지 제어:

```json5
{
  channels: {
    bluebubbles: {
      chunkedStreaming: true, // 청크 스트리밍 활성화(기본값: false)
    },
  },
}
```

## 미디어 + 제한

- 미디어 다운로드 제한: `channels.bluebubbles.mediaMaxMb`(기본값: 10MB).
- 지원되는 미디어 유형: 이미지, 오디오, 비디오, 파일.

## 문제 해결

- **Webhook이 연결되지 않음:** BlueBubbles 서버 URL과 비밀번호 확인. Gateway 로그에서 webhook 등록 확인.
- **메시지가 수신되지 않음:** BlueBubbles의 webhook 설정이 올바른 URL을 가리키는지 확인. 네트워크 연결 확인.
- **첨부 파일이 작동하지 않음:** `mediaMaxMb` 제한 확인. BlueBubbles 서버가 첨부 파일에 접근할 수 있는지 확인.
- **타이핑 표시기가 지워지지 않음:** BlueBubbles가 타임아웃 시 자동으로 지웁니다. 수동 중지는 불안정합니다.

## 구성 참조

전체 구성: [구성](/gateway/configuration)

주요 옵션:

- `channels.bluebubbles.enabled`: 채널 활성화/비활성화.
- `channels.bluebubbles.serverUrl`: BlueBubbles 서버 URL.
- `channels.bluebubbles.password`: API 비밀번호.
- `channels.bluebubbles.webhookPath`: Webhook 경로(기본값: `/bluebubbles-webhook`).
- `channels.bluebubbles.dmPolicy`: DM 정책(`pairing` | `allowlist` | `open` | `disabled`).
- `channels.bluebubbles.allowFrom`: DM 허용 목록.
- `channels.bluebubbles.groupPolicy`: 그룹 정책.
- `channels.bluebubbles.groupAllowFrom`: 그룹 허용 목록.
- `channels.bluebubbles.sendReadReceipts`: 읽음 확인 전송 여부.
- `channels.bluebubbles.actions.*`: 고급 작업 활성화.
