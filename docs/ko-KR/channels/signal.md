---
read_when:
  - Signal 지원 설정 시
  - Signal 송수신 디버깅 시
summary: signal-cli(JSON-RPC + SSE)를 통한 Signal 지원, 설정 및 번호 모델
title: Signal
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: ca4de8b3685017f54a959e3e2699357ab40b3e4e68574bd7fb5739e4679e7d8a
  source_path: channels/signal.md
  workflow: 14
---

# Signal(signal-cli)

상태: 외부 CLI 통합. Gateway는 HTTP JSON-RPC + SSE를 통해 `signal-cli`와 통신합니다.

## 빠른 설정(초보자용)

1. 봇용 **별도 Signal 번호** 사용(권장).
2. `signal-cli` 설치(Java 필요).
3. 봇 장치를 연결하고 데몬 시작:
   - `signal-cli link -n "OpenClaw"`
4. OpenClaw를 설정하고 Gateway 시작.

최소 설정:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## 개요

- `signal-cli`를 통한 Signal 채널(임베디드 libsignal 아님).
- 확정적 라우팅: 응답은 항상 Signal로 돌아감.
- DM은 에이전트의 메인 세션을 공유; 그룹은 격리됨(`agent:<agentId>:signal:group:<groupId>`).

## 설정 쓰기

기본적으로 Signal은 `/config set|unset`으로 트리거되는 설정 업데이트 쓰기를 허용합니다(`commands.config: true` 필요).

비활성화 방법:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## 번호 모델(중요)

- Gateway는 하나의 **Signal 장치**(`signal-cli` 계정)에 연결합니다.
- **개인 Signal 계정**에서 봇을 실행하면 자신의 메시지를 무시합니다(루프 보호).
- "내가 봇에게 메시지를 보내면 나에게 응답"을 원하면 **별도 봇 번호**를 사용하세요.

## 설정(빠른 경로)

1. `signal-cli` 설치(Java 필요).
2. 봇 계정 연결:
   - `signal-cli link -n "OpenClaw"` 후 Signal에서 QR 코드 스캔.
3. Signal을 설정하고 Gateway 시작.

예시:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

다중 계정: `channels.signal.accounts`를 사용하여 각 계정에 독립적인 옵션과 선택적 `name`을 설정하세요. 공유 패턴은 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 참조.

## 외부 데몬 모드(httpUrl)

`signal-cli`를 직접 관리하려면(JVM 콜드 스타트 느림, 컨테이너 초기화 또는 공유 CPU) 데몬을 별도로 실행하고 OpenClaw가 이를 가리키도록 설정:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

이렇게 하면 OpenClaw 내부의 자동 시작 및 시작 대기를 건너뜁니다. 자동 시작이 느린 경우 `channels.signal.startupTimeoutMs`를 설정하세요.

## 접근 제어(DM + 그룹)

DM:

- 기본값: `channels.signal.dmPolicy = "pairing"`.
- 알 수 없는 발신자는 페어링 코드를 받음; 승인 전까지 메시지 무시(페어링 코드 1시간 후 만료).
- 승인 방법:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- 페어링은 Signal DM의 기본 토큰 교환입니다. 자세한 내용: [페어링](/start/pairing)
- UUID 전용 발신자(`sourceUuid` 기반)는 `channels.signal.allowFrom`에 `uuid:<id>` 형식으로 저장됩니다.

그룹:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `allowlist`로 설정하면 `channels.signal.groupAllowFrom`이 그룹에서 트리거할 수 있는 사람을 제어합니다.

## 작동 방식(동작)

- `signal-cli`가 데몬으로 실행됨; Gateway는 SSE를 통해 이벤트를 읽음.
- 인바운드 메시지는 공유 채널 엔벨로프로 정규화됨.
- 응답은 항상 동일한 번호 또는 그룹으로 라우팅됨.

## 미디어 + 제한

- 아웃바운드 텍스트는 `channels.signal.textChunkLimit`로 분할됨(기본값 4000).
- 선택적 줄바꿈 분할: `channels.signal.chunkMode="newline"`으로 설정하면 길이로 분할하기 전에 빈 줄(단락 경계)에서 분할.
- 첨부 파일 지원(`signal-cli`에서 base64 가져오기).
- 기본 미디어 한도: `channels.signal.mediaMaxMb`(기본값 8).
- `channels.signal.ignoreAttachments`를 사용하여 미디어 다운로드 건너뛰기.
- 그룹 히스토리 컨텍스트는 `channels.signal.historyLimit`(또는 `channels.signal.accounts.*.historyLimit`)를 사용하며, `messages.groupChat.historyLimit`로 폴백. `0`으로 설정하여 비활성화(기본값 50).

## 입력 표시기 + 읽음 확인

- **입력 표시기**: OpenClaw는 `signal-cli sendTyping`을 통해 입력 신호를 보내고 응답 실행 중 새로 고침.
- **읽음 확인**: `channels.signal.sendReadReceipts`가 true일 때 OpenClaw는 허용된 DM에 대해 읽음 확인을 전달.
- signal-cli는 그룹의 읽음 확인을 노출하지 않음.

## 반응(message 도구)

- `message action=react`를 `channel=signal`과 함께 사용.
- 대상: 발신자 E.164 또는 UUID(페어링 출력의 `uuid:<id>` 사용; 순수 UUID도 가능).
- `messageId`는 반응할 메시지의 Signal 타임스탬프.
- 그룹 반응에는 `targetAuthor` 또는 `targetAuthorUuid` 필요.

예시:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

설정:

- `channels.signal.actions.reactions`: 반응 작업 활성화/비활성화(기본값 true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack`는 에이전트 반응 비활성화(message 도구 `react`가 오류 발생).
  - `minimal`/`extensive`는 에이전트 반응을 활성화하고 안내 수준 설정.
- 계정별 재정의: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## 전달 대상(CLI/크론)

- DM: `signal:+15551234567`(또는 순수 E.164).
- UUID DM: `uuid:<id>`(또는 순수 UUID).
- 그룹: `signal:group:<groupId>`.
- 사용자 이름: `username:<name>`(Signal 계정이 지원하는 경우).

## 설정 참조(Signal)

전체 설정: [설정](/gateway/configuration)

제공자 옵션:

- `channels.signal.enabled`: 채널 시작 활성화/비활성화.
- `channels.signal.account`: 봇 계정의 E.164.
- `channels.signal.cliPath`: `signal-cli` 경로.
- `channels.signal.httpUrl`: 전체 데몬 URL(host/port 재정의).
- `channels.signal.httpHost`, `channels.signal.httpPort`: 데몬 바인딩(기본값 127.0.0.1:8080).
- `channels.signal.autoStart`: 데몬 자동 시작(`httpUrl`이 설정되지 않은 경우 기본값 true).
- `channels.signal.startupTimeoutMs`: 시작 대기 타임아웃(밀리초, 상한 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: 첨부 파일 다운로드 건너뛰기.
- `channels.signal.ignoreStories`: 데몬에서 오는 스토리 무시.
- `channels.signal.sendReadReceipts`: 읽음 확인 전달.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled`(기본값: pairing).
- `channels.signal.allowFrom`: DM 허용 목록(E.164 또는 `uuid:<id>`). `open`은 `"*"` 필요. Signal은 사용자 이름이 없음; 전화/UUID 식별자 사용.
- `channels.signal.groupPolicy`: `open | allowlist | disabled`(기본값: allowlist).
- `channels.signal.groupAllowFrom`: 그룹 발신자 허용 목록.
- `channels.signal.historyLimit`: 컨텍스트로 포함할 최대 그룹 메시지 수(0은 비활성화).
- `channels.signal.dmHistoryLimit`: DM 히스토리 제한(사용자 턴 수). 사용자별 재정의: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: 아웃바운드 청크 크기(문자).
- `channels.signal.chunkMode`: `length`(기본값) 또는 `newline`, 길이로 분할하기 전에 빈 줄(단락 경계)에서 분할.
- `channels.signal.mediaMaxMb`: 인바운드/아웃바운드 미디어 한도(MB).

관련 전역 옵션:

- `agents.list[].groupChat.mentionPatterns`(Signal은 네이티브 멘션을 지원하지 않음).
- `messages.groupChat.mentionPatterns`(전역 폴백).
- `messages.responsePrefix`.
