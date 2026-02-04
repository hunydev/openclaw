---
read_when:
  - 가장 빠른 로컬 개발 사이클 (bun + watch)을 원할 때
  - Bun 설치/패치/라이프사이클 스크립트 문제가 있을 때
summary: Bun 워크플로우 (실험적) - 설치 방법 및 pnpm 대비 주의 사항
title: Bun (실험적)
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: eb3f4c222b6bae49938d8bf53a0818fe5f5e0c0c3c1adb3e0a832ce8f785e1e3
  source_path: install/bun.md
  workflow: 14
---

# Bun (실험적)

목표: pnpm 워크플로우를 벗어나지 않으면서 **Bun**으로 이 저장소를 실행합니다 (선택 사항, WhatsApp/Telegram에는 권장하지 않음).

⚠️ **게이트웨이 런타임에는 권장하지 않음** (WhatsApp/Telegram 버그 있음). 프로덕션에는 Node를 사용하세요.

## 상태

- Bun은 TypeScript를 직접 실행하기 위한 선택적 로컬 런타임입니다 (`bun run …`, `bun --watch …`).
- `pnpm`이 기본 빌드 도구이며 여전히 완전히 지원됩니다 (일부 문서 도구도 사용).
- Bun은 `pnpm-lock.yaml`을 사용할 수 없으며 무시합니다.

## 설치

기본 방식:

```sh
bun install
```

참고: `bun.lock`/`bun.lockb`는 gitignore되어 있어 저장소가 변경되지 않습니다. lock 파일을 쓰지 않으려면:

```sh
bun install --no-save
```

## 빌드 / 테스트 (Bun)

```sh
bun run build
bun run vitest run
```

## Bun 라이프사이클 스크립트 (기본 차단)

Bun은 명시적으로 신뢰하지 않으면 의존성의 라이프사이클 스크립트를 차단할 수 있습니다 (`bun pm untrusted` / `bun pm trust`).
이 저장소의 경우 일반적으로 차단되는 스크립트는 필수가 아닙니다:

- `@whiskeysockets/baileys` `preinstall`: Node 메이저 버전 >= 20 확인 (우리는 Node 22+ 실행).
- `protobufjs` `postinstall`: 호환되지 않는 버전 체계에 대한 경고 발생 (빌드 아티팩트 없음).

이 스크립트가 실제로 필요한 런타임 문제가 발생하면 명시적으로 신뢰하세요:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## 주의 사항

- 일부 스크립트는 여전히 pnpm을 하드코딩합니다 (예: `docs:build`, `ui:*`, `protocol:check`). 현재는 pnpm을 통해 실행하세요.
