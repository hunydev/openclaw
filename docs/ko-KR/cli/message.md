---
read_when:
  - 메시지 CLI 작업을 추가하거나 수정할 때
  - 아웃바운드 채널 동작을 변경할 때
summary: "`openclaw message`(전송 + 채널 작업)의 CLI 레퍼런스"
title: message
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw message`

메시지 전송 및 채널 작업을 위한 단일 아웃바운드 명령
(Discord/Google Chat/Slack/Mattermost(플러그인)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## 사용법

```
openclaw message <subcommand> [flags]
```

채널 선택:

- 여러 채널이 구성된 경우 `--channel`을 지정해야 합니다.
- 하나의 채널만 구성된 경우 해당 채널이 기본값입니다.
- 사용 가능한 값: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`(Mattermost는 플러그인 필요)

대상 형식(`--target`):

- WhatsApp: E.164 또는 그룹 JID
- Telegram: 채팅 ID 또는 `@username`
- Discord: `channel:<id>` 또는 `user:<id>`(또는 `<@id>` 멘션; 순수 숫자 ID는 채널로 처리)
- Google Chat: `spaces/<spaceId>` 또는 `users/<userId>`
- Slack: `channel:<id>` 또는 `user:<id>`(순수 채널 ID 허용)
- Mattermost(플러그인): `channel:<id>`, `user:<id>` 또는 `@username`(순수 ID는 채널로 처리)
- Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>` 또는 `username:<name>`/`u:<name>`
- iMessage: 핸들, `chat_id:<id>`, `chat_guid:<guid>` 또는 `chat_identifier:<id>`
- MS Teams: 대화 ID(`19:...@thread.tacv2`) 또는 `conversation:<id>` 또는 `user:<aad-object-id>`

이름 조회:

- 지원되는 제공자(Discord/Slack 등)의 경우 `Help` 또는 `#help`와 같은 채널 이름이 디렉토리 캐시를 통해 해석됩니다.
- 캐시 미스 시 제공자가 지원하면 OpenClaw가 실시간 디렉토리 조회를 시도합니다.

## 공통 플래그

- `--channel <name>`
- `--account <id>`
- `--target <dest>`(send/poll/read 등의 대상 채널 또는 사용자)
- `--targets <name>`(반복 가능; 브로드캐스트 전용)
- `--json`
- `--dry-run`
- `--verbose`

## 작업

### 핵심

- `send`
  - 채널: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost(플러그인)/Signal/iMessage/MS Teams
  - 필수: `--target`, `--message` 또는 `--media`
  - 선택: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Telegram 전용: `--buttons`(활성화하려면 `channels.telegram.capabilities.inlineButtons` 필요)
  - Telegram 전용: `--thread-id`(포럼 주제 ID)
  - Slack 전용: `--thread-id`(스레드 타임스탬프; `--reply-to`도 동일 필드 사용)
  - WhatsApp 전용: `--gif-playback`

- `poll`
  - 채널: WhatsApp/Discord/MS Teams
  - 필수: `--target`, `--poll-question`, `--poll-option`(반복 가능)
  - 선택: `--poll-multi`
  - Discord 전용: `--poll-duration-hours`, `--message`

- `react`
  - 채널: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - 필수: `--message-id`, `--target`
  - 선택: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - 참고: `--remove`는 `--emoji`가 필요합니다(`--emoji`를 생략하면 지원되는 경우 자신의 이모지 반응을 지울 수 있음; /tools/reactions 참조)
  - WhatsApp 전용: `--participant`, `--from-me`
  - Signal 그룹 이모지 반응: `--target-author` 또는 `--target-author-uuid` 필요

- `reactions`
  - 채널: Discord/Google Chat/Slack
  - 필수: `--message-id`, `--target`
  - 선택: `--limit`

- `read`
  - 채널: Discord/Slack
  - 필수: `--target`
  - 선택: `--limit`, `--before`, `--after`
  - Discord 전용: `--around`

- `edit`
  - 채널: Discord/Slack
  - 필수: `--message-id`, `--message`, `--target`

- `delete`
  - 채널: Discord/Slack/Telegram
  - 필수: `--message-id`, `--target`

- `pin` / `unpin`
  - 채널: Discord/Slack
  - 필수: `--message-id`, `--target`

- `pins`(목록)
  - 채널: Discord/Slack
  - 필수: `--target`

- `permissions`
  - 채널: Discord
  - 필수: `--target`

- `search`
  - 채널: Discord
  - 필수: `--guild-id`, `--query`
  - 선택: `--channel-id`, `--channel-ids`(반복 가능), `--author-id`, `--author-ids`(반복 가능), `--limit`

### 스레드

- `thread create`
  - 채널: Discord
  - 필수: `--thread-name`, `--target`(채널 ID)
  - 선택: `--message-id`, `--auto-archive-min`

- `thread list`
  - 채널: Discord
  - 필수: `--guild-id`
  - 선택: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - 채널: Discord
  - 필수: `--target`(스레드 ID), `--message`
  - 선택: `--media`, `--reply-to`

### 이모지

- `emoji list`
  - Discord: `--guild-id`
  - Slack: 추가 플래그 불필요

- `emoji upload`
  - 채널: Discord
  - 필수: `--guild-id`, `--emoji-name`, `--media`
  - 선택: `--role-ids`(반복 가능)

### 스티커

- `sticker send`
  - 채널: Discord
  - 필수: `--target`, `--sticker-id`(반복 가능)
  - 선택: `--message`

- `sticker upload`
  - 채널: Discord
  - 필수: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### 역할 / 채널 / 멤버 / 음성

- `role info`(Discord): `--guild-id`
- `role add` / `role remove`(Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info`(Discord): `--target`
- `channel list`(Discord): `--guild-id`
- `member info`(Discord/Slack): `--user-id`(Discord는 `--guild-id`도 필요)
- `voice status`(Discord): `--guild-id`, `--user-id`

### 이벤트

- `event list`(Discord): `--guild-id`
- `event create`(Discord): `--guild-id`, `--event-name`, `--start-time`
  - 선택: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### 관리(Discord)

- `timeout`: `--guild-id`, `--user-id`(선택적 `--duration-min` 또는 `--until`; 둘 다 생략하면 타임아웃 해제)
- `kick`: `--guild-id`, `--user-id`(+ `--reason`)
- `ban`: `--guild-id`, `--user-id`(+ `--delete-days`, `--reason`)
  - `timeout`도 `--reason` 지원

### 브로드캐스트

- `broadcast`
  - 채널: 구성된 모든 채널; 모든 제공자를 대상으로 하려면 `--channel all` 사용
  - 필수: `--targets`(반복 가능)
  - 선택: `--message`, `--media`, `--dry-run`

## 예시

Discord 답장 전송:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Discord 투표 만들기:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Teams 사전 메시지 전송:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Teams 투표 만들기:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Slack에서 이모지 반응 추가:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Signal 그룹에서 이모지 반응 추가:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram 인라인 버튼 전송:

```
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
