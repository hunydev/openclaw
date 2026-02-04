---
read_when:
  - WebChat 접근을 디버깅하거나 구성할 때
summary: 로컬 루프백 WebChat 정적 호스트 및 채팅 UI에서 Gateway WebSocket 사용
title: WebChat
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# WebChat(Gateway WebSocket UI)

상태: macOS/iOS SwiftUI 채팅 UI가 Gateway WebSocket에 직접 연결합니다.

## 소개

- Gateway용 네이티브 채팅 UI(임베디드 브라우저 없음, 로컬 정적 서버 없음).
- 다른 채널과 동일한 세션 및 라우팅 규칙 사용.
- 결정론적 라우팅: 응답은 항상 WebChat으로 돌아옵니다.

## 빠른 시작

1. Gateway를 시작합니다.
2. WebChat UI(macOS/iOS 앱) 또는 콘솔 UI의 채팅 탭을 엽니다.
3. Gateway 인증이 구성되어 있는지 확인합니다(로컬 루프백에서도 기본적으로 필요).

## 작동 방식(동작)

- UI는 Gateway WebSocket에 연결하여 `chat.history`, `chat.send`, `chat.inject`를 사용합니다.
- `chat.inject`는 어시스턴트 메모를 대화 기록에 직접 추가하고 UI에 브로드캐스트합니다(에이전트 실행 트리거 안 함).
- 기록은 항상 Gateway에서 가져옵니다(로컬 파일 감시 없음).
- Gateway에 연결할 수 없으면 WebChat은 읽기 전용 모드입니다.

## 원격 사용

- 원격 모드는 SSH/Tailscale 터널을 통해 Gateway WebSocket을 전송합니다.
- 별도의 WebChat 서버를 실행할 필요가 없습니다.

## 구성 참조(WebChat)

전체 구성: [구성](/gateway/configuration)

채널 옵션:

- 전용 `webchat.*` 구성 블록이 없습니다. WebChat은 아래의 Gateway 엔드포인트 + 인증 설정을 사용합니다.

관련 전역 옵션:

- `gateway.port`, `gateway.bind`: WebSocket 호스트/포트.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 인증.
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: 원격 Gateway 대상.
- `session.*`: 세션 저장소 및 메인 키 기본값.
