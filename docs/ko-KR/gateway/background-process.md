---
read_when:
  - 백그라운드 exec 동작 추가 또는 수정 시
  - 장시간 실행되는 exec 작업 디버깅 시
summary: 백그라운드 exec 실행 및 프로세스 관리
title: 백그라운드 Exec 및 Process 도구
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: e11a7d74a75000d6882f703693c2c49a2ecd3e730b6ec2b475ac402abde9e465
  source_path: gateway/background-process.md
  workflow: 14
---

# 백그라운드 Exec + Process 도구

OpenClaw는 `exec` 도구로 셸 명령을 실행하고 장시간 실행되는 작업을 메모리에 유지합니다. `process` 도구는 이러한 백그라운드 세션을 관리합니다.

## exec 도구

주요 파라미터:

- `command` (필수)
- `yieldMs` (기본값 10000): 이 지연 후 자동으로 백그라운드로 전환
- `background` (불리언): 즉시 백그라운드로 전환
- `timeout` (초, 기본값 1800): 타임아웃 후 프로세스 종료
- `elevated` (불리언): 상승 모드가 활성화/허용된 경우 호스트에서 실행
- 실제 TTY가 필요하면 `pty: true` 설정
- `workdir`, `env`

동작:

- 포그라운드 실행은 직접 출력을 반환합니다.
- 백그라운드로 전환되면 (명시적으로 지정하거나 타임아웃 트리거), 도구는 `status: "running"` + `sessionId`와 끝부분의 출력을 반환합니다.
- 출력은 세션이 폴링되거나 정리될 때까지 메모리에 유지됩니다.
- `process` 도구가 비활성화되면 `exec`는 동기적으로 실행되고 `yieldMs`/`background`를 무시합니다.

## 서브프로세스 브릿지

exec/process 도구 외부에서 장시간 실행되는 서브프로세스를 시작할 때 (예: CLI 재시작 또는 Gateway 헬퍼 프로세스), 서브프로세스 브릿지 헬퍼를 마운트하세요. 이렇게 하면 종료 신호가 올바르게 전달되고 종료/오류 시 리스너가 분리됩니다. 이것은 systemd에서 고아 프로세스를 방지하고 플랫폼 간 종료 동작의 일관성을 유지합니다.

환경 변수 오버라이드:

- `PI_BASH_YIELD_MS`: 기본 yield 시간 (밀리초)
- `PI_BASH_MAX_OUTPUT_CHARS`: 메모리 출력 상한 (문자 수)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: 스트림당 대기 중인 stdout/stderr 상한 (문자 수)
- `PI_BASH_JOB_TTL_MS`: 완료된 세션의 TTL (밀리초, 1분에서 3시간 사이로 제한)

구성 (권장 방법):

- `tools.exec.backgroundMs` (기본값 10000)
- `tools.exec.timeoutSec` (기본값 1800)
- `tools.exec.cleanupMs` (기본값 1800000)
- `tools.exec.notifyOnExit` (기본값 true): 백그라운드 exec 종료 시 시스템 이벤트 큐에 추가하고 하트비트 요청

## process 도구

작업:

- `list`: 실행 중인 세션과 완료된 세션 나열
- `poll`: 세션의 새 출력 가져오기 (종료 상태도 보고)
- `log`: 집계된 출력 읽기 (`offset` + `limit` 지원)
- `write`: stdin 전송 (`data`, 선택적 `eof`)
- `kill`: 백그라운드 세션 종료
- `clear`: 완료된 세션을 메모리에서 제거
- `remove`: 실행 중이면 종료, 완료되었으면 정리

참고 사항:

- 백그라운드 세션만 나열/메모리에 유지됩니다.
- 프로세스 재시작 후 세션이 손실됩니다 (디스크 지속성 없음).
- 세션 로그는 `process poll/log`를 실행하고 도구 결과가 기록될 때만 채팅 히스토리에 저장됩니다.
- `process`의 범위는 단일 에이전트입니다; 해당 에이전트가 시작한 세션만 볼 수 있습니다.
- `process list`는 빠른 탐색을 위한 파생된 `name` (명령 동사 + 대상)을 포함합니다.
- `process log`는 행 기반 `offset`/`limit`를 사용합니다 (`offset` 생략 시 마지막 N 행).

## 예제

장시간 작업을 실행하고 나중에 폴링:

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

즉시 백그라운드에서 시작:

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

stdin 전송:

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```
