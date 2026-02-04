---
read_when:
  - macOS UI 없이 노드 페어링 승인 구현 시
  - 원격 노드 승인을 위한 CLI 흐름 추가 시
  - 노드 관리를 위해 Gateway 프로토콜 확장 시
summary: Gateway 소유 노드 페어링 (방안 B), iOS 및 기타 원격 노드용
title: Gateway 소유 페어링
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 1f5154292a75ea2c1470324babc99c6c46a5e4e16afb394ed323d28f6168f459
  source_path: gateway/pairing.md
  workflow: 14
---

# Gateway 소유 페어링 (방안 B)

Gateway 소유 페어링에서 **Gateway**가 어떤 노드가 참여할 수 있는지 결정하는 권위자입니다. UI (macOS 앱, 미래 클라이언트)는 대기 중인 요청을 승인하거나 거부하기 위한 프론트엔드일 뿐입니다.

**중요 참고:** WS 노드는 `connect` 중에 **장치 페어링** (역할 `node`)을 사용합니다. `node.pair.*`는 별도의 페어링 저장소이며 WS 핸드셰이크를 **제어하지 않습니다**. 명시적으로 `node.pair.*`를 호출하는 클라이언트만 이 흐름을 사용합니다.

## 개념

- **대기 중인 요청**: 노드가 참여를 요청하며 승인 필요.
- **페어링된 노드**: 승인된 노드, 발급된 인증 토큰 보유.
- **전송 계층**: Gateway WS 엔드포인트가 요청을 전달하지만 멤버십을 결정하지 않음. (레거시 TCP 브릿지 지원은 더 이상 사용 안 함/제거됨.)

## 페어링 작동 방식

1. 노드가 Gateway WS에 연결하고 페어링을 요청.
2. Gateway가 **대기 중인 요청**을 저장하고 `node.pair.requested` 이벤트 발생.
3. 요청을 승인하거나 거부 (CLI 또는 UI를 통해).
4. 승인되면 Gateway가 **새 토큰** 발급 (재페어링 시 토큰 교체).
5. 노드가 해당 토큰으로 재연결하면 "페어링됨" 상태.

대기 중인 요청은 **5분** 후 자동 만료됩니다.

## CLI 워크플로 (헤드리스 환경에 적합)

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status`는 페어링됨/연결됨 노드와 그 기능을 표시합니다.

## API 인터페이스 (Gateway 프로토콜)

이벤트:

- `node.pair.requested` — 새 대기 중인 요청 생성 시 발생.
- `node.pair.resolved` — 요청이 승인/거부/만료 시 발생.

메서드:

- `node.pair.request` — 대기 중인 요청 생성 또는 재사용.
- `node.pair.list` — 대기 중 및 페어링된 노드 목록.
- `node.pair.approve` — 대기 중인 요청 승인 (토큰 발급).
- `node.pair.reject` — 대기 중인 요청 거부.
- `node.pair.verify` — `{ nodeId, token }` 검증.

참고:

- `node.pair.request`는 노드당 멱등: 반복 호출은 동일한 대기 중인 요청 반환.
- 승인은 **항상** 새 토큰 생성; `node.pair.request`는 토큰을 반환하지 않음.
- 요청에 `silent: true`를 포함하여 자동 승인 흐름의 힌트로 사용 가능.

## 자동 승인 (macOS 앱)

macOS 앱은 다음 조건이 충족될 때 선택적으로 **무음 승인**을 시도할 수 있습니다:

- 요청이 `silent`로 표시됨, 그리고
- 앱이 동일한 사용자 검증으로 Gateway 호스트에 SSH 연결 가능.

무음 승인 실패 시 일반 "승인/거부" 프롬프트로 폴백.

## 저장소 (로컬, 비공개)

페어링 상태는 Gateway 상태 디렉토리 아래에 저장됩니다 (기본값 `~/.openclaw`):

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

`OPENCLAW_STATE_DIR`을 오버라이드하면 `nodes/` 폴더도 함께 이동합니다.

보안 참고:

- 토큰은 비밀입니다; `paired.json`을 민감한 파일로 취급하세요.
- 토큰 교체는 재승인 (또는 노드 항목 삭제)이 필요합니다.

## 전송 계층 동작

- 전송 계층은 **무상태**입니다; 멤버십 정보를 저장하지 않습니다.
- Gateway가 오프라인이거나 페어링이 비활성화되면 노드가 페어링할 수 없습니다.
- Gateway가 원격 모드이면 페어링은 여전히 원격 Gateway의 저장소를 기반으로 합니다.
