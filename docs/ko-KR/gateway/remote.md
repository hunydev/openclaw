---
read_when:
  - 원격 Gateway 설정 실행 또는 문제 해결 시
summary: SSH 터널 (Gateway WebSocket) 및 tailnet을 사용한 원격 액세스
title: 원격 액세스
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 7e00bd2e048dfbd829913bef0f40a791b8d8c3e2f8a115fc0a13b03f136ebc93
  source_path: gateway/remote.md
  workflow: 14
---

# 원격 액세스 (SSH, 터널 및 tailnet)

이 저장소는 전용 호스트 (데스크톱/서버)에서 단일 Gateway (메인 노드)를 실행하고 클라이언트가 연결하는 "SSH를 통한 원격 액세스"를 지원합니다.

- **운영자 (당신 / macOS 앱)**: SSH 터널이 범용 폴백.
- **노드 (iOS/Android 및 향후 장치)**: Gateway의 **WebSocket**에 연결 (필요에 따라 LAN/tailnet 또는 SSH 터널 통해).

## 핵심 아이디어

- Gateway WebSocket은 구성된 포트의 **local loopback**에 바인딩 (기본값 18789).
- 원격 사용의 경우 해당 local loopback 포트를 SSH로 포워딩 (또는 터널 필요를 줄이기 위해 tailnet/VPN 사용).

## 일반적인 VPN/tailnet 설정 (에이전트가 실행되는 곳)

**Gateway 호스트**를 "에이전트가 실행되는 곳"으로 생각하세요. 세션, 인증 구성, 채널 및 상태를 보유합니다.
노트북/데스크톱 (및 노드)은 해당 호스트에 연결합니다.

### 1) tailnet에서 상시 실행되는 Gateway (VPS 또는 홈 서버)

영구 호스트에서 Gateway를 실행하고 **Tailscale** 또는 SSH를 통해 액세스.

- **최상의 경험:** `gateway.bind: "loopback"` 유지하고 **Tailscale Serve**로 제어 UI 제공.
- **폴백:** local loopback 유지 + 액세스가 필요한 모든 컴퓨터에서 SSH 터널.
- **예시:** [exe.dev](/platforms/exe-dev) (간단한 VM) 또는 [Hetzner](/platforms/hetzner) (프로덕션 VPS).

노트북이 자주 절전되지만 에이전트가 항상 온라인이길 원할 때 이상적.

### 2) 홈 데스크톱에서 Gateway 실행, 노트북이 원격 제어

노트북은 에이전트를 **실행하지 않음**. 원격으로 연결:

- macOS 앱의 **SSH를 통한 원격 연결** 모드 사용 (설정 → 일반 → "OpenClaw 실행 위치").
- 앱이 터널을 열고 관리하므로 WebChat + 상태 확인이 "바로 작동".

가이드: [macOS 원격 액세스](/platforms/mac/remote).

### 3) 노트북에서 Gateway 실행, 다른 컴퓨터에서 원격 액세스

Gateway를 로컬에서 실행하지만 안전하게 노출:

- 다른 컴퓨터에서 노트북으로 SSH 터널, 또는
- Tailscale Serve로 제어 UI 제공하고 Gateway는 local loopback에만 바인딩 유지.

가이드: [Tailscale](/gateway/tailscale) 및 [Web 개요](/web).

## 명령 흐름 (무엇이 어디서 실행되나)

하나의 Gateway 서비스가 상태와 채널을 소유합니다. 노드는 주변 장치입니다.

흐름 예시 (Telegram → 노드):

- Telegram 메시지가 **Gateway**에 도착.
- Gateway가 **에이전트**를 실행하고 노드 도구 호출 여부 결정.
- Gateway가 Gateway WebSocket을 통해 **노드** 호출 (`node.*` RPC).
- 노드가 결과 반환; Gateway가 Telegram에 회신 전송.

설명:

- **노드는 Gateway 서비스를 실행하지 않음.** 격리된 프로필을 의도적으로 실행하지 않는 한 호스트당 하나의 Gateway만 실행해야 함 ([다중 Gateway](/gateway/multiple-gateways) 참조).
- macOS 앱 "노드 모드"는 Gateway WebSocket을 통해 연결하는 노드 클라이언트일 뿐.

## SSH 터널 (CLI + 도구)

원격 Gateway WebSocket으로의 로컬 터널 생성:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

터널 설정 후:

- `openclaw health`와 `openclaw status --deep`이 이제 `ws://127.0.0.1:18789`를 통해 원격 Gateway에 액세스.
- `openclaw gateway {status,health,send,agent,call}`도 필요시 `--url`로 포워딩된 URL을 가리킬 수 있음.

참고: `18789`를 구성된 `gateway.port` (또는 `--port`/`OPENCLAW_GATEWAY_PORT`)로 교체.

## CLI 원격 기본값

원격 대상을 영구화하여 CLI 명령이 기본적으로 사용하도록:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

Gateway가 local loopback에만 바인딩된 경우 URL을 `ws://127.0.0.1:18789`로 유지하고 먼저 SSH 터널을 열어야 함.

## SSH를 통한 채팅 UI 액세스

WebChat은 더 이상 별도의 HTTP 포트를 사용하지 않습니다. SwiftUI 채팅 UI가 Gateway WebSocket에 직접 연결합니다.

- `18789`를 SSH로 포워딩 (위 참조), 그런 다음 클라이언트를 `ws://127.0.0.1:18789`에 연결.
- macOS에서는 터널을 자동 관리하는 앱의 "SSH를 통한 원격 연결" 모드 사용 권장.

## macOS 앱 "SSH를 통한 원격 연결"

macOS 메뉴바 앱은 동일한 설정을 종단간으로 구동할 수 있습니다 (원격 상태 확인, WebChat 및 음성 웨이크 포워딩).

가이드: [macOS 원격 액세스](/platforms/mac/remote).

## 보안 규칙 (원격/VPN)

간단 버전: **Gateway를 local loopback에만 바인딩 유지**, 다른 바인딩이 정말 필요한 게 아니면.

- **local loopback + SSH/Tailscale Serve**가 가장 안전한 기본값 (공개 노출 없음).
- **비-local loopback 바인딩** (`lan`/`tailnet`/`custom`, 또는 local loopback 불가용 시 `auto`)은 인증 토큰/비밀번호 사용 필수.
- `gateway.remote.token`은 원격 CLI 호출에**만** 사용 — 로컬 인증을 활성화하지 **않음**.
- `gateway.remote.tlsFingerprint`는 `wss://` 사용 시 원격 TLS 인증서를 고정.
- **Tailscale Serve**는 `gateway.auth.allowTailscale: true`일 때 ID 헤더로 인증 가능. 토큰/비밀번호를 사용하려면 `false`로 설정.
- 브라우저 제어를 운영자 액세스로 취급: tailnet 전용 + 의도적인 노드 페어링.

심층 설명: [보안](/gateway/security).
