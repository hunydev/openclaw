---
read_when:
  - Mattermost 설정 시
  - Mattermost 라우팅 디버깅 시
summary: Mattermost 봇 설정 및 OpenClaw 설정
title: Mattermost
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 57fabe5eb0efbcb885f4178b317b2fa99a41daf609e3a471de2b44db9def4ad7
  source_path: channels/mattermost.md
  workflow: 14
---

# Mattermost(플러그인)

상태: 플러그인을 통해 지원(봇 토큰 + WebSocket 이벤트). 채널, 그룹 및 DM 지원. Mattermost는 자체 호스팅 가능한 팀 메시징 플랫폼입니다. 제품 세부 정보 및 다운로드는 공식 웹사이트 [mattermost.com](https://mattermost.com)을 방문하세요.

## 플러그인 필요

Mattermost는 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

CLI를 통해 설치(npm 레지스트리):

```bash
openclaw plugins install @openclaw/mattermost
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/mattermost
```

설정/온보딩 중 Mattermost를 선택하고 git 체크아웃이 감지되면 OpenClaw가 자동으로 로컬 설치 경로를 제안합니다.

자세한 내용: [플러그인](/plugin)

## 빠른 설정

1. Mattermost 플러그인 설치.
2. Mattermost 봇 계정을 만들고 **봇 토큰**을 복사.
3. Mattermost **베이스 URL** 복사(예: `https://chat.example.com`).
4. OpenClaw를 설정하고 Gateway 시작.

최소 설정:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## 환경 변수(기본 계정)

환경 변수를 선호하는 경우 Gateway 호스트에서 설정:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

환경 변수는 **기본** 계정(`default`)에만 적용됩니다. 다른 계정은 설정 값을 사용해야 합니다.

## 채팅 모드

Mattermost는 DM에 자동 응답합니다. 채널 동작은 `chatmode`로 제어됩니다:

- `oncall`(기본값): 채널에서 @멘션될 때만 응답.
- `onmessage`: 채널의 모든 메시지에 응답.
- `onchar`: 메시지가 트리거 접두사로 시작할 때 응답.

설정 예시:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

참고:

- `onchar` 모드는 여전히 명시적 @멘션에 응답합니다.
- `channels.mattermost.requireMention`은 레거시 설정에서 여전히 작동하지만 `chatmode`를 권장합니다.

## 접근 제어(DM)

- 기본값: `channels.mattermost.dmPolicy = "pairing"`(알 수 없는 발신자는 페어링 코드를 받음).
- 승인 방법:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- 공개 DM: `channels.mattermost.dmPolicy="open"` + `channels.mattermost.allowFrom=["*"]`.

## 채널(그룹)

- 기본값: `channels.mattermost.groupPolicy = "allowlist"`(멘션 게이팅).
- `channels.mattermost.groupAllowFrom`으로 발신자 허용 목록(사용자 ID 또는 `@username`).
- 열린 채널: `channels.mattermost.groupPolicy="open"`(멘션 게이팅).

## 아웃바운드 전달 대상

`openclaw message send` 또는 크론/웹훅에서 다음 대상 형식 사용:

- `channel:<id>` - 채널용
- `user:<id>` - DM용
- `@username` - DM용(Mattermost API를 통해 해석됨)

순수 ID는 채널로 취급됩니다.

## 다중 계정

Mattermost는 `channels.mattermost.accounts` 아래에서 여러 계정을 지원합니다:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## 문제 해결

- 채널에서 응답 없음: 봇이 채널에 참여했는지 확인하고 멘션(oncall 모드)하거나, 트리거 접두사(onchar 모드)를 사용하거나, `chatmode: "onmessage"`를 설정하세요.
- 인증 오류: 봇 토큰, 베이스 URL, 계정 활성화 여부를 확인하세요.
- 다중 계정 문제: 환경 변수는 `default` 계정에만 적용됩니다.
