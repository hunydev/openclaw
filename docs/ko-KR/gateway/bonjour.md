---
read_when:
  - macOS/iOS에서 Bonjour 발견 문제 디버깅 시
  - mDNS 서비스 타입, TXT 레코드 또는 발견 관련 UX 변경 시
summary: Bonjour/mDNS 발견 + 디버깅 (Gateway 비컨, 클라이언트 및 일반적인 장애 모드)
title: Bonjour 발견
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 47569da55f0c0523bd5ff05275dc95ccb52f75638193cfbdb4eaaa162aadf08c
  source_path: gateway/bonjour.md
  workflow: 14
---

# Bonjour / mDNS 발견

OpenClaw는 Bonjour (mDNS / DNS-SD)를 **LAN 전용 편의 기능**으로 사용하여 활성 Gateway (WebSocket 엔드포인트)를 발견합니다. 이것은 최선의 노력으로 수행되며 SSH 또는 Tailnet 기반 연결을 **대체하지 않습니다**.

## Tailscale을 통한 광역 Bonjour (유니캐스트 DNS-SD)

노드와 Gateway가 다른 네트워크에 있으면 멀티캐스트 mDNS가 네트워크 경계를 넘을 수 없습니다. Tailscale 기반 **유니캐스트 DNS-SD** ("광역 Bonjour")로 전환하여 동일한 발견 경험을 유지할 수 있습니다.

개요 단계:

1. Gateway 호스트에서 DNS 서버 실행 (Tailnet을 통해 접근 가능).
2. 전용 존 아래에 `_openclaw-gw._tcp`의 DNS-SD 레코드 게시 (예: `openclaw.internal.`).
3. Tailscale **Split DNS**를 구성하여 선택한 도메인이 iOS를 포함한 클라이언트에서 해당 DNS 서버를 통해 해석되도록 합니다.

OpenClaw는 임의의 발견 도메인을 지원합니다; `openclaw.internal.`은 예시일 뿐입니다. iOS/Android 노드는 `local.`과 구성된 광역 도메인을 모두 탐색합니다.

### Gateway 구성 (권장)

```json5
{
  gateway: { bind: "tailnet" }, // tailnet 전용 (권장)
  discovery: { wideArea: { enabled: true } }, // 광역 DNS-SD 게시 활성화
}
```

### 일회성 DNS 서버 설정 (Gateway 호스트)

```bash
openclaw dns setup --apply
```

이 명령은 CoreDNS를 설치하고 다음과 같이 구성합니다:

- Gateway의 Tailscale 인터페이스에서만 53번 포트에서 리스닝
- `~/.openclaw/dns/<domain>.db`에서 선택한 도메인 제공 (예: `openclaw.internal.`)

Tailnet에 연결된 머신에서 확인:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS 설정

Tailscale 관리 콘솔에서:

- Gateway Tailnet IP를 가리키는 네임서버 추가 (UDP/TCP 53).
- 발견 도메인이 해당 네임서버를 사용하도록 Split DNS 추가.

클라이언트가 Tailnet DNS를 수락하면 iOS 노드가 멀티캐스트 없이 발견 도메인에서 `_openclaw-gw._tcp`를 탐색할 수 있습니다.

### Gateway 리스너 보안 (권장)

Gateway WS 포트 (기본값 `18789`)는 기본적으로 local loopback에 바인딩됩니다. LAN/Tailnet 액세스가 필요하면 명시적으로 바인딩하고 인증을 활성화된 상태로 유지하세요.

Tailnet 전용 설정의 경우:

- `~/.openclaw/openclaw.json`에서 `gateway.bind: "tailnet"` 설정.
- Gateway 재시작 (또는 macOS 메뉴바 앱 재시작).

## 브로드캐스터

Gateway만 `_openclaw-gw._tcp`를 브로드캐스트합니다.

## 서비스 타입

- `_openclaw-gw._tcp` — Gateway 전송 비컨 (macOS/iOS/Android 노드용).

## TXT 키 (비밀이 아닌 힌트)

Gateway는 UI 흐름을 용이하게 하기 위해 작은 비밀이 아닌 힌트를 브로드캐스트합니다:

- `role=gateway`
- `displayName=<친근한 이름>`
- `lanHost=<호스트명>.local`
- `gatewayPort=<포트>` (Gateway WS + HTTP)
- `gatewayTls=1` (TLS 활성화 시에만)
- `gatewayTlsSha256=<sha256>` (TLS 활성화 시 및 지문 가용 시에만)
- `canvasPort=<포트>` (canvas 호스트 활성화 시에만; 기본값 `18793`)
- `sshPort=<포트>` (오버라이드되지 않으면 기본값 22)
- `transport=gateway`
- `cliPath=<경로>` (선택 사항; 실행 가능한 `openclaw` 진입점의 절대 경로)
- `tailnetDns=<magicdns>` (Tailnet 가용 시 선택적 힌트)

## macOS에서 디버깅

유용한 내장 도구:

- 인스턴스 탐색:
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
- 단일 인스턴스 해석 (`<instance>` 교체):
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

탐색은 성공하지만 해석이 실패하면 일반적으로 LAN 정책 또는 mDNS 리졸버 문제입니다.

## Gateway 로그에서 디버깅

Gateway는 롤링 로그 파일을 작성합니다 (시작 시 `gateway log file: ...`로 출력). `bonjour:` 행을 찾으세요, 특히:

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`

## iOS 노드에서 디버깅

iOS 노드는 `NWBrowser`를 사용하여 `_openclaw-gw._tcp`를 발견합니다.

로그를 얻으려면:

- 설정 → Gateway → 고급 → **발견 디버그 로그**
- 설정 → Gateway → 고급 → **발견 로그** → 문제 재현 → **복사**

로그에는 브라우저 상태 전환 및 결과 집합 변경이 포함됩니다.

## 일반적인 장애 모드

- **Bonjour는 네트워크를 넘지 않음**: Tailnet 또는 SSH를 사용하세요.
- **멀티캐스트가 차단됨**: 일부 Wi-Fi 네트워크가 mDNS를 비활성화합니다.
- **슬립 / 인터페이스 변경**: macOS가 일시적으로 mDNS 결과를 잃을 수 있음; 재시도하세요.
- **탐색 성공하지만 해석 실패**: 머신 이름을 단순하게 유지 (이모지나 구두점 피함)하고 Gateway를 재시작하세요. 서비스 인스턴스 이름은 호스트명에서 유래하므로 너무 복잡한 이름은 일부 리졸버를 혼란스럽게 할 수 있습니다.

## 이스케이프된 인스턴스 이름 (`\032`)

Bonjour/DNS-SD는 서비스 인스턴스 이름의 바이트를 십진수 `\DDD` 시퀀스로 이스케이프합니다 (예: 공백이 `\032`가 됨).

- 이것은 프로토콜 수준에서 정상입니다.
- UI는 표시 전에 디코딩해야 합니다 (iOS는 `BonjourEscapes.decode` 사용).

## 비활성화 / 구성

- `OPENCLAW_DISABLE_BONJOUR=1`은 브로드캐스트를 비활성화합니다 (레거시: `OPENCLAW_DISABLE_BONJOUR`).
- `~/.openclaw/openclaw.json`의 `gateway.bind`는 Gateway 바인딩 모드를 제어합니다.
- `OPENCLAW_SSH_PORT`는 TXT에 브로드캐스트되는 SSH 포트를 오버라이드합니다 (레거시: `OPENCLAW_SSH_PORT`).
- `OPENCLAW_TAILNET_DNS`는 TXT에 MagicDNS 힌트를 게시합니다 (레거시: `OPENCLAW_TAILNET_DNS`).
- `OPENCLAW_CLI_PATH`는 브로드캐스트되는 CLI 경로를 오버라이드합니다 (레거시: `OPENCLAW_CLI_PATH`).

## 관련 문서

- 발견 전략 및 전송 선택: [발견](/gateway/discovery)
- 노드 페어링 + 승인: [Gateway 페어링](/gateway/pairing)
