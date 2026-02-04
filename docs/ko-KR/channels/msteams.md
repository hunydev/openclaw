---
read_when:
  - Microsoft Teams 채널을 구성하거나 디버깅할 때
  - 운영자가 Microsoft Teams 통합에 대해 질문할 때
summary: Microsoft Teams에 연결하기 위한 설치 및 구성 가이드
title: Microsoft Teams
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Microsoft Teams

OpenClaw는 Microsoft Teams 봇 통합을 지원합니다.

- Azure Bot Service를 통한 연결
- 개인 채팅, 그룹 채팅, 팀 채널 지원
- RSC(리소스별 동의) 권한 모델

Teams 메시지는 Pi 에이전트로 전달됩니다.

## 빠른 시작

### 사전 요구 사항

- Microsoft 365 조직 계정
- Azure 구독(무료 계층 가능)
- Teams 앱 업로드 권한(조직 관리자 또는 사이드로딩 허용)

### 설정 단계

1. **Azure Bot 생성**

   - [Azure Portal](https://portal.azure.com)에서 **Azure Bot** 리소스 생성
   - **Bot handle**: 고유한 이름 입력
   - **Type of App**: Multi Tenant
   - 생성 후 **Configuration**에서:
     - **Microsoft App ID** 복사
     - **Manage Password** → 새 클라이언트 시크릿 생성 및 복사

2. **메시징 엔드포인트 설정**

   - **Configuration** → **Messaging endpoint**
   - URL 설정: `https://your-domain.com/msteams/messages`
   - Gateway가 이 URL에서 수신 대기해야 함

3. **Teams 채널 추가**

   - **Channels** → **Microsoft Teams** 추가
   - 서비스 약관 동의

4. **OpenClaw 구성**

```json5
{
  channels: {
    msteams: {
      appId: "your-microsoft-app-id",
      appPassword: "your-client-secret",
      tenantId: "your-tenant-id", // 또는 "common" (멀티테넌트)
    },
  },
}
```

5. **Gateway 시작** 후 Teams 앱 설치

## Azure Bot 상세 설정

### Bot 생성 옵션

| 옵션                  | 권장 값                      | 설명                              |
| :-------------------- | :--------------------------- | :-------------------------------- |
| Bot handle            | `openclaw-bot`               | 고유 식별자                       |
| Subscription          | 사용 중인 구독               | 청구 대상                         |
| Resource group        | 신규 또는 기존               | 리소스 그룹화                     |
| Location              | 가까운 리전                  | 지연 시간 최소화                  |
| Pricing tier          | F0 (Free)                    | 개인 사용에 충분                  |
| Type of App           | Multi Tenant                 | 범용                              |
| Creation type         | Create new Microsoft App ID  | 새 앱 ID 자동 생성                |

### Microsoft App ID 및 시크릿

1. **Azure Bot** → **Configuration**
2. **Microsoft App ID** 확인
3. **Manage Password** 클릭 → Azure AD 앱으로 이동
4. **Certificates & secrets** → **New client secret**
5. 설명 입력, 만료 기간 선택 → **Add**
6. **Value** 즉시 복사 (나중에 볼 수 없음)

### 메시징 엔드포인트

Gateway의 퍼블릭 URL이 필요합니다:

```
https://your-domain.com/msteams/messages
```

**로컬 개발 시**:

- ngrok 또는 cloudflared 터널 사용
- 예: `ngrok http 3000` → `https://xxxx.ngrok.io/msteams/messages`

## Teams 앱 매니페스트

Teams에 봇을 설치하려면 앱 매니페스트가 필요합니다.

### manifest.json 생성

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.16/MicrosoftTeams.schema.json",
  "manifestVersion": "1.16",
  "version": "1.0.0",
  "id": "your-microsoft-app-id",
  "packageName": "com.yourorg.openclawbot",
  "developer": {
    "name": "Your Organization",
    "websiteUrl": "https://your-website.com",
    "privacyUrl": "https://your-website.com/privacy",
    "termsOfUseUrl": "https://your-website.com/terms"
  },
  "name": {
    "short": "OpenClaw",
    "full": "OpenClaw AI Assistant"
  },
  "description": {
    "short": "AI assistant powered by OpenClaw",
    "full": "Personal AI assistant for Teams, powered by OpenClaw and Claude."
  },
  "icons": {
    "outline": "outline.png",
    "color": "color.png"
  },
  "accentColor": "#6366F1",
  "bots": [
    {
      "botId": "your-microsoft-app-id",
      "scopes": ["personal", "team", "groupchat"],
      "supportsFiles": true,
      "isNotificationOnly": false,
      "commandLists": [
        {
          "scopes": ["personal", "team", "groupchat"],
          "commands": [
            {
              "title": "help",
              "description": "도움말 표시"
            },
            {
              "title": "clear",
              "description": "대화 초기화"
            }
          ]
        }
      ]
    }
  ],
  "permissions": ["identity", "messageTeamMembers"],
  "validDomains": ["your-domain.com"]
}
```

### 앱 패키지 생성

1. `manifest.json`, `color.png`(192x192), `outline.png`(32x32) 준비
2. ZIP 파일로 압축 (폴더 없이 파일만)

### Teams에 앱 설치

**개인 사이드로딩:**

1. Teams → **앱** → **앱 관리** → **앱 업로드**
2. **사용자 지정 앱 업로드** 선택
3. ZIP 파일 업로드
4. **추가** 클릭

**조직 배포:**

1. [Teams Admin Center](https://admin.teams.microsoft.com)
2. **Teams 앱** → **앱 관리**
3. **업로드** → 새 앱
4. 정책 할당으로 사용자에게 배포

## DM 정책

기본 모드는 `allowlist`입니다:

```json5
{
  channels: {
    msteams: {
      dmPolicy: "open", // 모든 DM 허용
      // 또는 "disabled" 로 모든 DM 무시
      // 또는 "allowlist" (기본값, allowFrom 필요)
      // 또는 "pairing" 으로 페어링 흐름 활성화
    },
  },
}
```

`allowlist` 모드에서:

```json5
{
  channels: {
    msteams: {
      dmPolicy: "allowlist",
      allowFrom: ["user@domain.com", "aadUserId-..."], // 이메일 또는 AAD ID
    },
  },
}
```

**사용자 ID 형식:**

- 이메일: `user@yourdomain.com`
- AAD 사용자 ID: `29:xxxx` 형식의 Teams 사용자 ID

## 팀/채널 설정

팀 채널에서 봇 사용:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist", // allowlist, open, disabled
      groups: ["team-id-1", "channel-id-2"], // 팀 또는 채널 ID
      groupChat: {
        activationMode: "mention", // mention (기본값), always
        mentionPatterns: ["claude", "assistant"], // 추가 트리거 단어
        historyLimit: 50, // 컨텍스트에 포함할 메시지 수
      },
    },
  },
}
```

### 활성화 모드

- `mention`: @봇 멘션 또는 `mentionPatterns` 포함 시에만 응답
- `always`: 채널의 모든 메시지에 응답

## RSC 권한

RSC(리소스별 동의)로 세분화된 권한 관리:

### 매니페스트에 RSC 추가

```json
{
  "webApplicationInfo": {
    "id": "your-microsoft-app-id",
    "resource": "https://your-domain.com"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        {
          "name": "ChannelMessage.Read.Group",
          "type": "Application"
        },
        {
          "name": "ChatMessage.Read.Chat",
          "type": "Application"
        },
        {
          "name": "TeamsActivity.Send.User",
          "type": "Application"
        }
      ]
    }
  }
}
```

### 주요 RSC 권한

| 권한                          | 유형          | 설명                          |
| :---------------------------- | :------------ | :---------------------------- |
| `ChannelMessage.Read.Group`   | Application   | 채널 메시지 읽기              |
| `ChatMessage.Read.Chat`       | Application   | 채팅 메시지 읽기              |
| `TeamsActivity.Send.User`     | Application   | 활동 피드에 알림 전송         |
| `TeamSettings.Read.Group`     | Application   | 팀 설정 읽기                  |
| `Member.Read.Group`           | Application   | 팀 멤버 읽기                  |

## 미디어

Teams는 파일 첨부를 지원합니다.

**인바운드:**

- 이미지, 문서, 비디오 등
- `mediaMaxMb`로 크기 제한(기본값 50MB)

**아웃바운드:**

- 이미지, 문서 전송 가능
- `agents.defaults.mediaMaxMb`로 제한(기본값 5MB)

```json5
{
  channels: {
    msteams: {
      mediaMaxMb: 50,
    },
  },
  agents: {
    defaults: {
      mediaMaxMb: 5,
    },
  },
}
```

## Adaptive Cards

Teams는 Adaptive Cards를 통해 풍부한 메시지 형식을 지원합니다:

```json5
{
  channels: {
    msteams: {
      adaptiveCards: {
        enabled: true, // Adaptive Cards 사용
      },
    },
  },
}
```

**기본 카드 예시:**

```json
{
  "type": "AdaptiveCard",
  "version": "1.5",
  "body": [
    {
      "type": "TextBlock",
      "text": "Hello from OpenClaw!",
      "size": "large",
      "weight": "bolder"
    },
    {
      "type": "TextBlock",
      "text": "This is a rich message with formatting.",
      "wrap": true
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "Learn More",
      "url": "https://docs.openclaw.ai"
    }
  ]
}
```

## 반응

메시지에 반응 추가:

```json5
{
  channels: {
    msteams: {
      actions: {
        reactions: true, // 기본값: 활성화
      },
    },
  },
}
```

**참고**: Teams 반응은 제한된 이모지 세트만 지원합니다.

## 타이핑 표시기

봇이 응답 생성 중임을 표시:

```json5
{
  channels: {
    msteams: {
      typingIndicator: true, // 기본값: 활성화
    },
  },
}
```

## 스트리밍

Teams는 메시지 업데이트를 통한 스트리밍을 지원합니다:

```json5
{
  channels: {
    msteams: {
      streaming: true, // 스트리밍 응답 활성화
      streamingThrottle: 1000, // 업데이트 간격(ms)
    },
  },
}
```

**제한**: Teams API 속도 제한으로 인해 너무 빈번한 업데이트는 제한될 수 있습니다.

## 명령

봇 명령 설정(매니페스트에서 정의):

```json
{
  "commands": [
    { "title": "help", "description": "도움말 표시" },
    { "title": "clear", "description": "대화 초기화" },
    { "title": "settings", "description": "설정 보기" }
  ]
}
```

OpenClaw 내장 명령:

- `/help` - 도움말
- `/clear` - 세션 초기화
- `/config` - 구성 보기/변경

## 다중 테넌트

여러 조직(테넌트) 지원:

```json5
{
  channels: {
    msteams: {
      tenantId: "common", // 모든 테넌트 허용
      // 또는 특정 테넌트: "your-tenant-id"
    },
  },
}
```

**테넌트 제한:**

```json5
{
  channels: {
    msteams: {
      allowedTenants: ["tenant-id-1", "tenant-id-2"], // 특정 테넌트만 허용
    },
  },
}
```

## 알림

사전 알림 전송(사용자가 먼저 대화를 시작해야 함):

```json5
{
  channels: {
    msteams: {
      proactiveMessaging: true, // 사전 메시지 활성화
    },
  },
}
```

**제한**: 사용자가 봇과 한 번 이상 대화해야 사전 메시지 전송 가능.

## 구성 요약

| 키                                    | 설명                        | 기본값      |
| :------------------------------------ | :-------------------------- | :---------- |
| `channels.msteams.appId`              | Microsoft App ID            | -           |
| `channels.msteams.appPassword`        | 클라이언트 시크릿           | -           |
| `channels.msteams.tenantId`           | 테넌트 ID                   | `common`    |
| `channels.msteams.dmPolicy`           | DM 정책                     | `allowlist` |
| `channels.msteams.allowFrom`          | 허용 사용자                 | `[]`        |
| `channels.msteams.groupPolicy`        | 그룹/채널 정책              | `allowlist` |
| `channels.msteams.groups`             | 허용 팀/채널 ID             | `[]`        |
| `channels.msteams.mediaMaxMb`         | 인바운드 미디어 제한(MB)    | `50`        |
| `channels.msteams.typingIndicator`    | 타이핑 표시기               | `true`      |
| `channels.msteams.streaming`          | 스트리밍 응답               | `false`     |
| `channels.msteams.streamingThrottle`  | 스트리밍 업데이트 간격(ms)  | `1000`      |
| `channels.msteams.adaptiveCards`      | Adaptive Cards 설정         | -           |
| `channels.msteams.actions.reactions`  | 반응 도구 활성화            | `true`      |
| `channels.msteams.allowedTenants`     | 허용 테넌트 목록            | -           |

## 문제 해결

### "Unauthorized" 오류

- **App ID**와 **App Password**가 올바른지 확인
- 클라이언트 시크릿이 만료되지 않았는지 확인
- Azure AD 앱에서 시크릿 재생성

### 봇이 응답하지 않음

1. `channels status`로 연결 상태 확인
2. 메시징 엔드포인트 URL이 올바른지 확인
3. Gateway가 해당 포트에서 수신 대기 중인지 확인
4. 로그 확인: `openclaw logs --follow`

### 앱 설치 실패

- 사이드로딩이 조직에서 허용되는지 확인
- 매니페스트의 `id`가 Azure Bot의 App ID와 일치하는지 확인
- 아이콘 파일이 올바른 크기인지 확인 (color: 192x192, outline: 32x32)

### 채널에서 메시지를 받지 못함

1. 봇이 팀에 추가되었는지 확인
2. `groupPolicy`와 `groups` 설정 확인
3. RSC 권한이 올바르게 설정되었는지 확인

### 속도 제한

Teams API는 초당 요청 수를 제한합니다:

- 스트리밍 사용 시 `streamingThrottle`을 높이세요
- 대량 메시지 전송 시 간격 두기

### 테넌트 접근 거부

- `tenantId`가 올바른지 확인
- 멀티테넌트 앱인 경우 `common` 사용
- `allowedTenants`에 테넌트가 포함되어 있는지 확인

## 보안 고려 사항

### 시크릿 관리

- 클라이언트 시크릿을 코드에 하드코딩하지 마세요
- 환경 변수 또는 시크릿 관리 서비스 사용
- 시크릿을 정기적으로 교체

### 접근 제어

- `allowFrom`으로 DM 접근 제한
- `groups`로 채널 접근 제한
- `allowedTenants`로 조직 접근 제한

### 감사 로깅

Teams 관리 센터에서 봇 활동 감사 가능.

## 참고

- [Microsoft Bot Framework](https://dev.botframework.com/)
- [Teams 앱 개발 문서](https://docs.microsoft.com/microsoftteams/platform/)
- [Adaptive Cards 디자이너](https://adaptivecards.io/designer/)
- [Azure Bot Service](https://azure.microsoft.com/services/bot-services/)
