---
read_when:
  - Tailscale을 통해 Gateway에 접근하려고 할 때
  - 브라우저 제어 UI 및 구성 편집을 사용하려고 할 때
summary: Gateway 웹 인터페이스: 제어 UI, 바인딩 모드 및 보안
title: Web
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Web(Gateway)

Gateway는 Gateway WebSocket과 동일한 포트에서 작은 **브라우저 제어 UI**(Vite + Lit)를 제공합니다:

- 기본값: `http://<host>:18789/`
- 선택적 접두사: `gateway.controlUi.basePath` 설정(예: `/openclaw`)

기능 세부 정보는 [제어 UI](/web/control-ui)를 참조하세요.
이 페이지는 바인딩 모드, 보안 및 웹 대면 인터페이스에 중점을 둡니다.

## Webhook

`hooks.enabled=true`이면 Gateway는 동일한 HTTP 서버에서 작은 webhook 엔드포인트도 노출합니다.
인증 및 페이로드는 [Gateway 구성](/gateway/configuration) → `hooks`를 참조하세요.

## 구성(기본적으로 활성화)

리소스 파일이 존재하면(`dist/control-ui`) 제어 UI는 **기본적으로 활성화**됩니다.
구성을 통해 제어할 수 있습니다:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath는 선택 사항
  },
}
```

## Tailscale 접근

### 통합 Serve(권장)

Gateway를 로컬 루프백에 유지하고 Tailscale Serve가 프록시하게 합니다:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

그런 다음 Gateway를 시작합니다:

```bash
openclaw gateway
```

열기:

- `https://<magicdns>/`(또는 구성된 `gateway.controlUi.basePath`)

### Tailnet 바인딩 + 토큰

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

그런 다음 Gateway를 시작합니다(비 로컬 루프백 바인딩에는 토큰 필요):

```bash
openclaw gateway
```

열기:

- `http://<tailscale-ip>:18789/`(또는 구성된 `gateway.controlUi.basePath`)

### 공개 접근(Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // 또는 OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## 보안 참고

- Gateway 인증은 기본적으로 필수입니다(토큰/비밀번호 또는 Tailscale 신원 헤더).
- 비 로컬 루프백 바인딩은 여전히 공유 토큰/비밀번호가 **필요합니다**(`gateway.auth` 또는 환경 변수).
- 마법사는 기본적으로 Gateway 토큰을 생성합니다(로컬 루프백에서도).
- UI는 `connect.params.auth.token` 또는 `connect.params.auth.password`를 보냅니다.
- Serve를 사용할 때 `gateway.auth.allowTailscale`이 `true`이면 Tailscale 신원 헤더가 인증 요구 사항을 충족합니다(토큰/비밀번호 불필요). 명시적 자격 증명을 요구하려면 `gateway.auth.allowTailscale: false`를 설정하세요. [Tailscale](/gateway/tailscale) 및 [보안](/gateway/security)을 참조하세요.
- `gateway.tailscale.mode: "funnel"`은 `gateway.auth.mode: "password"`가 필요합니다(공유 비밀번호).

## UI 빌드

Gateway는 `dist/control-ui`에서 정적 파일을 제공합니다. 다음으로 빌드하세요:

```bash
pnpm ui:build # 첫 실행 시 UI 종속성 자동 설치
```
