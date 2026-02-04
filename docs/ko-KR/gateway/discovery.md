---
read_when:
  - Bonjour 발견/브로드캐스트 기능 구현 또는 변경 시
  - 원격 연결 모드 조정 시 (직접 vs SSH)
  - 원격 노드를 위한 노드 발견 + 페어링 방식 설계 시
summary: 노드 발견 및 전송 (Bonjour, Tailscale, SSH)으로 Gateway 찾기
title: 발견 및 전송
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: e12172c181515bfa6aab8625ed3fbc335b80ba92e2b516c02c6066aeeb9f884c
  source_path: gateway/discovery.md
  workflow: 14
---

# 발견 및 전송

OpenClaw에는 표면적으로 비슷해 보이지만 실제로는 다른 두 가지 문제가 있습니다:

1. **운영자 원격 제어**: macOS 메뉴바 앱이 다른 곳에서 실행 중인 Gateway를 제어.
2. **노드 페어링**: iOS/Android (및 미래 노드)가 Gateway를 찾아 안전하게 페어링.

설계 목표는 모든 네트워크 발견/브로드캐스트를 **노드 Gateway** (`openclaw gateway`)에 유지하고 클라이언트 (Mac 앱, iOS)는 소비자로 두는 것입니다.

## 용어

- **Gateway**: 상태 (세션, 페어링, 노드 레지스트리)를 가지고 채널을 실행하는 장시간 실행 Gateway 프로세스. 대부분의 구성은 호스트당 하나를 사용합니다; 격리된 멀티 Gateway 설정도 가능합니다.
- **Gateway WS (제어 평면)**: 기본적으로 `127.0.0.1:18789`에 있는 WebSocket 엔드포인트; `gateway.bind`를 통해 LAN/Tailnet에 바인딩 가능.
- **직접 WS 전송**: Gateway WS 엔드포인트의 LAN/Tailnet 대면.
- **SSH 전송 (폴백)**: 원격 제어를 위해 SSH를 통해 `127.0.0.1:18789`를 터널링.
- **레거시 TCP 브릿지 (더 이상 사용 안 함/제거됨)**: 오래된 노드 전송 ([브릿지 프로토콜](/gateway/bridge-protocol) 참조); 더 이상 발견 브로드캐스트에 사용되지 않음.

프로토콜 세부 정보:

- [Gateway 프로토콜](/gateway/protocol)
- [브릿지 프로토콜 (레거시)](/gateway/bridge-protocol)

## "직접"과 SSH를 모두 유지하는 이유

- **직접 WS**는 동일 네트워크 및 Tailnet 내에서 최고의 UX를 제공합니다:
  - LAN에서 Bonjour를 통한 자동 발견
  - 페어링 토큰 + ACL은 Gateway가 관리
  - 셸 액세스 불필요; 프로토콜 인터페이스를 작게 유지하고 감사 가능
- **SSH**는 여전히 범용 폴백입니다:
  - SSH 액세스가 있는 모든 곳에서 작동 (관계없는 네트워크 간에도)
  - 멀티캐스트/mDNS 문제의 영향을 받지 않음
  - SSH 외에 새 인바운드 포트 불필요

## 발견 입력 (클라이언트가 Gateway 위치를 아는 방법)

### 1) Bonjour / mDNS (LAN 전용)

Bonjour는 최선의 노력이며 네트워크를 넘지 않습니다. "동일 LAN" 편의 발견에만 사용됩니다.

목표 방향:

- **Gateway**가 Bonjour를 통해 WS 엔드포인트를 브로드캐스트.
- 클라이언트가 탐색하여 "Gateway 선택" 목록을 표시하고 선택된 엔드포인트를 저장.

문제 해결 및 비컨 세부 정보: [Bonjour](/gateway/bonjour).

#### 서비스 비컨 세부 정보

- 서비스 타입:
  - `_openclaw-gw._tcp` (Gateway 전송 비컨)
- TXT 키 (비밀 아님):
  - `role=gateway`
  - `lanHost=<호스트명>.local`
  - `sshPort=22` (또는 실제 브로드캐스트된 포트)
  - `gatewayPort=18789` (Gateway WS + HTTP)
  - `gatewayTls=1` (TLS 활성화 시에만)
  - `gatewayTlsSha256=<sha256>` (TLS 활성화 시 및 지문 가용 시에만)
  - `canvasPort=18793` (기본 canvas 호스트 포트; `/__openclaw__/canvas/` 제공)
  - `cliPath=<경로>` (선택 사항; 실행 가능한 `openclaw` 진입점 또는 바이너리의 절대 경로)
  - `tailnetDns=<magicdns>` (선택적 힌트; Tailscale 가용 시 자동 감지)

비활성화/오버라이드:

- `OPENCLAW_DISABLE_BONJOUR=1`은 브로드캐스트를 비활성화합니다.
- `~/.openclaw/openclaw.json`의 `gateway.bind`는 Gateway 바인딩 모드를 제어합니다.
- `OPENCLAW_SSH_PORT`는 TXT에 브로드캐스트되는 SSH 포트를 오버라이드합니다 (기본값 22).
- `OPENCLAW_TAILNET_DNS`는 `tailnetDns` 힌트를 게시합니다 (MagicDNS).
- `OPENCLAW_CLI_PATH`는 브로드캐스트되는 CLI 경로를 오버라이드합니다.

### 2) Tailnet (네트워크 간)

런던/비엔나 스타일의 지역 간 배포에서 Bonjour는 도움이 되지 않습니다. 권장되는 "직접" 대상은:

- Tailscale MagicDNS 이름 (권장) 또는 안정적인 Tailnet IP.

Gateway가 Tailscale 아래에서 실행 중임을 감지하면 클라이언트 (광역 비컨 포함)에게 `tailnetDns`를 선택적 힌트로 게시합니다.

### 3) 수동 / SSH 대상

직접 라우트가 없거나 (또는 직접 연결이 비활성화된 경우) 클라이언트는 항상 SSH를 통해 local loopback Gateway 포트를 터널링하여 연결할 수 있습니다.

[원격 액세스](/gateway/remote) 참조.

## 전송 선택 (클라이언트 전략)

권장 클라이언트 동작:

1. 페어링된 직접 엔드포인트가 구성되어 있고 도달 가능하면 사용.
2. 그렇지 않으면, Bonjour가 LAN에서 Gateway를 발견하면 원클릭 "이 Gateway 사용" 옵션을 제공하고 직접 엔드포인트로 저장.
3. 그렇지 않으면, Tailnet DNS/IP가 구성되어 있으면 직접 연결 시도.
4. 그렇지 않으면, SSH로 폴백.

## 페어링 + 인증 (직접 전송)

Gateway는 노드/클라이언트 허가의 권위자입니다.

- 페어링 요청은 Gateway에서 생성/승인/거부됩니다 ([Gateway 페어링](/gateway/pairing) 참조).
- Gateway는 다음을 적용합니다:
  - 인증 (토큰 / 키페어)
  - 권한 범위/ACL (Gateway는 모든 메서드에 대한 원시 프록시가 아님)
  - 속도 제한

## 컴포넌트별 책임

- **Gateway**: 발견 비컨을 브로드캐스트하고 페어링 결정을 관리하며 WS 엔드포인트를 호스팅.
- **macOS 앱**: Gateway 선택을 돕고 페어링 프롬프트를 표시하며 SSH는 폴백으로만 사용.
- **iOS/Android 노드**: 편의를 위해 Bonjour 탐색을 사용하고 페어링된 Gateway WS에 연결.
