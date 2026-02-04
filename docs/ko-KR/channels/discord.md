---
read_when:
  - Discord 채널 기능 개발 시
summary: Discord 봇 지원 상태, 기능 및 구성
title: Discord
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Discord(Bot API)

상태: 공식 Discord 봇 Gateway를 통한 DM 및 서버 텍스트 채널에서 사용 가능합니다.

## 빠른 설정(초보자)

1. Discord 봇을 만들고 봇 토큰을 복사합니다.
2. Discord 앱 설정에서 **Message Content Intent**를 활성화합니다(허용 목록이나 이름 조회를 사용하려면 **Server Members Intent**도 활성화).
3. OpenClaw에 토큰을 설정합니다:
   - 환경 변수: `DISCORD_BOT_TOKEN=...`
   - 또는 구성: `channels.discord.token: "..."`
   - 둘 다 설정되면 구성이 우선합니다(환경 변수 폴백은 기본 계정에만 적용).
4. 봇을 서버에 초대하고 메시지 권한을 부여합니다(DM만 사용하려면 비공개 서버를 만들 수 있습니다).
5. Gateway를 시작합니다.
6. DM 접근은 기본적으로 페어링이 필요하며, 첫 연락 시 페어링 코드를 승인하세요.

최소 구성:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

## 목표

- Discord DM 또는 서버 채널을 통해 OpenClaw와 대화합니다.
- DM은 에이전트의 기본 세션(기본값 `agent:main:main`)에 병합되고, 서버 채널은 `agent:<agentId>:discord:channel:<channelId>`로 격리됩니다(표시 이름은 `discord:<guildSlug>#<channelSlug>` 사용).
- 그룹 DM은 기본적으로 무시됩니다. `channels.discord.dm.groupEnabled`로 활성화하고, 선택적으로 `channels.discord.dm.groupChannels`로 제한할 수 있습니다.
- 결정론적 라우팅 유지: 응답은 항상 메시지가 도착한 채널로 돌아갑니다.

## 작동 방식

1. Discord 앱 → Bot을 만들고, 필요한 intent를 활성화하고(DM + 서버 메시지 + 메시지 콘텐츠), 봇 토큰을 얻습니다.
2. 봇을 서버에 초대하고, 필요한 곳에서 메시지를 읽고 보내는 데 필요한 권한을 부여합니다.
3. `channels.discord.token`으로 OpenClaw를 구성합니다(또는 `DISCORD_BOT_TOKEN`을 폴백으로 사용).
4. Gateway를 실행합니다. 토큰을 사용할 수 있고(구성 우선, 환경 변수 폴백) `channels.discord.enabled`가 `false`가 아니면 Discord 채널이 자동으로 시작됩니다.
   - 환경 변수를 선호하면 `DISCORD_BOT_TOKEN`을 설정하세요(구성 블록은 선택 사항).
5. DM: 배달에 `user:<id>`(또는 `<@id>` 멘션) 사용. 모든 턴은 공유 `main` 세션으로 이동합니다. 숫자 ID만 사용하면 모호하므로 거부됩니다.
6. 서버 채널: 배달에 `channel:<channelId>` 사용. 기본적으로 멘션이 필요하며, 서버별 또는 채널별로 설정할 수 있습니다.
7. DM: 기본적으로 `channels.discord.dm.policy`(기본값: `"pairing"`)로 보호됩니다. 알 수 없는 발신자는 페어링 코드를 받습니다(1시간 후 만료). `openclaw pairing approve discord <code>`로 승인합니다.
   - 기존의 "누구에게나 열림" 동작을 유지하려면: `channels.discord.dm.policy="open"`과 `channels.discord.dm.allowFrom=["*"]`를 설정합니다.
   - 하드 허용 목록을 원하면: `channels.discord.dm.policy="allowlist"`를 설정하고 `channels.discord.dm.allowFrom`에 발신자를 나열합니다.
   - 모든 DM을 무시하려면: `channels.discord.dm.enabled=false` 또는 `channels.discord.dm.policy="disabled"`를 설정합니다.
8. 그룹 DM은 기본적으로 무시됩니다. `channels.discord.dm.groupEnabled`로 활성화하고, 선택적으로 `channels.discord.dm.groupChannels`로 제한합니다.
9. 선택적 서버 규칙: 서버 ID(권장) 또는 slug를 키로 하는 `channels.discord.guilds`를 설정하고, 채널별 규칙을 포함합니다.
10. 선택적 네이티브 명령: `commands.native`의 기본값은 `"auto"`입니다(Discord/Telegram 켜짐, Slack 꺼짐). `channels.discord.commands.native: true|false|"auto"`로 재정의하세요. `false`는 이전에 등록된 명령을 지웁니다. 텍스트 명령은 `commands.text`로 제어되며, 독립적인 `/...` 메시지로 전송해야 합니다. `commands.useAccessGroups: false`를 사용하면 명령의 접근 그룹 검사를 우회합니다.
    - 전체 명령 목록 + 구성: [슬래시 명령](/tools/slash-commands)
11. 선택적 서버 컨텍스트 기록: `channels.discord.historyLimit`(기본값 20, `messages.groupChat.historyLimit`으로 폴백)를 설정하여 멘션에 응답할 때 최근 N개의 서버 메시지를 컨텍스트로 포함합니다. `0`으로 설정하면 비활성화됩니다.
12. 반응: 에이전트는 `discord` 도구를 통해 반응을 트리거할 수 있습니다(`channels.discord.actions.*`로 제어).
    - 반응 제거 의미: [/tools/reactions](/tools/reactions) 참조.
    - `discord` 도구는 현재 채널이 Discord일 때만 노출됩니다.
13. 네이티브 명령은 격리된 세션 키(`agent:<agentId>:discord:slash:<userId>`)를 사용하며 공유 `main` 세션이 아닙니다.

참고: 이름 → ID 해석은 서버 멤버 검색을 사용하며 Server Members Intent가 필요합니다. 봇이 멤버를 검색할 수 없으면 ID 또는 `<@id>` 멘션을 사용하세요.
참고: Slug는 소문자이며 공백은 `-`로 대체됩니다. 채널 이름 slug에는 앞의 `#`이 포함되지 않습니다.
참고: 서버 컨텍스트 `[from:]` 행에는 직접 핑 가능한 응답을 위해 `author.tag` + `id`가 포함됩니다.

## 구성 쓰기

기본적으로 Discord는 `/config set|unset`으로 트리거되는 구성 업데이트 쓰기를 허용합니다(`commands.config: true` 필요).

비활성화 방법:

```json5
{
  channels: { discord: { configWrites: false } },
}
```

## 자신만의 봇 만들기

서버(길드) 채널(예: `#help`)에서 OpenClaw를 실행하기 위한 "Discord 개발자 포털" 설정입니다.

### 1) Discord 앱 + 봇 사용자 만들기

1. Discord 개발자 포털 → **Applications** → **New Application**
2. 앱에서:
   - **Bot** → **Add Bot**
   - **Bot Token** 복사(이것이 `DISCORD_BOT_TOKEN`에 넣는 값입니다)

### 2) OpenClaw에 필요한 Gateway Intent 활성화

Discord는 명시적으로 활성화하지 않으면 "특권 intent"를 차단합니다.

**Bot** → **Privileged Gateway Intents**에서 활성화:

- **Message Content Intent**(대부분의 서버에서 메시지 텍스트를 읽는 데 필요함. 없으면 "Used disallowed intents" 오류가 발생하거나 봇이 연결되지만 메시지에 응답하지 않음)
- **Server Members Intent**(권장. 일부 멤버/사용자 조회 및 서버의 허용 목록 매칭에 필요)

일반적으로 **Presence Intent**는 필요하지 **않습니다**.

### 3) 초대 URL 생성(OAuth2 URL 생성기)

앱에서: **OAuth2** → **URL Generator**

**Scopes**

- ✅ `bot`
- ✅ `applications.commands`(네이티브 명령에 필요)

**Bot Permissions**(최소 기준)

- ✅ View Channels
- ✅ Send Messages
- ✅ Read Message History
- ✅ Embed Links
- ✅ Attach Files
- ✅ Add Reactions(선택 사항이지만 권장)
- ✅ Use External Emojis / Stickers(선택 사항. 필요할 때만)

디버깅 중이고 봇을 완전히 신뢰하지 않는 한 **Administrator**는 피하세요.

생성된 URL을 복사하고, 열어서, 서버를 선택하고, 봇을 설치합니다.

### 4) ID 얻기(서버/사용자/채널)

Discord는 어디서나 숫자 ID를 사용합니다. OpenClaw 구성은 ID를 권장합니다.

1. Discord(데스크톱/웹) → **사용자 설정** → **고급** → **개발자 모드** 활성화
2. 우클릭:
   - 서버 이름 → **서버 ID 복사**(guild id)
   - 채널(예: `#help`) → **채널 ID 복사**
   - 사용자 → **사용자 ID 복사**

### 5) OpenClaw 구성

#### 토큰

환경 변수로 봇 토큰 설정(서버에 권장):

- `DISCORD_BOT_TOKEN=...`

또는 구성으로:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

다중 계정 지원: `channels.discord.accounts`를 사용하고, 각 계정에 독립적인 토큰과 선택적 `name` 구성. 공유 모드는 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 참조.

#### 허용 목록 + 채널 라우팅

예제 "단일 서버, 나만 허용, #help만 허용":

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

참고:

- `requireMention: true`는 봇이 멘션될 때만 응답함을 의미합니다(공유 채널에 권장).
- `agents.list[].groupChat.mentionPatterns`(또는 `messages.groupChat.mentionPatterns`)도 서버 메시지에서 멘션으로 간주됩니다.
- 다중 에이전트 재정의: `agents.list[].groupChat.mentionPatterns`에 에이전트별 패턴 설정.
- `channels`가 있으면 목록에 없는 채널은 기본적으로 거부됩니다.
- `"*"` 채널 항목을 사용하여 모든 채널에 기본값 적용. 명시적 채널 항목이 와일드카드를 재정의합니다.
- 스레드는 부모 채널 구성(허용 목록, `requireMention`, Skills, 프롬프트 등)을 상속합니다. 스레드 채널 ID를 명시적으로 추가하지 않는 한.
- 봇이 보낸 메시지는 기본적으로 무시됩니다. `channels.discord.allowBots=true`를 설정하면 허용합니다(자신의 메시지는 여전히 필터링).
- 경고: 다른 봇에 응답을 허용하면(`channels.discord.allowBots=true`), `requireMention`, `channels.discord.guilds.*.channels.<id>.users` 허용 목록 및/또는 `AGENTS.md`와 `SOUL.md`의 명시적 가드레일 규칙을 사용하여 봇 간 응답 루프를 방지하세요.

### 6) 작동 확인

1. Gateway를 시작합니다.
2. 서버 채널에서 다음을 보냅니다: `@Krill hello`(또는 봇 이름).
3. 응답이 없으면 아래 **문제 해결**을 확인하세요.

### 문제 해결

- 먼저: `openclaw doctor`와 `openclaw channels status --probe`를 실행합니다(실행 가능한 경고 + 빠른 감사).
- **"Used disallowed intents"**: 개발자 포털에서 **Message Content Intent**(및 아마도 **Server Members Intent**)를 활성화한 후 Gateway를 다시 시작합니다.
- **봇이 연결되지만 서버 채널에서 응답하지 않음**:
  - **Message Content Intent** 누락, 또는
  - 봇에 채널 권한(View/Send/Read History) 부족, 또는
  - 구성이 멘션을 요구하지만 멘션하지 않음, 또는
  - 서버/채널 허용 목록이 해당 채널/사용자를 거부함.
- **`requireMention: false`인데도 응답 없음**:
- `channels.discord.groupPolicy`의 기본값은 **allowlist**입니다. `"open"`으로 설정하거나 `channels.discord.guilds` 아래에 서버 항목을 추가하세요(선택적으로 `channels.discord.guilds.<id>.channels` 아래에 채널을 나열하여 제한).
  - `DISCORD_BOT_TOKEN`만 설정하고 `channels.discord` 섹션을 만들지 않았다면 런타임이 `groupPolicy`를 `open`으로 기본 설정합니다. `channels.discord.groupPolicy`, `channels.defaults.groupPolicy` 또는 서버/채널 허용 목록을 추가하여 잠그세요.
- `requireMention`은 `channels.discord.guilds`(또는 특정 채널) 아래에 있어야 합니다. 최상위 `channels.discord.requireMention`은 무시됩니다.
- **권한 감사**(`channels status --probe`)는 숫자 채널 ID만 확인합니다. `channels.discord.guilds.*.channels` 키로 slug/이름을 사용하면 감사가 권한을 확인할 수 없습니다.
- **DM이 작동하지 않음**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, 또는 아직 승인되지 않음(`channels.discord.dm.policy="pairing"`).

## 기능 및 제한

- DM 및 서버 텍스트 채널(스레드는 별도 채널로 취급됨. 음성 미지원).
- 타이핑 표시기는 최선의 노력으로 전송됩니다. 메시지 청킹은 `channels.discord.textChunkLimit`(기본값 2000)을 사용하고 긴 응답은 줄 수로 분할합니다(`channels.discord.maxLinesPerMessage`, 기본값 17).
- 선택적 줄바꿈 청킹: `channels.discord.chunkMode="newline"`을 설정하면 길이별 청킹 전에 빈 줄(단락 경계)에서 분할.
- 파일 업로드 지원, 구성된 `channels.discord.mediaMaxMb`(기본값 8 MB)까지.
- 서버 응답은 기본적으로 멘션 게이팅이 필요하여 시끄러운 봇을 방지합니다.
- 메시지가 다른 메시지를 참조하면 응답 컨텍스트(인용 콘텐츠 + ID)가 주입됩니다.
- 네이티브 응답 스레드는 **기본적으로 꺼져 있습니다**. `channels.discord.replyToMode`와 응답 태그로 활성화하세요.

## 재시도 전략

아웃바운드 Discord API 호출은 레이트 리밋(429)에서 Discord의 `retry_after`(가능한 경우)를 사용하여 지수 백오프와 지터로 재시도합니다. `channels.discord.retry`로 구성합니다. [재시도 전략](/concepts/retry) 참조.

## 구성

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

확인 반응은 `messages.ackReaction` + `messages.ackReactionScope`로 전역 제어됩니다. 봇이 응답한 후 확인 반응을 지우려면 `messages.removeAckAfterReply`를 사용하세요.

- `dm.enabled`: `false`로 설정하면 모든 DM 무시(기본값 `true`).
- `dm.policy`: DM 접근 제어(`pairing` 권장). `"open"`은 `dm.allowFrom=["*"]` 필요.
- `dm.allowFrom`: DM 허용 목록(사용자 ID 또는 이름). `dm.policy="allowlist"` 및 `dm.policy="open"` 검증에 사용. 마법사는 봇이 멤버를 검색할 수 있을 때 사용자 이름을 수락하고 ID로 해석합니다.
- `dm.groupEnabled`: 그룹 DM 활성화(기본값 `false`).
- `dm.groupChannels`: 그룹 DM 채널 ID 또는 slug의 선택적 허용 목록.
- `groupPolicy`: 서버 채널 처리 방법 제어(`open|disabled|allowlist`). `allowlist`는 채널 허용 목록 필요.
- `guilds`: 서버 ID(권장) 또는 slug를 키로 하는 서버별 규칙.
- `guilds."*"`: 명시적 항목이 없을 때 적용되는 기본 서버별 설정.
- `guilds.<id>.slug`: 표시 이름용 선택적 친근한 slug.
- `guilds.<id>.users`: 선택적 서버별 사용자 허용 목록(ID 또는 이름).
- `guilds.<id>.tools`: 선택적 서버별 도구 정책 재정의(`allow`/`deny`/`alsoAllow`), 채널 재정의가 없을 때 사용.
- `guilds.<id>.toolsBySender`: 선택적 발신자별 도구 정책 재정의(서버 수준, 채널 재정의가 없을 때 적용. `"*"` 와일드카드 지원).
- `guilds.<id>.channels.<channel>.allow`: `groupPolicy="allowlist"`일 때 채널 허용/거부.
- `guilds.<id>.channels.<channel>.requireMention`: 채널의 멘션 게이팅.
- `guilds.<id>.channels.<channel>.tools`: 선택적 채널별 도구 정책 재정의(`allow`/`deny`/`alsoAllow`).
- `guilds.<id>.channels.<channel>.toolsBySender`: 선택적 채널 내 발신자별 도구 정책 재정의(`"*"` 와일드카드 지원).
- `guilds.<id>.channels.<channel>.users`: 선택적 채널별 사용자 허용 목록.
- `guilds.<id>.channels.<channel>.skills`: Skills 필터(생략 = 모든 Skills, 비어 있음 = 없음).
- `guilds.<id>.channels.<channel>.systemPrompt`: 채널의 추가 시스템 프롬프트(채널 주제와 병합).
- `guilds.<id>.channels.<channel>.enabled`: `false`로 설정하면 채널 비활성화.
- `guilds.<id>.channels`: 채널 규칙(키는 채널 slug 또는 ID).
- `guilds.<id>.requireMention`: 서버별 멘션 요구 사항(채널별로 재정의 가능).
- `guilds.<id>.reactionNotifications`: 반응 시스템 이벤트 모드(`off`, `own`, `all`, `allowlist`).
- `textChunkLimit`: 아웃바운드 텍스트 청킹 크기(문자). 기본값: 2000.
- `chunkMode`: `length`(기본값)는 `textChunkLimit` 초과 시에만 분할. `newline`은 길이별 청킹 전에 빈 줄(단락 경계)에서 분할.
- `maxLinesPerMessage`: 메시지당 소프트 최대 줄 수. 기본값: 17.
- `mediaMaxMb`: 디스크에 저장되는 인바운드 미디어 크기 제한.
- `historyLimit`: 멘션에 응답할 때 컨텍스트로 포함할 최근 서버 메시지 수(기본값 20. `messages.groupChat.historyLimit`으로 폴백. `0`이면 비활성화).
- `dmHistoryLimit`: DM 기록 제한(사용자 턴 수). 사용자별 재정의: `dms["<user_id>"].historyLimit`.
- `retry`: 아웃바운드 Discord API 호출 재시도 전략(attempts, minDelayMs, maxDelayMs, jitter).
- `pluralkit`: PluralKit 프록시 메시지를 파싱하여 시스템 멤버가 다른 발신자로 표시되도록 합니다.
- `actions`: 작업별 도구 게이팅. 생략하면 모두 허용(`false`로 설정하면 비활성화).
  - `reactions`(반응 추가 + 반응 읽기 포함)
  - `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  - `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  - `channels`(채널 생성/편집/삭제 + 카테고리 + 권한)
  - `roles`(역할 추가/제거, 기본값 `false`)
  - `moderation`(타임아웃/킥/밴, 기본값 `false`)

반응 알림은 `guilds.<id>.reactionNotifications` 사용:

- `off`: 반응 이벤트 없음.
- `own`: 봇 자신의 메시지에 대한 반응(기본값).
- `all`: 모든 메시지에 대한 모든 반응.
- `allowlist`: `guilds.<id>.users`의 사용자가 모든 메시지에 대한 반응(빈 목록이면 비활성화).

### PluralKit(PK) 지원

PK 조회를 활성화하여 프록시 메시지가 기본 시스템 + 멤버로 해석됩니다. 활성화하면 OpenClaw가 허용 목록 매칭에 멤버 신원을 사용하고 발신자를 `Member (PK:System)`으로 레이블링하여 의도치 않은 Discord 핑을 방지합니다.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // 선택. 비공개 시스템에 필요
      },
    },
  },
}
```

허용 목록 참고(PK 활성화 시):

- `dm.allowFrom`, `guilds.<id>.users` 또는 채널별 `users`에 `pk:<memberId>` 사용.
- 멤버 표시 이름도 이름/slug 매칭을 통해 작동합니다.
- 조회는 **원본** Discord 메시지 ID(프록시 전 메시지)를 사용하므로 PK API는 30분 창 내에서만 해석합니다.
- PK 조회가 실패하면(예: 토큰 없는 비공개 시스템) 프록시 메시지는 봇 메시지로 취급되어 폐기됩니다. `channels.discord.allowBots=true`를 설정하지 않는 한.

### 도구 작업 기본값

| 작업 그룹 | 기본값 | 설명 |
| --- | --- | --- |
| reactions | 활성화 | 반응 추가 + 반응 나열 + emojiList |
| stickers | 활성화 | 스티커 전송 |
| emojiUploads | 활성화 | 이모지 업로드 |
| stickerUploads | 활성화 | 스티커 업로드 |
| polls | 활성화 | 투표 생성 |
| permissions | 활성화 | 채널 권한 스냅샷 |
| messages | 활성화 | 읽기/전송/편집/삭제 |
| threads | 활성화 | 생성/나열/응답 |
| pins | 활성화 | 고정/고정 해제/나열 |
| search | 활성화 | 메시지 검색(프리뷰 기능) |
| memberInfo | 활성화 | 멤버 정보 |
| roleInfo | 활성화 | 역할 나열 |
| channelInfo | 활성화 | 채널 정보 + 나열 |
| channels | 활성화 | 채널/카테고리 관리 |
| voiceStatus | 활성화 | 음성 상태 쿼리 |
| events | 활성화 | 예약된 이벤트 나열/생성 |
| roles | 비활성화 | 역할 추가/제거 |
| moderation | 비활성화 | 타임아웃/킥/밴 |

- `replyToMode`: `off`(기본값), `first` 또는 `all`. 모델 출력에 응답 태그가 포함된 경우에만 적용.

## 응답 태그

스레드 응답을 요청하려면 모델이 출력에 태그를 포함할 수 있습니다:

- `[[reply_to_current]]` — 트리거된 Discord 메시지에 응답.
- `[[reply_to:<id>]]` — 컨텍스트/기록의 특정 메시지 ID에 응답.
  현재 메시지 ID는 프롬프트에 `[message_id: …]`로 첨부됩니다. 기록 항목에는 이미 ID가 포함되어 있습니다.

동작은 `channels.discord.replyToMode`로 제어:

- `off`: 태그 무시.
- `first`: 첫 번째 아웃바운드 청크/첨부만 응답으로.
- `all`: 모든 아웃바운드 청크/첨부가 응답으로.

허용 목록 매칭 참고:

- `allowFrom`/`users`/`groupChannels`는 ID, 이름, 태그 또는 `<@id>` 형식의 멘션을 허용합니다.
- `discord:`/`user:`(사용자) 및 `channel:`(그룹 DM) 같은 접두사 지원.
- `*`를 사용하여 모든 발신자/채널 허용.
- `guilds.<id>.channels`가 있으면 목록에 없는 채널은 기본적으로 거부됩니다.
- `guilds.<id>.channels`가 생략되면 허용 목록의 서버의 모든 채널이 허용됩니다.
- 채널을 **허용하지 않으려면** `channels.discord.groupPolicy: "disabled"`를 설정(또는 빈 허용 목록 유지).
- 구성 마법사는 `Guild/Channel` 이름(공개 + 비공개)을 수락하고 가능한 경우 ID로 해석합니다.
- 시작 시 OpenClaw가 허용 목록의 채널/사용자 이름을 ID로 해석하고(봇이 멤버를 검색할 수 있을 때) 매핑을 기록합니다. 해석되지 않은 항목은 그대로 유지됩니다.

네이티브 명령 참고:

- 등록된 명령은 OpenClaw의 채팅 명령과 일치합니다.
- 네이티브 명령은 DM/서버 메시지와 동일한 허용 목록을 따릅니다(`channels.discord.dm.allowFrom`, `channels.discord.guilds`, 채널별 규칙).
- 슬래시 명령은 Discord UI에서 허용 목록에 없는 사용자에게도 보일 수 있습니다. OpenClaw는 실행 시 허용 목록을 적용하고 "권한 없음"으로 응답합니다.

## 도구 작업

에이전트는 `discord`를 호출하여 다음을 수행할 수 있습니다:

- `react` / `reactions`(반응 추가 또는 나열)
- `sticker`, `poll`, `permissions`
- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- 읽기/검색/고정 도구의 페이로드에는 정규화된 `timestampMs`(UTC 에포크 밀리초)와 `timestampUtc`가 포함되며, 원래 Discord `timestamp`도 보존됩니다.
- `threadCreate`, `threadList`, `threadReply`
- `pinMessage`, `unpinMessage`, `listPins`
- `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
- `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
- `timeout`, `kick`, `ban`

Discord 메시지 ID는 주입된 컨텍스트에 렌더링됩니다(`[discord message id: …]` 및 기록 행)로 에이전트가 대상을 지정할 수 있습니다.
이모지는 유니코드(예: `✅`) 또는 `<:party_blob:1234567890>` 같은 사용자 정의 이모지 구문이 될 수 있습니다.

## 보안 및 운영

- 봇 토큰을 비밀번호처럼 취급하세요. 관리되는 호스트에서는 `DISCORD_BOT_TOKEN` 환경 변수나 잠긴 구성 파일 권한을 권장합니다.
- 봇에 필요한 권한만 부여하세요(보통 메시지 읽기/전송).
- 봇이 멈추거나 레이트 리밋이 걸리면 다른 프로세스가 Discord 세션을 점유하고 있지 않은지 확인한 후 Gateway를 다시 시작하세요(`openclaw gateway --force`).
