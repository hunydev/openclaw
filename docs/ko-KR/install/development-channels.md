---
read_when:
  - stable/beta/dev 간 전환하고 싶을 때
  - 프리릴리스 버전 태그 또는 릴리스 작업 중
summary: stable, beta, dev 채널 - 의미, 전환 및 태그 관리
title: 개발 채널
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 2b01219b7e705044ce39838a0da7c7fa65c719809ab2f8a51e14529064ab81bf
  source_path: install/development-channels.md
  workflow: 14
---

# 개발 채널

최종 업데이트: 2026-01-21

OpenClaw는 세 가지 업데이트 채널을 제공합니다:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (테스트 중인 빌드).
- **dev**: `main` 브랜치의 최신 커밋 (git). npm dist-tag: `dev` (릴리스 시).

빌드를 **beta**로 릴리스하고, 테스트한 다음, **검증된 빌드를 `latest`로 프로모션**합니다. 버전 번호 변경 없이 — dist-tag가 npm 설치의 권위 있는 출처입니다.

## 채널 전환

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta`는 가장 최근 일치하는 태그로 체크아웃 (보통 같은 태그).
- `dev`는 `main`으로 전환하고 업스트림에서 리베이스.

npm/pnpm 전역 설치:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

해당 npm dist-tag (`latest`, `beta`, `dev`)를 통해 업데이트합니다.

`--channel`로 **명시적으로** 채널을 전환하면 OpenClaw도 설치 방법을 동기화합니다:

- `dev`는 git checkout이 존재하는지 확인 (기본값 `~/openclaw`, `OPENCLAW_GIT_DIR`로 재정의 가능)하고, 업데이트한 다음, 해당 checkout에서 전역 CLI를 설치합니다.
- `stable`/`beta`는 일치하는 dist-tag로 npm에서 설치합니다.

팁: stable과 dev를 동시에 사용하려면 두 개의 클론을 유지하고 게이트웨이를 stable 클론으로 지정하세요.

## 플러그인 및 채널

`openclaw update`로 채널을 전환하면 OpenClaw도 플러그인 소스를 동기화합니다:

- `dev`는 git checkout에 내장된 플러그인을 선호.
- `stable` 및 `beta`는 npm을 통해 설치된 플러그인 패키지로 복원.

## 태그 모범 사례

- git checkout이 도달할 버전에 태그 지정 (`vYYYY.M.D` 또는 `vYYYY.M.D-<patch>`).
- 태그를 불변으로 유지: 태그를 이동하거나 재사용하지 마세요.
- npm dist-tag가 여전히 npm 설치의 권위 있는 출처:
  - `latest` → stable
  - `beta` → 후보 빌드
  - `dev` → main 스냅샷 (선택 사항)

## macOS 앱 가용성

베타 및 개발 빌드에는 macOS 앱 릴리스가 **포함되지 않을 수** 있습니다. 이는 정상입니다:

- git 태그와 npm dist-tag는 여전히 릴리스될 수 있습니다.
- 릴리스 노트나 변경 로그에 "이 베타에는 macOS 빌드 없음"이라고 표시하면 됩니다.
