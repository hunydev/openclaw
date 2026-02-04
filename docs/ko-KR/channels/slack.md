---
read_when:
  - Slack 채널을 구성하거나 디버깅할 때
  - 운영자가 Slack 통합에 대해 질문할 때
summary: Slack에 연결하기 위한 설치 및 구성 가이드
title: Slack
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Slack

OpenClaw는 Slack 통합을 지원합니다:

- 소켓 모드(권장, 퍼블릭 IP 불필요)
- HTTP 모드(수신 URL 필요)

Slack 메시지는 Pi 에이전트로 전달됩니다.

## 빠른 시작(Socket 모드)

1. [Slack 앱 생성](https://api.slack.com/apps) + 앱 레벨 토큰(`connections:write`)으로 소켓 모드 활성화.
2. `/invite @YourBot`으로 앱을 채널에 추가.
3. `config.json`에 토큰 추가:

```json5
{
  channels: {
    slack: {
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

4. Gateway 재시작.

## 봇 범위(Bot Scopes)

다음 OAuth 범위를 가진 봇 토큰(`xoxb-...`) 필요:

| 범위                      | 목적                         |
| :------------------------ | :--------------------------- |
| `app_mentions:read`       | @멘션 수신                   |
| `channels:history`        | 퍼블릭 채널 메시지 읽기      |
| `channels:join`           | 퍼블릭 채널 자동 참가        |
| `chat:write`              | 메시지 전송                  |
| `files:read`              | 업로드된 파일 다운로드       |
| `files:write`             | 파일 업로드                  |
| `groups:history`          | 프라이빗 채널 메시지 읽기    |
| `groups:read`             | 프라이빗 채널 참가 상태 확인 |
| `groups:write`            | 프라이빗 채널 참가           |
| `im:history`              | DM 메시지 읽기               |
| `im:read`                 | DM 세부 정보 읽기            |
| `im:write`                | DM 대화 열기                 |
| `mpim:history`            | 그룹 DM 메시지 읽기          |
| `mpim:read`               | 그룹 DM 세부 정보 읽기       |
| `mpim:write`              | 그룹 DM 대화 열기            |
| `reactions:read`          | 메시지 반응 읽기             |
| `reactions:write`         | 메시지에 반응 추가           |
| `team:read`               | 팀 세부 정보 읽기            |
| `usergroups:read`         | 사용자 그룹 읽기             |
| `users:read`              | 사용자 정보 조회             |
| `users:read.email`        | 사용자 이메일 조회           |
| `users.profile:read`      | 사용자 프로필 읽기           |

→ DM 흐름의 경우 최소한 `im:history`, `im:read`, `im:write` 범위 필요.

→ 프라이빗 채널 프로브 필요 시 `groups:read`, `groups:history`, `groups:write` 범위 필요.

→ 그룹 DM 프로브 필요 시 `mpim:history`, `mpim:read`, `mpim:write` 범위 필요.

## 이벤트 구독(Event Subscriptions)

1. Slack 앱 구성에서 이벤트 구독 활성화.
2. 봇 이벤트에 다음 구독 추가:
   - `message.channels` — 퍼블릭 채널의 메시지
   - `message.groups` — 프라이빗 채널의 메시지
   - `message.im` — 봇과의 DM
   - `message.mpim` — 그룹 DM(다자 IM)
   - `app_mention` — @멘션
3. 변경 사항 저장 후 필요시 워크스페이스에 앱 재설치.

## 앱 레벨 토큰(소켓 모드용)

**앱 레벨 토큰**은 Slack의 실시간 WebSocket 연결 인증에 사용됩니다.
`xapp-...` 토큰 또는 앱 토큰이라고도 하며, 일반 `xoxb-...` 봇 토큰과 다릅니다.

앱 레벨 토큰 설정 방법:

1. [Slack 앱](https://api.slack.com/apps)에서 **설정 → 기본 정보**로 이동.
2. **앱 레벨 토큰**까지 스크롤, **토큰 생성**을 클릭.
3. 이름(예: `socket-mode`) 지정 후 범위에 **`connections:write`** 추가.
4. 토큰 생성.
5. `xapp-...` 토큰 복사 후 `config.json`에서 `channels.slack.appToken`에 추가.

## DM 정책

기본 모드는 `allowlist`입니다. `dmPolicy`로 변경:

```json5
{
  channels: {
    slack: {
      dmPolicy: "open", // 모든 DM 허용
      // 또는 "disabled" 로 모든 DM 무시
      // 또는 "allowlist" (기본값, allowFrom 필요)
      // 또는 "pairing" 으로 페어링 흐름 활성화
    },
  },
}
```

`allowlist` 모드에서는 `allowFrom`이 필요합니다(Slack 사용자 ID 배열):

```json5
{
  channels: {
    slack: {
      dmPolicy: "allowlist",
      allowFrom: ["U01234ABC"], // 허용 사용자 ID
    },
  },
}
```

## 세션 라우팅

메시지는 스레드 시작 또는 컨텍스트 기반 세션으로 라우팅됩니다.

- **스레드**: `thread_ts`로 그룹화
- **DM**: 사용자별 세션
- **채널**: 채널별 세션(스레드 외부 멘션 시)

## 스레딩

Slack에서 스레딩 동작 제어:

```json5
{
  channels: {
    slack: {
      threading: "reply", // 항상 스레드에 응답 (기본값)
      // 또는 "broadcast" 로 채널에도 전송
      // 또는 "channel" 로 스레드 무시, 채널에 직접 응답
    },
  },
}
```

## 응답 모드

기본적으로 모든 응답은 같은 채널의 스레드에 전송됩니다.

```json5
{
  channels: {
    slack: {
      responseMode: "thread", // 기본값: 스레드에 응답
      // 또는 "dm" 으로 항상 DM으로 응답
    },
  },
}
```

## 그룹 채팅 설정

채널/그룹에서의 동작 제어:

```json5
{
  channels: {
    slack: {
      groupPolicy: "allowlist", // allowlist, open, disabled
      groups: ["C01234ABC"], // 허용 채널 ID
      groupChat: {
        activationMode: "mention", // mention (기본값), always
        mentionPatterns: ["claude", "assistant"], // 추가 트리거 단어
        historyLimit: 50, // 컨텍스트에 포함할 메시지 수
      },
    },
  },
}
```

## 허용 목록 관리

`allowFrom` 및 `groups`에는 Slack ID 사용:

- **사용자 ID**: `U01234ABC` (프로필에서 찾기)
- **채널 ID**: `C01234ABC` (채널 세부 정보에서 찾기)
- 와일드카드: `"*"`로 모두 허용(주의 필요)

```json5
{
  channels: {
    slack: {
      allowFrom: ["U01234ABC", "U56789DEF"],
      groups: ["C01234ABC", "C56789DEF", "*"], // * = 모든 채널
    },
  },
}
```

## 미디어

Slack은 파일 업로드/다운로드를 지원합니다.

인바운드:

- 파일은 다운로드 후 컨텍스트에 추가
- `mediaMaxMb`로 크기 제한(기본값 50MB)

아웃바운드:

- 이미지, 문서 등 전송 가능
- `agents.defaults.mediaMaxMb`로 제한

```json5
{
  channels: {
    slack: {
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

## 파일 업로드

에이전트가 파일을 업로드할 수 있습니다:

```json5
{
  channels: {
    slack: {
      actions: {
        fileUpload: true, // 기본값: 활성화
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
    slack: {
      actions: {
        reactions: true, // 기본값: 활성화
      },
    },
  },
}
```

## 확인 반응

메시지 수신 시 자동 이모지 반응:

```json5
{
  channels: {
    slack: {
      ackReaction: {
        emoji: "eyes", // :eyes: 이모지
        direct: true, // DM에서 반응
        group: "mentions", // 채널에서: "always", "mentions", "never"
      },
    },
  },
}
```

## 타이핑 표시기

Slack은 실시간 타이핑 표시기를 지원하지 않습니다. 이 설정은 무시됩니다.

## 앱 홈

Slack 앱 홈 탭 지원(선택):

1. Slack 앱 구성에서 **앱 홈** 활성화.
2. **홈 탭** 켜기.
3. 필요시 사용자 지정 환영 메시지 구성.

## HTTP 모드(대안)

소켓 모드 대신 HTTP 엔드포인트 사용:

```json5
{
  channels: {
    slack: {
      mode: "http",
      signingSecret: "your-signing-secret",
      botToken: "xoxb-...",
    },
  },
}
```

요구 사항:

- 퍼블릭 URL 필요
- Slack 앱에서 요청 URL 구성: `https://your-domain/slack/events`
- `signingSecret`으로 요청 검증

## 슬래시 명령(선택)

슬래시 명령 활성화:

1. Slack 앱에서 **슬래시 명령** 추가.
2. 명령 생성(예: `/ask`).
3. 요청 URL: `https://your-domain/slack/commands`
4. 구성:

```json5
{
  channels: {
    slack: {
      slashCommands: {
        enabled: true,
        commands: ["/ask", "/claude"],
      },
    },
  },
}
```

## 다중 워크스페이스

여러 Slack 워크스페이스 지원:

```json5
{
  channels: {
    slack: {
      accounts: {
        workspace1: {
          appToken: "xapp-...",
          botToken: "xoxb-...",
          dmPolicy: "allowlist",
          allowFrom: ["U01234ABC"],
        },
        workspace2: {
          appToken: "xapp-...",
          botToken: "xoxb-...",
          dmPolicy: "open",
        },
      },
    },
  },
}
```

## 구성 요약

| 키                                  | 설명                                 | 기본값      |
| :---------------------------------- | :----------------------------------- | :---------- |
| `channels.slack.mode`               | `socket` 또는 `http`                 | `socket`    |
| `channels.slack.appToken`           | 앱 레벨 토큰(소켓 모드)              | -           |
| `channels.slack.botToken`           | 봇 토큰                              | -           |
| `channels.slack.signingSecret`      | 서명 시크릿(HTTP 모드)               | -           |
| `channels.slack.dmPolicy`           | DM 정책                              | `allowlist` |
| `channels.slack.allowFrom`          | 허용 사용자 ID                       | `[]`        |
| `channels.slack.groupPolicy`        | 그룹/채널 정책                       | `allowlist` |
| `channels.slack.groups`             | 허용 채널 ID                         | `[]`        |
| `channels.slack.threading`          | 스레딩 동작                          | `reply`     |
| `channels.slack.responseMode`       | 응답 모드                            | `thread`    |
| `channels.slack.mediaMaxMb`         | 인바운드 미디어 제한(MB)             | `50`        |
| `channels.slack.ackReaction`        | 확인 반응 설정                       | -           |
| `channels.slack.actions.reactions`  | 반응 도구 활성화                     | `true`      |
| `channels.slack.actions.fileUpload` | 파일 업로드 도구 활성화              | `true`      |

## 문제 해결

### 연결되지 않음

- `channels status`로 상태 확인.
- 앱 토큰에 `connections:write` 범위 있는지 확인.
- 소켓 모드가 Slack 앱에서 활성화되었는지 확인.

### 메시지를 받지 못함

- 이벤트 구독이 올바르게 구성되었는지 확인.
- 봇이 채널에 초대되었는지 확인(`/invite @YourBot`).
- 필요한 범위가 모두 있는지 확인.

### 권한 오류

- 봇 토큰 범위 확인.
- 필요시 워크스페이스에 앱 재설치.
- OAuth & 권한 페이지에서 범위 검토.

### HTTP 모드 문제

- 서명 시크릿이 올바른지 확인.
- URL이 퍼블릭하게 접근 가능한지 확인.
- Slack 앱의 요청 URL이 올바른지 확인.

## 참고

- [Slack API 문서](https://api.slack.com/)
- [소켓 모드](https://api.slack.com/apis/connections/socket)
- [이벤트 API](https://api.slack.com/events-api)
