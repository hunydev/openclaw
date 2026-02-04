---
read_when:
  - Google Chat 채널 기능 개발 시
summary: Google Chat 앱 지원 상태, 기능 및 구성
title: Google Chat
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Google Chat(Chat API)

상태: Google Chat API webhook(HTTP 전용)을 통해 DM과 스페이스에서 사용 가능합니다.

## 빠른 설정(초보자)

1. Google Cloud 프로젝트를 만들고 **Google Chat API**를 활성화합니다.
   - 이동: [Google Chat API 자격 증명](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - 아직 활성화되지 않았다면 API를 활성화합니다.
2. **서비스 계정** 생성:
   - **Create Credentials** > **Service Account** 클릭.
   - 이름 지정(예: `openclaw-chat`).
   - 권한은 비워두기(**Continue** 클릭).
   - 접근 권한이 있는 주체도 비워두기(**Done** 클릭).
3. **JSON 키** 생성 및 다운로드:
   - 서비스 계정 목록에서 방금 만든 것 클릭.
   - **Keys** 탭으로 이동.
   - **Add Key** > **Create new key** 클릭.
   - **JSON** 선택 후 **Create** 클릭.
4. 다운로드한 JSON 파일을 Gateway 호스트에 저장(예: `~/.openclaw/googlechat-service-account.json`).
5. [Google Cloud Console Chat 구성](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat)에서 Google Chat 앱 생성:
   - **Application info** 입력:
     - **App name**: (예: `OpenClaw`)
     - **Avatar URL**: (예: `https://openclaw.ai/logo.png`)
     - **Description**: (예: `Personal AI Assistant`)
   - **Interactive features** 활성화.
   - **Functionality**에서 **Join spaces and group conversations** 체크.
   - **Connection settings**에서 **HTTP endpoint URL** 선택.
   - **Triggers**에서 **Use a common HTTP endpoint URL for all triggers**를 선택하고 Gateway 공개 URL 뒤에 `/googlechat`을 추가.
     - _팁: `openclaw status`를 실행하여 Gateway 공개 URL을 확인하세요._
   - **Visibility**에서 **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;** 체크.
   - 텍스트 박스에 이메일 주소 입력(예: `user@example.com`).
   - 하단의 **Save** 클릭.
6. **앱 상태 활성화**:
   - 저장 후 **페이지 새로고침**.
   - **App status** 섹션 찾기(보통 저장 후 상단 또는 하단 근처).
   - 상태를 **Live - available to users**로 변경.
   - 다시 **Save** 클릭.
7. 서비스 계정 경로 + webhook audience로 OpenClaw 구성:
   - 환경 변수: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - 또는 구성: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Webhook audience 유형 + 값 설정(Chat 앱 구성과 일치).
9. Gateway 시작. Google Chat이 webhook 경로로 POST 요청을 보냅니다.

## Google Chat에 추가

Gateway가 실행 중이고 이메일이 가시성 목록에 추가되면:

1. [Google Chat](https://chat.google.com/)으로 이동.
2. **Direct Messages** 옆의 **+**(더하기) 아이콘 클릭.
3. 검색창(보통 사람 추가용)에 Google Cloud Console에서 구성한 **App name** 입력.
   - **참고**: 봇은 "Marketplace" 탐색 목록에 나타나지 *않습니다*(비공개 앱이므로). 이름으로 검색해야 합니다.
4. 결과에서 봇 선택.
5. **Add** 또는 **Chat** 클릭하여 1:1 대화 시작.
6. "Hello"를 보내 어시스턴트 트리거!

## 공개 URL(Webhook 전용)

Google Chat webhook은 공개 HTTPS 엔드포인트가 필요합니다. 보안을 위해 **`/googlechat` 경로만** 인터넷에 노출하세요. OpenClaw 대시보드와 기타 민감한 엔드포인트는 비공개 네트워크에 유지하세요.

### 옵션 A: Tailscale Funnel(권장)

Tailscale Serve를 비공개 대시보드용으로, Funnel을 공개 webhook 경로용으로 사용합니다. 이렇게 하면 `/`는 비공개로 유지되고 `/googlechat`만 노출됩니다.

1. **Gateway가 어떤 주소에 바인딩되어 있는지 확인:**

   ```bash
   ss -tlnp | grep 18789
   ```

   IP 주소 확인(예: `127.0.0.1`, `0.0.0.0` 또는 Tailscale IP `100.x.x.x`).

2. **대시보드를 tailnet에만 노출(포트 8443):**

   ```bash
   # localhost(127.0.0.1 또는 0.0.0.0)에 바인딩된 경우:
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Tailscale IP에만 바인딩된 경우(예: 100.106.161.80):
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **webhook 경로만 공개 노출:**

   ```bash
   # localhost(127.0.0.1 또는 0.0.0.0)에 바인딩된 경우:
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Tailscale IP에만 바인딩된 경우(예: 100.106.161.80):
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **노드에 Funnel 접근 승인:**
   프롬프트가 나타나면 출력에 표시된 승인 URL을 방문하여 tailnet 정책에서 이 노드에 대해 Funnel을 활성화합니다.

5. **구성 확인:**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

공개 webhook URL:
`https://<node-name>.<tailnet>.ts.net/googlechat`

비공개 대시보드(tailnet 전용):
`https://<node-name>.<tailnet>.ts.net:8443/`

Google Chat 앱 구성에서 공개 URL(`:8443` 없이)을 사용하세요.

> 참고: 이 구성은 재부팅 후에도 유지됩니다. 나중에 제거하려면 `tailscale funnel reset`과 `tailscale serve reset`을 실행하세요.

### 옵션 B: 리버스 프록시(Caddy)

Caddy 같은 리버스 프록시를 사용하는 경우 특정 경로만 프록시:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

이 구성으로 `your-domain.com/`에 대한 요청은 무시되거나 404를 반환하고, `your-domain.com/googlechat`은 OpenClaw로 안전하게 라우팅됩니다.

### 옵션 C: Cloudflare Tunnel

터널 입력 규칙을 webhook 경로만 라우팅하도록 구성:

- **경로**: `/googlechat` -> `http://localhost:18789/googlechat`
- **기본 규칙**: HTTP 404(Not Found)

## 작동 방식

1. Google Chat이 Gateway에 webhook POST 요청을 보냅니다. 각 요청에는 `Authorization: Bearer <token>` 헤더가 포함됩니다.
2. OpenClaw가 구성된 `audienceType` + `audience`에 대해 토큰을 검증합니다:
   - `audienceType: "app-url"` → audience는 HTTPS webhook URL.
   - `audienceType: "project-number"` → audience는 Cloud 프로젝트 번호.
3. 메시지는 스페이스별로 라우팅:
   - DM은 세션 키 `agent:<agentId>:googlechat:dm:<spaceId>` 사용.
   - 스페이스는 세션 키 `agent:<agentId>:googlechat:group:<spaceId>` 사용.
4. DM 접근은 기본적으로 페어링이 필요합니다. 알 수 없는 발신자는 페어링 코드를 받으며, 다음으로 승인:
   - `openclaw pairing approve googlechat <code>`
5. 그룹 스페이스는 기본적으로 @멘션이 필요합니다. 멘션 감지에 앱 사용자 이름이 필요하면 `botUser`를 사용하세요.

## 대상

배달 및 허용 목록에 다음 식별자 사용:

- DM: `users/<userId>` 또는 `users/<email>`(이메일 주소 허용).
- 스페이스: `spaces/<spaceId>`.

## 구성 하이라이트

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 선택; 멘션 감지 보조
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

참고:

- 서비스 계정 자격 증명은 `serviceAccount`(JSON 문자열)를 통해 인라인으로 전달할 수도 있습니다.
- `audienceType`과 `audience`는 Chat 앱 구성과 일치해야 합니다.
- `botUser`는 그룹에서 멘션 감지에 도움이 됩니다.

## 기능

| 기능 | 상태 |
| --- | --- |
| DM | ✅ 지원 |
| 스페이스 | ✅ 지원 |
| 스레드 | ✅ 지원 |
| 미디어 | ✅ 지원 |
| 반응 | ✅ 지원(작업을 통해) |
| 타이핑 표시기 | ✅ 지원 |

## 문제 해결

- **Webhook이 연결되지 않음:** 서비스 계정 파일과 audience 구성 확인. Gateway 로그에서 인증 오류 확인.
- **메시지가 수신되지 않음:** Chat 앱 구성에서 webhook URL이 올바른지 확인. 앱 상태가 "Live"인지 확인.
- **권한 오류:** 서비스 계정에 필요한 권한이 있는지 확인. Chat API가 활성화되어 있는지 확인.
