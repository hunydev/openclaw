---
read_when:
  - 노드 클라이언트 (iOS/Android/macOS 노드 모드) 빌드 또는 디버깅 시
  - 페어링 또는 브릿지 인증 장애 해결 시
  - Gateway가 노출하는 노드 인터페이스 감사 시
summary: 브릿지 프로토콜 (레거시 노드): TCP JSONL, 페어링, 범위 지정된 RPC
title: 브릿지 프로토콜
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 789bcf3cbc6841fc293e054b919e63d661b3cc4cd205b2094289f00800127fe2
  source_path: gateway/bridge-protocol.md
  workflow: 14
---

# 브릿지 프로토콜 (레거시 노드 전송)

브릿지 프로토콜은 **레거시** 노드 전송 방식입니다 (TCP JSONL). 새 노드 클라이언트는 통합 Gateway WebSocket 프로토콜을 사용해야 합니다.

오퍼레이터 또는 노드 클라이언트를 구축 중이라면 [Gateway 프로토콜](/gateway/protocol)을 사용하세요.

**참고:** OpenClaw의 현재 버전은 더 이상 TCP 브릿지 리스너를 포함하지 않습니다; 이 문서는 역사적 참조로만 유지됩니다. 레거시 `bridge.*` 구성 키는 더 이상 구성 스키마의 일부가 아닙니다.

## 두 개의 프로토콜이 있는 이유

- **보안 경계**: 브릿지는 전체 Gateway API 표면이 아닌 작은 허용 목록만 노출합니다.
- **페어링 + 노드 ID**: 노드 허가는 Gateway가 관리하며 노드별 토큰에 바인딩됩니다.
- **발견 경험**: 노드는 LAN에서 Bonjour를 통해 Gateway를 발견하거나 tailnet을 통해 직접 연결할 수 있습니다.
- **local loopback WS**: 전체 WS 제어 평면은 SSH 터널을 통해 전달되지 않는 한 로컬에 유지됩니다.

## 전송

- TCP, 행당 하나의 JSON 객체 (JSONL).
- 선택적 TLS (`bridge.tls.enabled`가 true일 때).
- 레거시 기본 포트는 `18790`이었습니다 (현재 버전은 TCP 브릿지를 시작하지 않음).

TLS가 활성화되면 발견 TXT 레코드에 `bridgeTls=1`과 `bridgeTlsSha256`이 포함되어 노드가 인증서를 고정할 수 있습니다.

## 핸드셰이크 + 페어링

1. 클라이언트가 노드 메타데이터와 토큰 (페어링된 경우)과 함께 `hello` 전송.
2. 페어링되지 않은 경우 Gateway가 `error` (`NOT_PAIRED`/`UNAUTHORIZED`) 응답.
3. 클라이언트가 `pair-request` 전송.
4. Gateway가 승인을 기다린 후 `pair-ok` + `hello-ok` 전송.

`hello-ok`는 `serverName`을 반환하고 `canvasHostUrl`을 포함할 수 있습니다.

## 프레임 타입

클라이언트 → Gateway:

- `req` / `res`: 범위 지정된 Gateway RPC (채팅, 세션, 구성, 상태 검사, 음성 깨우기, skills.bins)
- `event`: 노드 신호 (음성 전사, 에이전트 요청, 채팅 구독, exec 수명 주기)

Gateway → 클라이언트:

- `invoke` / `invoke-res`: 노드 명령 (`canvas.*`, `camera.*`, `screen.record`, `location.get`, `sms.send`)
- `event`: 구독된 세션의 채팅 업데이트
- `ping` / `pong`: 킵얼라이브

레거시 허용 목록 적용 로직은 `src/gateway/server-bridge.ts`에 있었습니다 (제거됨).

## Exec 수명 주기 이벤트

노드는 `exec.finished` 또는 `exec.denied` 이벤트를 발생시켜 system.run 활동을 표시할 수 있습니다. 이것들은 Gateway에서 시스템 이벤트로 매핑됩니다. (레거시 노드는 여전히 `exec.started`를 발생시킬 수 있습니다.)

페이로드 필드 (별도로 명시되지 않으면 선택 사항):

- `sessionKey` (필수): 시스템 이벤트를 수신할 에이전트 세션.
- `runId`: 그룹화를 위한 고유 실행 ID.
- `command`: 원시 또는 포맷된 명령 문자열.
- `exitCode`, `timedOut`, `success`, `output`: 완료 세부 정보 (finished만).
- `reason`: 거부 사유 (denied만).

## Tailnet 사용법

- 브릿지를 tailnet IP에 바인딩: `~/.openclaw/openclaw.json`에서 `bridge.bind: "tailnet"` 설정.
- 클라이언트가 MagicDNS 이름 또는 tailnet IP를 통해 연결.
- Bonjour는 네트워크를 **넘지 않습니다**; 필요시 수동 호스트/포트 또는 광역 DNS-SD를 사용하세요.

## 버전 관리

브릿지는 현재 **암시적 v1**입니다 (최소/최대 버전 협상 없음). 하위 호환성 유지 예정; 호환되지 않는 변경을 도입하기 전에 브릿지 프로토콜 버전 필드를 추가해야 합니다.
