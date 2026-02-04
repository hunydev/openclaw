---
read_when:
  - 대시보드 인증 또는 노출 모드를 변경할 때
summary: Gateway 대시보드(제어 UI) 접근 및 인증
title: 대시보드
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 대시보드(제어 UI)

Gateway 대시보드는 `/` 경로에서 기본으로 제공되는 브라우저 제어 UI입니다(`gateway.controlUi.basePath`로 재정의 가능).

빠른 열기(로컬 Gateway):

- http://127.0.0.1:18789/(또는 http://localhost:18789/)

주요 참조:

- [제어 UI](/web/control-ui)에서 사용법과 UI 기능.
- [Tailscale](/gateway/tailscale)에서 Serve/Funnel 자동화.
- [웹 인터페이스](/web)에서 바인딩 모드와 보안 참고.

인증은 WebSocket 핸드셰이크 시 `connect.params.auth`(토큰 또는 비밀번호)를 통해 적용됩니다. [Gateway 구성](/gateway/configuration)의 `gateway.auth`를 참조하세요.

보안 참고: 제어 UI는 **관리 인터페이스**입니다(채팅, 구성, 실행 승인). 공개적으로 노출하지 마세요. UI는 첫 로드 후 토큰을 `localStorage`에 저장합니다. localhost, Tailscale Serve 또는 SSH 터널을 권장합니다.

## 빠른 경로(권장)

- 온보딩 완료 후 CLI가 토큰과 함께 대시보드를 자동으로 열고 동일한 토큰 포함 링크를 출력합니다.
- 언제든지 다시 열기: `openclaw dashboard`(링크 복사, 가능하면 브라우저 열기, 헤드리스 환경에서는 SSH 힌트 표시).
- 토큰은 로컬에 유지됩니다(쿼리 매개변수로만); UI는 첫 로드 후 이를 제거하고 localStorage에 저장합니다.

## 토큰 기초(로컬 vs 원격)

- **Localhost**: `http://127.0.0.1:18789/`를 엽니다. "unauthorized"가 표시되면 `openclaw dashboard`를 실행하고 토큰 포함 링크(`?token=...`)를 사용하세요.
- **토큰 출처**: `gateway.auth.token`(또는 `OPENCLAW_GATEWAY_TOKEN`); UI는 첫 로드 후 이를 저장합니다.
- **비 localhost**: Tailscale Serve(`gateway.auth.allowTailscale: true`일 때 토큰 불필요), 토큰이 있는 tailnet 바인딩 또는 SSH 터널을 사용하세요. [웹 인터페이스](/web)를 참조하세요.

## "unauthorized"/ 1008이 표시되면

- `openclaw dashboard`를 실행하여 새 토큰 포함 링크를 받으세요.
- Gateway에 연결할 수 있는지 확인하세요(로컬: `openclaw status`; 원격: SSH 터널 `ssh -N -L 18789:127.0.0.1:18789 user@host`, 그런 다음 `http://127.0.0.1:18789/?token=...` 열기).
- 대시보드 설정에서 `gateway.auth.token`(또는 `OPENCLAW_GATEWAY_TOKEN`)에 구성한 동일한 토큰을 붙여넣으세요.
