---
permalink: /security/formal-verification/
summary: OpenClaw의 가장 위험한 경로에 대한 기계 검증 보안 모델.
title: 정형 검증(보안 모델)
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 8dff6ea41a37fb6b870424e4e788015c3f8a6099075eece5dbf909883c045106
  source_path: gateway/security/formal-verification.md
  workflow: 14
---

# 정형 검증(보안 모델)

이 페이지는 OpenClaw의 **정형 보안 모델**을 추적합니다(현재 TLA+/TLC 사용; 필요에 따라 확장 예정).

> 참고: 일부 오래된 링크는 이전 프로젝트 이름을 참조할 수 있습니다.

**목표(북극성):** 명시된 가정 하에서 OpenClaw가 의도한 보안 정책(인가, 세션 격리, 도구 게이팅, 오설정 안전성)을 시행함을 기계 검증된 논증으로 제공합니다.

**현재 상태:** 실행 가능한, 공격자 주도 **보안 회귀 테스트 스위트**:

- 각 주장에는 유한 상태 공간에서 실행되는 모델 검사가 있습니다.
- 많은 주장에는 특정 종류의 현실적 취약점에 대한 반례 추적을 생성하는 **네거티브 모델**이 함께 제공됩니다.

**아직 아닌 것:** "OpenClaw는 모든 면에서 안전합니다"를 증명하거나 전체 TypeScript 구현의 정확성을 증명하지 않습니다.

## 모델 위치

모델은 별도의 리포지토리에서 유지됩니다: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## 중요 사항

- 이것들은 **모델**이지 전체 TypeScript 구현이 아닙니다. 모델과 코드 사이에 차이가 있을 수 있습니다.
- 결과는 TLC가 탐색하는 상태 공간에 의해 제한됩니다. "녹색"이 모델링 가정과 경계 밖에서도 안전하다는 것을 의미하지 않습니다.
- 일부 주장은 명시적인 환경 가정(예: 올바른 배포, 정확한 설정 입력)에 의존합니다.

## 결과 재현

현재 결과를 재현하려면 모델 리포지토리를 로컬에 클론하고 TLC를 실행해야 합니다(아래 참조). 향후 반복에서는 다음을 제공할 수 있습니다:

- CI에서 모델을 실행하고 아티팩트(반례 추적, 실행 로그)를 공개
- 소규모 바운드 검사를 위한 호스팅된 "이 모델 실행" 워크플로우

빠른 시작:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ 필요(TLC는 JVM에서 실행됨).
# 리포지토리에는 고정된 `tla2tools.jar`(TLA+ 도구)가 포함되어 있고 `bin/tlc` + Make 타겟을 제공합니다.

make <target>
```

### Gateway 노출 및 Open Gateway 오설정

**주장:** 인증 없이 비 local loopback에 바인딩하면 원격 침해 / 노출 면적이 증가할 수 있습니다. 토큰/비밀번호는 인가되지 않은 공격자를 차단할 수 있습니다(모델 가정 기반).

- 녹색 실행:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 빨간색(예상됨):
  - `make gateway-exposure-v2-negative`

참조: 모델 리포지토리의 `docs/gateway-exposure-matrix.md`.

### Nodes.run 파이프라인(가장 위험한 기능)

**주장:** `nodes.run`은 (a) 노드 명령 허용 목록과 선언된 명령, 그리고 (b) 구성 후 라이브 승인이 필요합니다. 승인은 토큰화를 통해 재생을 방지합니다(모델 내).

- 녹색 실행:
  - `make nodes-pipeline`
  - `make approvals-token`
- 빨간색(예상됨):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### 페어링 저장소(DM 게이팅)

**주장:** 페어링 요청은 TTL 및 대기 요청 상한을 따릅니다.

- 녹색 실행:
  - `make pairing`
  - `make pairing-cap`
- 빨간색(예상됨):
  - `make pairing-negative`
  - `make pairing-cap-negative`

### 인그레스 게이팅(멘션 + 제어 명령 우회)

**주장:** 멘션이 필요한 그룹 컨텍스트에서 인가되지 않은 "제어 명령"은 멘션 게이팅을 우회할 수 없습니다.

- 녹색:
  - `make ingress-gating`
- 빨간색(예상됨):
  - `make ingress-gating-negative`

### 라우팅/세션 키 격리

**주장:** 다른 피어의 DM은 명시적으로 연결/구성되지 않는 한 동일한 세션에 합쳐지지 않습니다.

- 녹색:
  - `make routing-isolation`
- 빨간색(예상됨):
  - `make routing-isolation-negative`

## v1++: 추가 바운드 모델(동시성, 재시도, 추적 정확성)

이것들은 실제 세계 실패 모드(비원자적 업데이트, 재시도, 메시지 팬아웃)에서 충실도를 높이기 위한 후속 모델입니다.

### 페어링 저장소 동시성 / 멱등성

**주장:** 페어링 저장소는 인터리브된 실행에서도 `MaxPending`과 멱등성을 강제해야 합니다(즉, "확인 후 쓰기"는 원자적/잠금이어야 합니다. 새로고침은 중복을 생성하면 안 됩니다).

의미:

- 동시 요청 하에서 채널의 `MaxPending` 상한을 초과할 수 없습니다.
- 동일한 `(channel, sender)`에 대한 중복 요청/새로고침은 중복된 활성 대기 행을 생성하면 안 됩니다.

- 녹색 실행:
  - `make pairing-race`(원자적/잠금된 상한 검사)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 빨간색(예상됨):
  - `make pairing-race-negative`(비원자적 begin/commit 상한 레이스)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### 인그레스 추적 상관관계 / 멱등성

**주장:** 인제스트는 팬아웃 전반에 걸쳐 추적 상관관계를 유지해야 하고 제공자 재시도 시 멱등성이 있어야 합니다.

의미:

- 하나의 외부 이벤트가 여러 내부 메시지가 될 때 각 조각은 동일한 추적/이벤트 식별자를 유지합니다.
- 재시도는 중복 처리를 일으키지 않습니다.
- 제공자 이벤트 ID가 없으면 중복 제거는 다른 이벤트를 버리지 않도록 안전 키(예: 추적 ID)로 대체됩니다.

- 녹색:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 빨간색(예상됨):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### 라우팅 dmScope 우선순위 + identityLinks

**주장:** 라우팅은 기본적으로 DM 세션 격리를 유지해야 하며, 명시적으로 구성된 경우에만 세션을 병합해야 합니다(채널 우선순위 + identity 링크).

의미:

- 채널별 dmScope 오버라이드는 전역 기본값보다 우선해야 합니다.
- identityLinks는 명시적으로 연결된 그룹 내에서만 병합해야 하며 관련 없는 피어 간에는 병합하지 않아야 합니다.

- 녹색:
  - `make routing-precedence`
  - `make routing-identitylinks`
- 빨간색(예상됨):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
