---
read_when:
  - 브라우저로 Gateway를 조작하려고 할 때
  - SSH 터널 없이 Tailnet 접근을 원할 때
summary: 브라우저 기반 Gateway 제어 인터페이스(채팅, 노드, 구성)
title: 제어 인터페이스
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 제어 인터페이스(브라우저)

제어 인터페이스는 Gateway가 제공하는 작은 **Vite + Lit** 단일 페이지 애플리케이션입니다:

- 기본 주소: `http://<host>:18789/`
- 선택적 접두사: `gateway.controlUi.basePath` 설정(예: `/openclaw`)

동일한 포트에서 **Gateway WebSocket과 직접** 통신합니다.

## 빠른 열기(로컬)

이 컴퓨터에서 Gateway가 실행 중이면 다음을 여세요:

- http://127.0.0.1:18789/(또는 http://localhost:18789/)

페이지가 로드되지 않으면 먼저 Gateway를 시작하세요: `openclaw gateway`.

인증은 WebSocket 핸드셰이크 중에 다음을 통해 제공됩니다:

- `connect.params.auth.token`
- `connect.params.auth.password`
  대시보드 설정 패널에서 토큰을 저장할 수 있습니다; 비밀번호는 지속되지 않습니다.
  온보딩 마법사가 기본적으로 Gateway 토큰을 생성하며, 첫 연결 시 여기에 붙여넣으세요.

## 장치 페어링(첫 연결)

새 브라우저나 장치에서 제어 인터페이스에 연결할 때, Gateway는 **일회성 페어링 승인**이 필요합니다 — 같은 Tailnet에 있고
`gateway.auth.allowTailscale: true`를 설정했더라도 마찬가지입니다. 이는 무단 접근을 방지하는 보안 조치입니다.

**표시되는 내용:** "disconnected (1008): pairing required"

**장치 승인:**

```bash
# 대기 중인 요청 나열
openclaw devices list

# 요청 ID로 승인
openclaw devices approve <requestId>
```

승인 후 장치는 기억되며 `openclaw devices revoke --device <id> --role <role>`로 취소하지 않는 한 재승인이 필요 없습니다. 토큰 교체 및 취소는
[장치 CLI](/cli/devices)를 참조하세요.

**참고:**

- 로컬 연결(`127.0.0.1`)은 자동으로 승인됩니다.
- 원격 연결(LAN, Tailnet 등)은 명시적 승인이 필요합니다.
- 각 브라우저 프로필은 고유한 장치 ID를 생성하므로 브라우저를 전환하거나
  브라우저 데이터를 지우면 재페어링이 필요합니다.

## 현재 기능

- Gateway WS를 통해 모델과 채팅(`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- 채팅에서 도구 호출 스트리밍 + 실시간 도구 출력 카드(에이전트 이벤트)
- 채널: WhatsApp/Telegram/Discord/Slack + 플러그인 채널(Mattermost 등) 상태 + QR 코드 로그인 + 채널별 구성(`channels.status`, `web.login.*`, `config.patch`)
- 인스턴스: 온라인 목록 + 새로고침(`system-presence`)
- 세션: 목록 + 세션별 사고/상세 모드 재정의(`sessions.list`, `sessions.patch`)
- 크론: 목록/추가/실행/활성화/비활성화 + 실행 기록(`cron.*`)
- Skills: 상태, 활성화/비활성화, 설치, API 키 업데이트(`skills.*`)
- 노드: 목록 + 기능(`node.list`)
- 실행 승인: Gateway 또는 노드 허용 목록 편집 + `exec host=gateway/node`의 요청 정책(`exec.approvals.*`)
- 구성: `~/.openclaw/openclaw.json` 보기/편집(`config.get`, `config.set`)
- 구성: 적용 + 유효성 검사와 함께 재시작(`config.apply`)하고 마지막 활성 세션 깨우기
- 구성 쓰기에는 동시 편집 덮어쓰기 방지를 위한 기본 해시 보호 포함
- 구성 스키마 + 폼 렌더링(`config.schema`, 플러그인 + 채널 스키마 포함); 원시 JSON 편집기도 사용 가능
- 디버그: 상태/건강/모델 스냅샷 + 이벤트 로그 + 수동 RPC 호출(`status`, `health`, `models.list`)
- 로그: 필터/내보내기가 있는 Gateway 파일 로그 실시간 추적(`logs.tail`)
- 업데이트: 패키지/git 업데이트 실행 + 재시작(`update.run`)과 재시작 보고서

## 채팅 동작

- `chat.send`는 **논블로킹**입니다: 즉시 `{ runId, status: "started" }`로 확인하고 응답은 `chat` 이벤트로 스트리밍됩니다.
- 동일한 `idempotencyKey`로 재전송하면 진행 중일 때 `{ status: "in_flight" }`, 완료 후 `{ status: "ok" }`를 반환합니다.
- `chat.inject`는 어시스턴트 메모를 세션 기록에 추가하고 UI 전용 업데이트를 위해 `chat` 이벤트를 브로드캐스트합니다(에이전트 실행을 트리거하지 않고 채널에 전달하지 않음).
- 중지:
  - **중지** 클릭(`chat.abort` 호출)
  - `/stop`(또는 `stop|esc|abort|wait|exit|interrupt`) 입력으로 대역 외 중단
  - `chat.abort`는 `{ sessionKey }`를 지원하여(`runId` 불필요) 해당 세션의 모든 활성 실행을 중단

## Tailnet 접근(권장)

### 통합 Tailscale Serve(선호)

Gateway를 로컬 루프백에 유지하고 Tailscale Serve가 HTTPS로 프록시:

```bash
openclaw gateway --tailscale serve
```

열기:

- `https://<magicdns>/`(또는 구성된 `gateway.controlUi.basePath`)

기본적으로 `gateway.auth.allowTailscale`이 `true`이면 Serve 요청은 Tailscale 신원 헤더(`tailscale-user-login`)로 인증할 수 있습니다. OpenClaw는
`tailscale whois`로 `x-forwarded-for` 주소를 해석하고 헤더와 일치시켜 신원을 확인하며, 요청이 Tailscale `x-forwarded-*` 헤더와 함께 로컬 루프백을 통해 도착할 때만 수락합니다. Serve 트래픽에도 토큰/비밀번호를 요구하려면
`gateway.auth.allowTailscale: false`를 설정하세요(또는 `gateway.auth.mode: "password"` 강제).

### Tailnet에 바인딩 + 토큰

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

그런 다음 열기:

- `http://<tailscale-ip>:18789/`(또는 구성된 `gateway.controlUi.basePath`)

UI 설정에서 토큰을 붙여넣으세요(`connect.params.auth.token`으로 전송).

## 불안전한 HTTP

평문 HTTP(`http://<lan-ip>` 또는 `http://<tailscale-ip>`)로 대시보드를 열면
브라우저가 **불안전한 컨텍스트**에서 실행되어 WebCrypto를 차단합니다. 기본적으로
OpenClaw는 장치 신원 없이 제어 인터페이스 연결을 **차단**합니다.

**권장 수정:** HTTPS(Tailscale Serve)를 사용하거나 로컬에서 UI를 여세요:

- `https://<magicdns>/`(Serve)
- `http://127.0.0.1:18789/`(Gateway 호스트에서)

**다운그레이드 예시(토큰 전용, HTTP를 통해):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

이렇게 하면 제어 인터페이스에서 장치 신원 + 페어링이 비활성화됩니다(HTTPS에서도). 네트워크를
신뢰할 때만 사용하세요.

HTTPS 설정 안내는 [Tailscale](/gateway/tailscale)을 참조하세요.

## UI 빌드

Gateway는 `dist/control-ui`에서 정적 파일을 제공합니다. 다음으로 빌드하세요:

```bash
pnpm ui:build # 첫 실행 시 UI 종속성 자동 설치
```

선택적 절대 기본 경로(고정 리소스 URL이 필요할 때):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

로컬 개발(독립 개발 서버):

```bash
pnpm ui:dev # 첫 실행 시 UI 종속성 자동 설치
```

그런 다음 UI를 Gateway WS URL(예: `ws://127.0.0.1:18789`)로 지정하세요.

## 디버그/테스트: 개발 서버 + 원격 Gateway

제어 인터페이스는 정적 파일입니다; WebSocket 대상은 구성 가능하며 HTTP 원본과 다를 수 있습니다. 로컬에서 Vite 개발 서버를 사용하지만 Gateway가 다른 곳에서 실행 중일 때 유용합니다.

1. UI 개발 서버 시작: `pnpm ui:dev`
2. 다음과 같은 URL 열기:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

선택적 일회성 인증(필요한 경우):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

참고:

- `gatewayUrl`은 로드 후 localStorage에 저장되고 URL에서 제거됩니다.
- `token`은 localStorage에 저장됩니다; `password`는 메모리에만 유지됩니다.
- Gateway가 TLS 뒤에 있을 때(Tailscale Serve, HTTPS 프록시 등) `wss://`를 사용하세요.

원격 접근 설정 세부 정보: [원격 접근](/gateway/remote).
