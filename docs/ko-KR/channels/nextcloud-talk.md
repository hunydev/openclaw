---
read_when:
  - Nextcloud Talk 채널 기능 개발 시
summary: Nextcloud Talk 지원 상태, 기능 및 설정
title: Nextcloud Talk
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 21b7b9756c4356a76dc0f14c10e44ed74a284cf3badf87e2df75eb88d8a90c31
  source_path: channels/nextcloud-talk.md
  workflow: 14
---

# Nextcloud Talk(플러그인)

상태: 플러그인을 통해 지원(webhook 봇). DM, 방, 반응 및 markdown 메시지 지원.

## 플러그인 필요

Nextcloud Talk는 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

CLI를 통해 설치(npm 레지스트리):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

설정/온보딩 중 Nextcloud Talk를 선택하고 git 체크아웃이 감지되면 OpenClaw가 자동으로 로컬 설치 경로를 제안합니다.

자세한 내용: [플러그인](/plugin)

## 빠른 설정(초보자용)

1. Nextcloud Talk 플러그인 설치.
2. Nextcloud 서버에서 봇 생성:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. 대상 방 설정에서 봇 활성화.
4. OpenClaw 설정:
   - 설정: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   - 또는 환경 변수: `NEXTCLOUD_TALK_BOT_SECRET`(기본 계정만)
5. Gateway 재시작(또는 온보딩 완료).

최소 설정:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## 참고

- 봇은 DM을 먼저 시작할 수 없습니다. 사용자가 먼저 봇에게 메시지를 보내야 합니다.
- Webhook URL은 Gateway에서 접근 가능해야 합니다; 프록시 뒤에 있으면 `webhookPublicUrl`을 설정하세요.
- 봇 API는 미디어 업로드를 지원하지 않습니다; 미디어는 URL로 전송됩니다.
- Webhook 페이로드는 DM과 방을 구분하지 않습니다; `apiUser` + `apiPassword`를 설정하여 방 타입 조회를 활성화하세요(그렇지 않으면 DM이 방으로 취급됩니다).

## 접근 제어(DM)

- 기본값: `channels.nextcloud-talk.dmPolicy = "pairing"`. 알 수 없는 발신자는 페어링 코드를 받습니다.
- 승인 방법:
  - `openclaw pairing list nextcloud-talk`
  - `openclaw pairing approve nextcloud-talk <CODE>`
- 공개 DM: `channels.nextcloud-talk.dmPolicy="open"` + `channels.nextcloud-talk.allowFrom=["*"]`.

## 방(그룹)

- 기본값: `channels.nextcloud-talk.groupPolicy = "allowlist"`(멘션 게이팅).
- `channels.nextcloud-talk.rooms`를 사용하여 방 허용 목록:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

- 모든 방을 허용하지 않으려면 허용 목록을 비워두거나 `channels.nextcloud-talk.groupPolicy="disabled"` 설정.

## 기능

| 기능          | 상태      |
| ------------- | --------- |
| DM            | 지원됨    |
| 방            | 지원됨    |
| 스레드        | 미지원    |
| 미디어        | URL만     |
| 반응          | 지원됨    |
| 네이티브 명령 | 미지원    |

## 설정 참조(Nextcloud Talk)

전체 설정: [설정](/gateway/configuration)

제공자 옵션:

- `channels.nextcloud-talk.enabled`: 채널 시작 활성화/비활성화.
- `channels.nextcloud-talk.baseUrl`: Nextcloud 인스턴스 URL.
- `channels.nextcloud-talk.botSecret`: 봇 공유 시크릿.
- `channels.nextcloud-talk.botSecretFile`: 시크릿 파일 경로.
- `channels.nextcloud-talk.apiUser`: 방 조회용 API 사용자(DM 감지).
- `channels.nextcloud-talk.apiPassword`: 방 조회용 API/앱 비밀번호.
- `channels.nextcloud-talk.apiPasswordFile`: API 비밀번호 파일 경로.
- `channels.nextcloud-talk.webhookPort`: webhook 수신 포트(기본값: 8788).
- `channels.nextcloud-talk.webhookHost`: webhook 호스트(기본값: 0.0.0.0).
- `channels.nextcloud-talk.webhookPath`: webhook 경로(기본값: /nextcloud-talk-webhook).
- `channels.nextcloud-talk.webhookPublicUrl`: 외부 접근 가능한 webhook URL.
- `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
- `channels.nextcloud-talk.allowFrom`: DM 허용 목록(사용자 ID). `open`은 `"*"` 필요.
- `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
- `channels.nextcloud-talk.groupAllowFrom`: 그룹 허용 목록(사용자 ID).
- `channels.nextcloud-talk.rooms`: 방별 설정 및 허용 목록.
- `channels.nextcloud-talk.historyLimit`: 그룹 히스토리 제한(0은 비활성화).
- `channels.nextcloud-talk.dmHistoryLimit`: DM 히스토리 제한(0은 비활성화).
- `channels.nextcloud-talk.dms`: DM별 재정의(historyLimit).
- `channels.nextcloud-talk.textChunkLimit`: 아웃바운드 텍스트 청크 크기(문자).
- `channels.nextcloud-talk.chunkMode`: `length`(기본값) 또는 `newline`, 길이로 분할하기 전에 빈 줄(단락 경계)에서 분할.
- `channels.nextcloud-talk.blockStreaming`: 이 채널에 대한 청크 스트리밍 비활성화.
- `channels.nextcloud-talk.blockStreamingCoalesce`: 청크 스트리밍 병합 튜닝.
- `channels.nextcloud-talk.mediaMaxMb`: 인바운드 미디어 한도(MB).
