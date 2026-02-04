---
read_when:
  - 소스 체크아웃을 안전하게 업데이트하고 싶을 때
  - `--update` 단축키 동작을 이해해야 할 때
summary: "`openclaw update`(안전한 소스 업데이트 + Gateway 자동 재시작)의 CLI 레퍼런스"
title: update
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw update`

OpenClaw를 안전하게 업데이트하고 stable/beta/dev 채널 간에 전환합니다.

**npm/pnpm**으로 설치한 경우(git 메타데이터 없는 전역 설치), 업데이트는 [업데이트](/install/updating)의 패키지 관리자 흐름을 통해 진행됩니다.

## 사용법

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## 옵션

- `--no-restart`: 성공적인 업데이트 후 Gateway 서비스 재시작 건너뛰기.
- `--channel <stable|beta|dev>`: 업데이트 채널 설정(git + npm; 구성에 저장됨).
- `--tag <dist-tag|version>`: 이번 업데이트에만 npm dist-tag 또는 버전 재정의.
- `--json`: 기계 판독 가능한 `UpdateRunResult` JSON 출력.
- `--timeout <seconds>`: 단계별 타임아웃(기본값 1200초).

참고: 다운그레이드는 확인이 필요합니다. 이전 버전이 구성을 손상시킬 수 있습니다.

## `update status`

현재 활성 업데이트 채널 + git 태그/브랜치/SHA(소스 체크아웃의 경우) 및 업데이트 가용성을 표시합니다.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

옵션:

- `--json`: 기계 판독 가능한 상태 JSON 출력.
- `--timeout <seconds>`: 확인 타임아웃(기본값 3초).

## `update wizard`

업데이트 채널을 선택하고 업데이트 후 Gateway 재시작 여부를 확인하는 대화형 흐름(기본적으로 재시작). `dev`를 선택했지만 git 체크아웃이 없으면 생성 옵션을 제공합니다.

## 작동 방식

채널을 명시적으로 전환(`--channel ...`)하면 OpenClaw는 설치 유형도 일관되게 유지합니다:

- `dev` → git 체크아웃이 존재하는지 확인(기본값: `~/openclaw`, `OPENCLAW_GIT_DIR`로 재정의 가능), 업데이트하고 해당 체크아웃에서 전역 CLI를 설치합니다.
- `stable`/`beta` → 일치하는 dist-tag로 npm에서 설치합니다.

## Git 체크아웃 흐름

채널:

- `stable`: 최신 비 beta 태그를 체크아웃한 후 빌드 + doctor.
- `beta`: 최신 `-beta` 태그를 체크아웃한 후 빌드 + doctor.
- `dev`: `main`을 체크아웃한 후 fetch + rebase.

개요 흐름:

1. 작업 트리가 깨끗해야 합니다(커밋되지 않은 변경 사항 없음).
2. 선택한 채널(태그 또는 브랜치)로 전환합니다.
3. 업스트림을 풀합니다(dev만).
4. dev만: 임시 작업 트리에서 사전 검사 lint + TypeScript 빌드; 최신 커밋이 실패하면 최신 성공 빌드 커밋을 찾기 위해 최대 10개 커밋을 뒤로 이동합니다.
5. 선택한 커밋으로 Rebase합니다(dev만).
6. 종속성 설치(pnpm 선호; npm으로 폴백).
7. 프로젝트 빌드 + 콘솔 UI 빌드.
8. 최종 "안전 업데이트" 검사로 `openclaw doctor` 실행.
9. 활성 채널에 플러그인 동기화(dev는 내장 확장 사용; stable/beta는 npm 사용) 및 npm으로 설치된 플러그인 업데이트.

## `--update` 단축키

`openclaw --update`는 `openclaw update`로 재작성됩니다(셸 및 시작 스크립트 편의용).

## 참고

- `openclaw doctor`(git 체크아웃에서 먼저 업데이트 실행 옵션 제공)
- [개발 채널](/install/development-channels)
- [업데이트](/install/updating)
