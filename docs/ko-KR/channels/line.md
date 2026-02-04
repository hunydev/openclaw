---
read_when:
  - OpenClaw를 LINE에 연결하려 할 때
  - LINE webhook + 자격 증명 설정이 필요할 때
  - LINE 전용 메시지 옵션이 필요할 때
summary: LINE Messaging API 플러그인 설정, 구성 및 사용법
title: LINE
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# LINE(플러그인)

LINE은 LINE Messaging API를 통해 OpenClaw에 연결됩니다. 플러그인은 Gateway에서 webhook 수신기로 실행되며, 채널 접근 토큰 + 채널 시크릿으로 인증합니다.

상태: 플러그인을 통해 지원됨. DM, 그룹 채팅, 미디어, 위치, Flex 메시지, 템플릿 메시지 및 빠른 답장 지원. 반응과 스레드는 지원되지 않음.

## 플러그인 필요

LINE 플러그인 설치:

```bash
openclaw plugins install @openclaw/line
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/line
```

## 설정

1. LINE Developers 계정을 만들고 콘솔 열기:
   https://developers.line.biz/console/
2. Provider를 만들거나 선택하고 **Messaging API** 채널 추가.
3. 채널 설정에서 **Channel access token**과 **Channel secret** 복사.
4. Messaging API 설정에서 **Use webhook** 활성화.
5. Webhook URL을 Gateway 엔드포인트로 설정(HTTPS 필요):

```
https://gateway-host/line/webhook
```

Gateway는 LINE의 webhook 검증(GET)과 인바운드 이벤트(POST)에 응답합니다. 사용자 정의 경로가 필요하면 `channels.line.webhookPath` 또는 `channels.line.accounts.<id>.webhookPath`를 설정하고 URL을 업데이트하세요.

## 구성

최소 구성:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

환경 변수(기본 계정만):

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

토큰/시크릿 파일:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

다중 계정:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## 접근 제어

DM은 기본적으로 페어링이 필요합니다. 알 수 없는 발신자는 페어링 코드를 받으며, 승인 전까지 메시지는 무시됩니다.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

허용 목록 및 정책:

- `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
- `channels.line.allowFrom`: DM에 허용된 LINE 사용자 ID 목록
- `channels.line.groupPolicy`: `allowlist | open | disabled`
- `channels.line.groupAllowFrom`: 그룹에 허용된 LINE 사용자 ID 목록
- 그룹별 재정의: `channels.line.groups.<groupId>.allowFrom`

LINE ID는 대소문자를 구분합니다. 유효한 ID 형식:

- 사용자: `U` + 32자리 16진수
- 그룹: `C` + 32자리 16진수
- 룸: `R` + 32자리 16진수

## 메시지 동작

- 텍스트는 5000자에서 청크로 분할됩니다.
- Markdown 포맷은 제거되며, 코드 블록과 테이블은 가능한 경우 Flex 카드로 변환됩니다.
- 스트리밍 응답은 버퍼링되며, 에이전트 작업 중 LINE은 전체 청크를 수신하고 로딩 애니메이션을 표시합니다.
- 미디어 다운로드 제한: `channels.line.mediaMaxMb`(기본값 10).

## 채널 데이터(리치 메시지)

`channelData.line`을 사용하여 빠른 답장, 위치, Flex 카드 또는 템플릿 메시지 전송.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex 페이로드 */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

LINE 플러그인에는 Flex 메시지 프리셋용 `/card` 명령도 있습니다:

```
/card info "Welcome" "Thanks for joining!"
```

## 문제 해결

- **Webhook 검증 실패:** webhook URL이 HTTPS인지, `channelSecret`이 LINE 콘솔과 일치하는지 확인.
- **인바운드 이벤트 없음:** webhook 경로가 `channels.line.webhookPath`와 일치하고 Gateway가 LINE에서 접근 가능한지 확인.
- **미디어 다운로드 오류:** 미디어가 기본 제한을 초과하면 `channels.line.mediaMaxMb` 증가.
