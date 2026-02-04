---
read_when:
  - OpenClaw Twitch 채팅 통합 설정 시
summary: Twitch 채팅 봇 설정 및 구성
title: Twitch
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: aa7d60444e7f7e5dd7d02ce21527089058e024b8f427aeedf9e200a2818eb007
  source_path: channels/twitch.md
  workflow: 14
---

# Twitch(플러그인)

IRC를 통한 Twitch 채팅 지원. OpenClaw는 Twitch 사용자(봇 계정)로 연결하여 채널에서 메시지를 받고 보냅니다.

## 플러그인 필요

Twitch는 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

CLI를 통해 설치(npm 레지스트리):

```bash
openclaw plugins install @openclaw/twitch
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/twitch
```

자세한 내용: [플러그인](/plugin)

## 빠른 설정(초보자용)

1. 봇용 전용 Twitch 계정 생성(또는 기존 계정 사용).
2. 자격 증명 생성: [Twitch Token Generator](https://twitchtokengenerator.com/)
   - **Bot Token** 선택
   - `chat:read` 및 `chat:write` 스코프가 체크되어 있는지 확인
   - **Client ID**와 **Access Token** 복사
3. Twitch 사용자 ID 찾기: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. 토큰 설정:
   - 환경 변수: `OPENCLAW_TWITCH_ACCESS_TOKEN=...`(기본 계정만)
   - 또는 설정: `channels.twitch.accessToken`
   - 둘 다 설정되면 설정이 우선(환경 변수 폴백은 기본 계정만).
5. Gateway 시작.

**⚠️ 중요:** 권한 없는 사용자가 봇을 트리거하지 못하도록 접근 제어(`allowFrom` 또는 `allowedRoles`) 추가. `requireMention`은 기본적으로 `true`.

최소 설정:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // 봇의 Twitch 계정
      accessToken: "oauth:abc123...", // OAuth Access Token(또는 OPENCLAW_TWITCH_ACCESS_TOKEN 환경 변수 사용)
      clientId: "xyz789...", // Token Generator의 Client ID
      channel: "vevisk", // 참여할 Twitch 채널 채팅(필수)
      allowFrom: ["123456789"], // (권장) 본인 Twitch 사용자 ID만 - https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/에서 획득
    },
  },
}
```

## 작동 방식

- Gateway가 소유한 Twitch 채널.
- 확정적 라우팅: 응답은 항상 Twitch로 돌아감.
- 각 계정은 격리된 세션 키 `agent:<agentId>:twitch:<accountName>`에 매핑됨.
- `username`은 봇 계정(인증용), `channel`은 참여할 채팅방.

## 설정(상세)

### 자격 증명 생성

[Twitch Token Generator](https://twitchtokengenerator.com/) 사용:

- **Bot Token** 선택
- `chat:read` 및 `chat:write` 스코프가 체크되어 있는지 확인
- **Client ID**와 **Access Token** 복사

앱을 수동으로 등록할 필요 없음. 토큰은 몇 시간 후 만료됩니다.

### 봇 설정

**환경 변수(기본 계정만):**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**또는 설정:**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

환경 변수와 설정 둘 다 설정되면 설정이 우선.

### 접근 제어(권장)

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (권장) 본인 Twitch 사용자 ID만
      allowedRoles: ["moderator"], // 또는 역할로 제한
    },
  },
}
```

**사용 가능한 역할:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**왜 사용자 ID를 사용하나요?** 사용자 이름은 변경될 수 있어 사칭 위험이 있습니다. 사용자 ID는 영구적입니다.

Twitch 사용자 ID 찾기: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/(Twitch 사용자 이름을 ID로 변환)

## 토큰 새로고침(선택사항)

[Twitch Token Generator](https://twitchtokengenerator.com/)의 토큰은 자동 새로고침되지 않습니다—만료 시 다시 생성해야 합니다.

자동 토큰 새로고침을 원하면 [Twitch Developer Console](https://dev.twitch.tv/console)에서 자체 Twitch 앱을 만들고 설정에 추가하세요:

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```
