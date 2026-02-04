---
read_when:
  - 저장소에서 스크립트를 실행할 때
  - ./scripts 아래에 스크립트를 추가하거나 수정할 때
summary: 저장소 스크립트: 목적, 범위 및 보안 참고 사항
title: 스크립트
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 스크립트

`scripts/` 디렉토리에는 로컬 워크플로와 운영 작업을 위한 도우미 스크립트가 포함되어 있습니다.
작업이 명확하게 특정 스크립트와 관련된 경우에만 사용하세요; 그렇지 않으면 CLI를 우선하세요.

## 규칙

- 문서나 릴리스 체크리스트에서 참조되지 않는 한 스크립트는 **선택 사항**입니다.
- CLI 인터페이스가 존재하면 우선 사용하세요(예: 인증 모니터링에는 `openclaw models status --check` 사용).
- 스크립트는 특정 호스트에 연결되어 있다고 가정합니다; 새 머신에서 실행하기 전에 스크립트를 읽으세요.

## Git 훅

- `scripts/setup-git-hooks.js`: git 저장소에서 `core.hooksPath`를 최선의 노력으로 설정합니다.
- `scripts/format-staged.js`: 스테이징된 `src/` 및 `test/` 파일을 위한 사전 커밋 포매터.

## 인증 모니터링 스크립트

인증 모니터링 스크립트 문서는 다음을 참조하세요:
[/automation/auth-monitoring](/automation/auth-monitoring)

## 스크립트 추가 시

- 스크립트를 집중적이고 문서화된 상태로 유지하세요.
- 관련 문서에 간략한 항목을 추가하세요(없으면 만드세요).
