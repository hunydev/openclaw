---
read_when:
  - Gateway 프로세스 실행 또는 디버깅 시
  - 단일 인스턴스 강제 메커니즘 조사 시
summary: Gateway 싱글톤 가드: WebSocket 리스너 바인딩을 통해 구현
title: Gateway 잠금
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 15fdfa066d1925da8b4632073a876709f77ca8d40e6828c174a30d953ba4f8e9
  source_path: gateway/gateway-lock.md
  workflow: 14
---

# Gateway 잠금

최종 업데이트: 2025-12-11

## 이유

- 동일 호스트에서 각 기본 포트당 하나의 Gateway 인스턴스만 실행되도록 보장; 추가 Gateway는 격리된 프로필과 고유 포트를 사용해야 함.
- 충돌/SIGKILL 후에도 오래된 잠금 파일이 남지 않음.
- 제어 포트가 이미 점유되면 명확한 오류 메시지와 함께 빠르게 실패.

## 메커니즘

- Gateway는 시작 시 즉시 독점 TCP 리스너를 통해 WebSocket 리스너 (기본값 `ws://127.0.0.1:18789`)에 바인딩.
- 바인딩이 `EADDRINUSE`로 실패하면 시작 시 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`가 발생.
- OS가 충돌 및 SIGKILL을 포함하여 프로세스가 어떤 방식으로 종료되든 자동으로 리스너를 해제—별도의 잠금 파일이나 정리 단계 불필요.
- 종료 시 Gateway는 WebSocket 서버와 기본 HTTP 서버를 닫아 적시에 포트를 해제.

## 오류 표현

- 다른 프로세스가 포트를 점유 중이면 시작 시 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`가 발생.
- 다른 바인딩 실패는 `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`로 표현됨.

## 운영 참고사항

- 포트가 *다른* 프로세스에 의해 점유되면 오류 메시지가 동일; 포트를 해제하거나 `openclaw gateway --port <port>`로 다른 포트 선택.
- macOS 앱은 Gateway를 시작하기 전에 자체 경량 PID 가드를 여전히 유지; 런타임 잠금은 WebSocket 바인딩에 의해 강제됨.
