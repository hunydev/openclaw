---
read_when:
  - TUI의 초보자 친화적인 가이드가 필요할 때
  - TUI 기능, 명령 및 단축키의 전체 목록이 필요할 때
summary: 터미널 UI(TUI): 어디서든 Gateway에 연결
title: TUI
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# TUI(터미널 UI)

## 빠른 시작

1. Gateway를 시작합니다.

```bash
openclaw gateway
```

2. TUI를 엽니다.

```bash
openclaw tui
```

3. 메시지를 입력하고 Enter를 누릅니다.

원격 Gateway:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Gateway가 비밀번호 인증을 사용하는 경우 `--password`를 사용하세요.

## 인터페이스 개요

- 헤더: 연결 URL, 현재 에이전트, 현재 세션.
- 채팅 기록: 사용자 메시지, 어시스턴트 응답, 시스템 알림, 도구 카드.
- 상태 줄: 연결/실행 상태(연결 중, 실행 중, 스트리밍 중, 유휴, 오류).
- 하단 바: 연결 상태 + 에이전트 + 세션 + 모델 + 사고/상세/추론 + 토큰 수 + 전달 상태.
- 입력 상자: 자동 완성이 있는 텍스트 편집기.

## 개념 모델: 에이전트 + 세션

- 에이전트는 고유 식별자입니다(예: `main`, `research`). Gateway가 사용 가능한 목록을 제공합니다.
- 세션은 현재 에이전트에 속합니다.
- 세션 키는 `agent:<agentId>:<sessionKey>`로 저장됩니다.
  - `/session main`을 입력하면 TUI가 `agent:<currentAgent>:main`으로 확장합니다.
  - `/session agent:other:main`을 입력하면 해당 에이전트 세션으로 명시적으로 전환합니다.
- 세션 범위:
  - `per-sender`(기본값): 에이전트당 여러 세션.
  - `global`: TUI는 항상 `global` 세션을 사용합니다(선택기가 비어 있을 수 있음).
- 현재 에이전트 + 세션은 항상 하단 바에 표시됩니다.

## 전송 및 전달

- 메시지는 Gateway로 전송됩니다; 기본적으로 제공자에게 전달되지 않습니다.
- 전달 켜기:
  - `/deliver on`
  - 또는 설정 패널을 통해
  - 또는 시작 시 `openclaw tui --deliver`

## 선택기와 오버레이

- 모델 선택기: 사용 가능한 모델을 나열하고 세션 재정의를 설정합니다.
- 에이전트 선택기: 다른 에이전트를 선택합니다.
- 세션 선택기: 현재 에이전트의 세션만 표시합니다.
- 설정: 전달, 도구 출력 확장 및 사고 가시성을 토글합니다.

## 키보드 단축키

- Enter: 메시지 전송
- Esc: 활성 실행 중단
- Ctrl+C: 입력 지우기(두 번 누르면 종료)
- Ctrl+D: 종료
- Ctrl+L: 모델 선택기
- Ctrl+G: 에이전트 선택기
- Ctrl+P: 세션 선택기
- Ctrl+O: 도구 출력 확장 토글
- Ctrl+T: 사고 가시성 토글(기록 다시 로드)

## 슬래시 명령

핵심:

- `/help`
- `/status`
- `/agent <id>`(또는 `/agents`)
- `/session <key>`(또는 `/sessions`)
- `/model <provider/model>`(또는 `/models`)

세션 제어:

- `/think <off|minimal|low|medium|high>`
- `/verbose <on|full|off>`
- `/reasoning <on|off|stream>`
- `/usage <off|tokens|full>`
- `/elevated <on|off|ask|full>`(별칭: `/elev`)
- `/activation <mention|always>`
- `/deliver <on|off>`

세션 생명주기:

- `/new` 또는 `/reset`(세션 초기화)
- `/abort`(활성 실행 중단)
- `/settings`
- `/exit`

다른 Gateway 슬래시 명령(예: `/context`)은 Gateway로 전달되어 시스템 출력으로 표시됩니다. [슬래시 명령](/tools/slash-commands)을 참조하세요.

## 로컬 shell 명령

- 줄 시작에 `!`를 붙이면 TUI 호스트에서 로컬 shell 명령을 실행합니다.
- TUI는 세션당 한 번 로컬 실행을 허용하는지 묻습니다; 거부하면 해당 세션에서 `!`가 비활성화됩니다.
- 명령은 TUI 작업 디렉토리에서 새로운 비대화형 shell에서 실행됩니다(지속적인 `cd`/env 없음).
- 단독 `!`는 일반 메시지로 전송됩니다; 선행 공백은 로컬 실행을 트리거하지 않습니다.

## 도구 출력

- 도구 호출은 매개변수와 결과가 포함된 카드로 표시됩니다.
- Ctrl+O로 접힘/확장 보기 사이를 전환합니다.
- 도구가 실행되는 동안 부분 업데이트가 같은 카드로 스트리밍됩니다.

## 기록 및 스트리밍

- 연결 시 TUI는 최신 기록을 로드합니다(기본값 200개 메시지).
- 스트리밍 응답은 최종 확정될 때까지 실시간으로 업데이트됩니다.
- TUI는 더 풍부한 도구 카드를 위해 에이전트 도구 이벤트도 수신합니다.

## 연결 세부 정보

- TUI는 `mode: "tui"`로 Gateway에 등록합니다.
- 재연결 시 시스템 메시지가 표시됩니다; 이벤트 갭은 로그에 표시됩니다.

## 옵션

- `--url <url>`: Gateway WebSocket URL(기본값은 구성 또는 `ws://127.0.0.1:<port>` 사용)
- `--token <token>`: Gateway 토큰(필요한 경우)
- `--password <password>`: Gateway 비밀번호(필요한 경우)
- `--session <key>`: 세션 키(기본값: `main`, 글로벌 범위에서는 `global`)
- `--deliver`: 어시스턴트 응답을 제공자에게 전달(기본값 꺼짐)
- `--thinking <level>`: 전송 시 사고 수준 재정의
- `--timeout-ms <ms>`: 에이전트 타임아웃(밀리초, 기본값은 `agents.defaults.timeoutSeconds`)
- `--history-limit <n>`: 로드할 기록 항목 수(기본값 200)

## 문제 해결

메시지를 보낸 후 출력이 없음:

- TUI에서 `/status`를 실행하여 Gateway가 연결되어 있고 유휴/바쁨 상태인지 확인합니다.
- Gateway 로그를 확인합니다: `openclaw logs --follow`.
- 에이전트가 실행될 수 있는지 확인합니다: `openclaw status` 및 `openclaw models status`.
- 채팅 채널에 메시지가 나타나기를 기대한다면 전달을 활성화하세요(`/deliver on` 또는 `--deliver`).

- `disconnected`: Gateway가 실행 중이고 `--url/--token/--password`가 올바른지 확인합니다.
- 선택기에 에이전트가 없음: `openclaw agents list`와 라우팅 구성을 확인합니다.
- 세션 선택기가 비어 있음: 글로벌 범위이거나 아직 세션이 없을 수 있습니다.
