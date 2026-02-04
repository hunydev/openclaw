---
read_when:
  - Gateway WebSocket 클라이언트 구현 또는 업데이트 시
  - 프로토콜 불일치 또는 연결 실패 디버깅 시
  - 프로토콜 schema/모델 재생성 시
summary: Gateway WebSocket 프로토콜: 핸드셰이크, 프레임, 버전 관리
title: Gateway 프로토콜
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: bdafac40d53565901b2df450617287664d77fe4ff52681fa00cab9046b2fd850
  source_path: gateway/protocol.md
  workflow: 14
---

# Gateway 프로토콜 (WebSocket)

Gateway WebSocket 프로토콜은 OpenClaw의 **통합 제어 평면 + 노드 전송 계층**입니다. 모든 클라이언트 (CLI, Web UI, macOS 앱, iOS/Android 노드, 헤드리스 노드)가 WebSocket으로 연결하고 핸드셰이크 시 **역할** + **범위**를 선언합니다.

## 전송

- WebSocket, JSON 페이로드를 담은 텍스트 프레임.
- 첫 번째 프레임은 **반드시** `connect` 요청이어야 함.

## 핸드셰이크 (connect)

Gateway → 클라이언트 (연결 전 챌린지):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

클라이언트 → Gateway:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway → 클라이언트:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

장치 토큰이 발급되면 `hello-ok`에 다음도 포함됩니다:

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### 노드 예제

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## 프레임 형식

- **요청**: `{type:"req", id, method, params}`
- **응답**: `{type:"res", id, ok, payload|error}`
- **이벤트**: `{type:"event", event, payload, seq?, stateVersion?}`

부작용이 있는 메서드는 **멱등 키**가 필요합니다 (schema 참조).

## 역할 + 범위

### 역할

- `operator` = 제어 평면 클라이언트 (CLI/UI/자동화).
- `node` = 기능 호스트 (카메라/화면/캔버스/system.run).

### 범위 (operator)

일반적인 범위:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

### 기능/명령/권한 (node)

노드는 연결 시 기능을 선언합니다:

- `caps`: 상위 수준 기능 카테고리.
- `commands`: 호출 허용된 명령 화이트리스트.
- `permissions`: 세분화된 스위치 (예: `screen.record`, `camera.capture`).

Gateway는 이것들을 **선언**으로 취급하고 서버 측에서 화이트리스트를 적용합니다.

## 온라인 상태

- `system-presence`는 장치 ID를 키로 하는 항목을 반환합니다.
- 온라인 상태 항목에는 `deviceId`, `roles`, `scopes`가 포함되어 UI가 장치당 하나의 행을 표시할 수 있습니다. 해당 장치가 **operator**와 **node** 역할을 모두 가지고 연결되어 있어도 마찬가지입니다.

### 노드 헬퍼 메서드

- 노드는 `skills.bins`를 호출하여 현재 Skills 실행 파일 목록을 가져와 자동 허용 확인에 사용할 수 있습니다.

## 실행 승인

- 실행 요청이 승인을 필요로 하면 Gateway가 `exec.approval.requested`를 브로드캐스트합니다.
- Operator 클라이언트는 `exec.approval.resolve`를 호출하여 처리합니다 (`operator.approvals` 범위 필요).

## 버전 관리

- `PROTOCOL_VERSION`은 `src/gateway/protocol/schema.ts`에 정의됩니다.
- 클라이언트가 `minProtocol` + `maxProtocol` 전송; 서버가 불일치 시 연결 거부.
- Schema와 모델은 TypeBox 정의에서 생성됩니다:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

## 인증

- `OPENCLAW_GATEWAY_TOKEN` (또는 `--token`)이 설정되면 `connect.params.auth.token`이 일치해야 하며 그렇지 않으면 연결이 닫힙니다.
- 페어링 완료 후 Gateway가 연결된 역할 + 범위로 범위가 지정된 **장치 토큰**을 발급합니다. 이 토큰은 `hello-ok.auth.deviceToken`에 반환되며 클라이언트는 후속 연결을 위해 이를 영구 저장해야 합니다.
- 장치 토큰은 `device.token.rotate` 및 `device.token.revoke`를 통해 교체/철회할 수 있습니다 (`operator.pairing` 범위 필요).

## 장치 ID + 페어링

- 노드는 키페어 지문에서 파생된 안정적인 장치 ID (`device.id`)를 포함해야 합니다.
- Gateway는 장치 + 역할별로 토큰을 발급합니다.
- 새 장치 ID는 로컬 자동 승인이 활성화되지 않은 한 페어링 승인이 필요합니다.
- **로컬** 연결에는 local loopback과 Gateway 호스트 자체의 tailnet 주소가 포함됩니다 (따라서 동일 호스트의 tailnet 바인딩도 자동 승인 가능).
- 모든 WebSocket 클라이언트는 `connect` 시 `device` ID를 포함해야 합니다 (operator + node). 제어 UI는 `gateway.controlUi.allowInsecureAuth`가 활성화된 경우에만 생략 가능 (또는 긴급 조치로 `gateway.controlUi.dangerouslyDisableDeviceAuth` 사용).
- 비로컬 연결은 서버가 제공한 `connect.challenge` nonce에 서명해야 합니다.

## TLS + 인증서 고정

- WebSocket 연결은 TLS를 지원합니다.
- 클라이언트는 선택적으로 Gateway 인증서 지문을 고정할 수 있습니다 (`gateway.tls` 구성 및 `gateway.remote.tlsFingerprint` 또는 CLI `--tls-fingerprint` 참조).

## 범위

이 프로토콜은 **전체 Gateway API** (상태, 채널, 모델, 채팅, 에이전트, 세션, 노드, 승인 등)를 노출합니다. 정확한 API 인터페이스는 `src/gateway/protocol/schema.ts`의 TypeBox schema에 의해 정의됩니다.
