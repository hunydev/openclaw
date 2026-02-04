---
read_when:
  - Gateway 제어 인터페이스를 localhost 외부로 노출 시
  - tailnet 또는 공개 대시보드 액세스 자동화 시
summary: Gateway 대시보드를 위한 Tailscale Serve/Funnel 통합
title: Tailscale
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: c900c70a9301f2909a3a29a6fb0e6edfc8c18dba443f2e71b9cfadbc58167911
  source_path: gateway/tailscale.md
  workflow: 14
---

# Tailscale (Gateway 대시보드)

OpenClaw는 Gateway 대시보드와 WebSocket 포트를 위해 Tailscale **Serve** (tailnet) 또는 **Funnel** (공개 인터넷)을 자동으로 구성할 수 있습니다. 이렇게 하면 Gateway가 local loopback에 바인딩된 상태로 유지되면서 Tailscale이 HTTPS, 라우팅 및 (Serve의 경우) ID 헤더를 제공합니다.

## 모드

- `serve`: `tailscale serve`를 통한 Tailnet 전용 Serve. Gateway는 `127.0.0.1`에 유지.
- `funnel`: `tailscale funnel`을 통한 공개 인터넷 HTTPS. OpenClaw는 공유 비밀번호 설정을 요구.
- `off`: 기본값 (Tailscale 자동화 없음).

## 인증

핸드셰이크 방식을 제어하려면 `gateway.auth.mode` 설정:

- `token` (`OPENCLAW_GATEWAY_TOKEN` 설정 시 기본값)
- `password` (`OPENCLAW_GATEWAY_PASSWORD` 또는 구성으로 설정된 공유 비밀)

`tailscale.mode = "serve"`이고 `gateway.auth.allowTailscale`이 `true`이면 유효한 Serve 프록시 요청이 토큰/비밀번호 없이 Tailscale ID 헤더 (`tailscale-user-login`)를 통해 인증될 수 있습니다. OpenClaw는 로컬 Tailscale 데몬 (`tailscale whois`)을 통해 `x-forwarded-for` 주소를 해석하고 헤더와 일치시켜 ID를 검증합니다. OpenClaw는 요청이 local loopback에서 오고 Tailscale의 `x-forwarded-for`, `x-forwarded-proto`, `x-forwarded-host` 헤더가 있을 때만 Serve 요청으로 취급합니다.
명시적 자격 증명을 강제하려면 `gateway.auth.allowTailscale: false`를 설정하거나 `gateway.auth.mode: "password"`를 강제 지정하세요.

## 구성 예제

### Tailnet 전용 (Serve)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

열기: `https://<magicdns>/` (또는 구성된 `gateway.controlUi.basePath`)

### Tailnet 전용 (Tailnet IP에 바인딩)

Gateway가 Tailnet IP에서 직접 리스닝하길 원할 때 사용 (Serve/Funnel 없이).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

다른 Tailnet 장치에서 연결:

- 제어 인터페이스: `http://<tailscale-ip>:18789/`
- WebSocket: `ws://<tailscale-ip>:18789`

참고: 이 모드에서 local loopback (`http://127.0.0.1:18789`)은 **사용 불가**.

### 공개 인터넷 (Funnel + 공유 비밀번호)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

디스크에 비밀번호를 커밋하는 대신 `OPENCLAW_GATEWAY_PASSWORD` 사용 권장.

## CLI 예제

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 참고 사항

- Tailscale Serve/Funnel은 `tailscale` CLI가 설치되고 로그인되어 있어야 합니다.
- `tailscale.mode: "funnel"`은 인증 모드가 `password`가 아니면 공개 노출을 방지하기 위해 시작을 거부합니다.
- OpenClaw가 종료 시 `tailscale serve` 또는 `tailscale funnel` 구성을 철회하길 원하면 `gateway.tailscale.resetOnExit`를 설정하세요.
- `gateway.bind: "tailnet"`은 직접 Tailnet 바인딩입니다 (HTTPS 없음, Serve/Funnel 없음).
- `gateway.bind: "auto"`는 local loopback을 선호합니다; Tailnet만 원하면 `tailnet`을 사용하세요.
- Serve/Funnel은 **Gateway 제어 인터페이스 + WS**만 노출합니다. 노드는 동일한 Gateway WS 엔드포인트를 통해 연결하므로 Serve가 노드 액세스에 작동합니다.

## 브라우저 제어 (원격 Gateway + 로컬 브라우저)

한 머신에서 Gateway를 실행하지만 다른 머신에서 브라우저를 구동하려면 브라우저가 있는 머신에서 **노드 호스트**를 실행하고 둘을 동일한 tailnet에 유지하세요. Gateway가 브라우저 작업을 노드로 프록시합니다; 별도의 제어 서버나 Serve URL이 필요 없습니다.

브라우저 제어에 Funnel 사용을 피하세요; 노드 페어링을 운영자 수준 액세스로 취급하세요.

## Tailscale 전제 조건 + 제한 사항

- Serve는 tailnet에서 HTTPS가 활성화되어 있어야 합니다; 활성화되지 않으면 CLI가 프롬프트를 표시합니다.
- Serve는 Tailscale ID 헤더를 주입합니다; Funnel은 그렇지 않습니다.
- Funnel은 Tailscale v1.38.3+, MagicDNS, 활성화된 HTTPS 및 funnel 노드 속성이 필요합니다.
- Funnel은 TLS를 통해 포트 `443`, `8443`, `10000`만 지원합니다.
- macOS의 Funnel은 Tailscale 앱의 오픈 소스 버전이 필요합니다.

## 더 알아보기

- Tailscale Serve 개요: https://tailscale.com/kb/1312/serve
- `tailscale serve` 명령: https://tailscale.com/kb/1242/tailscale-serve
- Tailscale Funnel 개요: https://tailscale.com/kb/1223/tailscale-funnel
- `tailscale funnel` 명령: https://tailscale.com/kb/1311/tailscale-funnel
