---
read_when:
  - 원시 모델 출력에서 추론 누출을 검사해야 하는 경우
  - 반복 개발 중 감시 모드로 Gateway를 실행하려는 경우
  - 재현 가능한 디버깅 워크플로가 필요한 경우
summary: 디버깅 도구: 감시 모드, 원시 모델 스트림 및 추론 누출 추적
title: 디버깅
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 디버깅

이 페이지는 스트리밍 출력을 위한 디버깅 도우미를 다루며, 특히 제공자가 추론 내용을 일반 텍스트에 혼합하는 경우에 유용합니다.

## 런타임 디버그 재정의

채팅에서 `/debug`를 사용하여 **런타임 전용** 구성 재정의를 설정합니다(메모리에만, 디스크에 쓰지 않음).
`/debug`는 기본적으로 비활성화되어 있습니다. `commands.debug: true`로 활성화하세요.
자주 사용하지 않는 설정을 전환하면서 `openclaw.json`을 편집하고 싶지 않을 때 유용합니다.

예시:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset`은 모든 재정의를 지우고 디스크의 구성으로 되돌립니다.

## Gateway 감시 모드

빠른 반복을 위해 파일 감시자 아래에서 Gateway를 실행합니다:

```bash
pnpm gateway:watch --force
```

해당 명령:

```bash
tsx watch src/entry.ts gateway --force
```

`gateway:watch` 뒤에 Gateway CLI 플래그를 추가하면 매 재시작마다 전달됩니다.

## 개발 프로필 + 개발 Gateway(--dev)

개발 프로필을 사용하여 상태를 격리하고 안전하게 버릴 수 있는 디버깅 환경을 만드세요. **두 개의** `--dev` 플래그가 있습니다:

- **전역 `--dev`(프로필):** 상태를 `~/.openclaw-dev`로 격리하고 Gateway 기본 포트를 `19001`로 설정합니다(파생 포트도 오프셋됨).
- **`gateway --dev`:** Gateway에 구성과 작업 공간이 없을 때 기본 구성 + 작업 공간을 자동 생성하도록 지시합니다(BOOTSTRAP.md 건너뜀).

권장 흐름(개발 프로필 + 개발 부트스트랩):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

전역 설치가 없다면 `pnpm openclaw ...`로 CLI를 실행할 수 있습니다.

효과:

1. **프로필 격리**(전역 `--dev`)
   - `OPENCLAW_PROFILE=dev`
   - `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   - `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   - `OPENCLAW_GATEWAY_PORT=19001`(브라우저/캔버스 포트도 오프셋됨)

2. **개발 부트스트랩**(`gateway --dev`)
   - 구성이 없으면 최소 구성 작성(`gateway.mode=local`, 로컬 루프백에 바인딩).
   - `agent.workspace`를 개발 작업 공간으로 설정.
   - `agent.skipBootstrap=true` 설정(BOOTSTRAP.md 사용 안 함).
   - 작업 공간 파일이 없으면 초기화:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   - 기본 아이덴티티: **C3‑PO**(프로토콜 드로이드).
   - 개발 모드에서 채널 제공자 건너뜀(`OPENCLAW_SKIP_CHANNELS=1`).

리셋 흐름(새로 시작):

```bash
pnpm gateway:dev:reset
```

참고: `--dev`는 **전역** 프로필 플래그이며 일부 러너에서 삼켜질 수 있습니다.
명시적으로 지정하려면 환경 변수 형식을 사용하세요:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset`은 구성, 자격 증명, 세션 및 개발 작업 공간을 지우고(`rm` 대신 `trash` 사용) 기본 개발 환경을 다시 만듭니다.

팁: 비개발 Gateway가 이미 실행 중이라면(launchd/systemd) 먼저 중지하세요:

```bash
openclaw gateway stop
```

## 원시 스트림 로그(OpenClaw)

OpenClaw는 필터링/포맷팅 전에 **원시 어시스턴트 스트림**을 로그할 수 있습니다.
추론 내용이 일반 텍스트 델타로 도착하는지 별도의 생각 블록으로 도착하는지 확인하는 가장 좋은 방법입니다.

CLI를 통해 활성화:

```bash
pnpm gateway:watch --force --raw-stream
```

선택적 경로 재정의:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

동등한 환경 변수:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

기본 파일:

`~/.openclaw/logs/raw-stream.jsonl`

## 원시 청크 로그(pi-mono)

청크로 파싱되기 전에 **원시 OpenAI 호환 청크**를 캡처하려면 pi-mono에서 별도의 로거를 제공합니다:

```bash
PI_RAW_STREAM=1
```

선택적 경로:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

기본 파일:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 참고: 이 로그는 pi-mono의 `openai-completions` 제공자를 사용하는 프로세스에서만 생성됩니다.

## 보안 고려 사항

- 원시 스트림 로그에는 전체 프롬프트, 도구 출력 및 사용자 데이터가 포함될 수 있습니다.
- 로그를 로컬에 보관하고 디버깅 후 즉시 삭제하세요.
- 로그를 공유해야 하는 경우 먼저 비밀과 개인 식별 정보를 제거하세요.
