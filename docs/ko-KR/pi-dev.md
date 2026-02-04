---
title: Pi 개발 워크플로
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Pi 개발 워크플로

이 가이드는 OpenClaw에서 Pi 통합 개발을 위한 합리적인 워크플로를 요약합니다.

## 타입 검사 및 코드 검사

- 타입 검사 및 빌드: `pnpm build`
- 코드 검사: `pnpm lint`
- 포맷 검사: `pnpm format`
- 푸시 전 전체 검사: `pnpm lint && pnpm build && pnpm test`

## Pi 테스트 실행

Pi 통합 테스트 집합을 위한 전용 스크립트 사용:

```bash
scripts/pi/run-tests.sh
```

실제 제공자 동작을 실행하는 라이브 테스트를 포함하려면:

```bash
scripts/pi/run-tests.sh --live
```

이 스크립트는 다음 glob 패턴을 통해 모든 Pi 관련 단위 테스트를 실행합니다:

- `src/agents/pi-*.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-tool-definition-adapter.test.ts`
- `src/agents/pi-extensions/*.test.ts`

## 수동 테스트

권장 흐름:

- 개발 모드로 Gateway 실행:
  - `pnpm gateway:dev`
- 에이전트 직접 트리거:
  - `pnpm openclaw agent --message "Hello" --thinking low`
- 대화형 디버깅을 위한 TUI 사용:
  - `pnpm tui`

도구 호출 동작을 테스트하려면 `read` 또는 `exec` 작업을 수행하도록 프롬프트하여 도구 스트리밍과 페이로드 처리를 관찰하세요.

## 전체 리셋

상태는 OpenClaw 상태 디렉토리에 저장됩니다. 기본값은 `~/.openclaw`입니다. `OPENCLAW_STATE_DIR`이 설정되어 있으면 해당 디렉토리를 사용합니다.

모든 것을 리셋하려면:

- `openclaw.json` 구성용
- `credentials/` 인증 프로필 및 토큰용
- `agents/<agentId>/sessions/` 에이전트 세션 기록용
- `agents/<agentId>/sessions.json` 세션 인덱스용
- `sessions/` 레거시 경로가 있는 경우
- `workspace/` 빈 작업 공간이 필요한 경우

세션만 리셋하려면 해당 에이전트의 `agents/<agentId>/sessions/`와 `agents/<agentId>/sessions.json`을 삭제하세요. 재인증을 원하지 않으면 `credentials/`를 유지하세요.

## 참조

- https://docs.openclaw.ai/testing
- https://docs.openclaw.ai/start/getting-started
