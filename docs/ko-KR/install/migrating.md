---
read_when:
  - OpenClaw를 새 노트북/서버로 마이그레이션 중
  - 세션, 인증 및 채널 로그인 (WhatsApp 등)을 유지하고 싶을 때
summary: OpenClaw 설치를 한 컴퓨터에서 다른 컴퓨터로 마이그레이션
title: 마이그레이션 가이드
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 604d862c4bf86e7924d09028db8cc2514ca6f1d64ebe8bb7d1e2dde57ef70caa
  source_path: install/migrating.md
  workflow: 14
---

# OpenClaw를 새 컴퓨터로 마이그레이션

이 가이드는 **온보딩을 다시 하지 않고** OpenClaw 게이트웨이를 한 컴퓨터에서 다른 컴퓨터로 마이그레이션하는 방법을 다룹니다.

마이그레이션은 개념적으로 간단합니다:

- **상태 디렉토리** (`$OPENCLAW_STATE_DIR`, 기본값: `~/.openclaw/`) 복사 — 구성, 인증, 세션 및 채널 상태 포함.
- **작업 공간** (기본값 `~/.openclaw/workspace/`) 복사 — 에이전트 파일 (메모리, 프롬프트 등) 포함.

그러나 **프로필**, **권한**, **불완전한 복사**에 관한 몇 가지 일반적인 함정이 있습니다.

## 시작 전 (마이그레이션 대상)

### 1) 상태 디렉토리 확인

대부분의 설치는 기본 경로를 사용합니다:

- **상태 디렉토리:** `~/.openclaw/`

그러나 다음을 사용한 경우 경로가 다를 수 있습니다:

- `--profile <name>` (보통 `~/.openclaw-<profile>/`이 됨)
- `OPENCLAW_STATE_DIR=/some/path`

확실하지 않으면 **이전** 컴퓨터에서 실행:

```bash
openclaw status
```

출력에서 `OPENCLAW_STATE_DIR` / 프로필 관련 정보를 찾으세요. 여러 게이트웨이를 실행한 경우 각 프로필에 대해 반복하세요.

### 2) 작업 공간 확인

일반적인 기본 경로:

- `~/.openclaw/workspace/` (권장 작업 공간)
- 직접 만든 커스텀 폴더

작업 공간은 `MEMORY.md`, `USER.md`, `memory/*.md` 등의 파일이 있는 곳입니다.

### 3) 유지되는 것 이해

상태 디렉토리와 작업 공간을 **모두** 복사하면 다음을 유지합니다:

- 게이트웨이 구성 (`openclaw.json`)
- 인증 구성 / API 키 / OAuth 토큰
- 세션 기록 + 에이전트 상태
- 채널 상태 (예: WhatsApp 로그인/세션)
- 작업 공간 파일 (메모리, Skills 노트 등)

작업 공간**만** 복사하면 (예: Git을 통해) 다음을 유지**하지 않습니다**:

- 세션
- 자격 증명
- 채널 로그인 상태

이들은 `$OPENCLAW_STATE_DIR` 아래에 저장됩니다.

## 마이그레이션 단계 (권장)

### 단계 0 — 백업 (이전 컴퓨터)

**이전** 컴퓨터에서 먼저 게이트웨이를 중지하여 복사 중 파일이 변경되지 않도록 합니다:

```bash
openclaw gateway stop
```

(선택 사항이지만 권장) 상태 디렉토리와 작업 공간 아카이브:

```bash
# 프로필이나 커스텀 경로를 사용한 경우 경로 조정
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

여러 프로필/상태 디렉토리가 있으면 (예: `~/.openclaw-main`, `~/.openclaw-work`) 각각 아카이브하세요.

### 단계 1 — 새 컴퓨터에 OpenClaw 설치

**새** 컴퓨터에 CLI 설치 (필요하면 Node도):

- 참조: [설치](/install)

이 단계에서 온보딩이 새로운 `~/.openclaw/`를 생성해도 괜찮습니다 — 다음 단계에서 덮어씁니다.

### 단계 2 — 상태 디렉토리 + 작업 공간을 새 컴퓨터로 복사

**둘 다** 복사:

- `$OPENCLAW_STATE_DIR` (기본값 `~/.openclaw/`)
- 작업 공간 (기본값 `~/.openclaw/workspace/`)

일반적인 방법:

- `scp`로 압축 파일 전송 및 압축 해제
- SSH를 통한 `rsync -a`
- 외부 저장 장치

복사 후 확인:

- 숨김 디렉토리가 포함됨 (예: `.openclaw/`)
- 게이트웨이를 실행하는 사용자에 대해 파일 소유권이 올바름

### 단계 3 — Doctor 실행 (마이그레이션 + 서비스 수정)

**새** 컴퓨터에서:

```bash
openclaw doctor
```

Doctor는 "안전한" 명령입니다. 서비스를 수정하고, 구성 마이그레이션을 적용하고, 불일치에 대해 경고합니다.

그런 다음:

```bash
openclaw gateway restart
openclaw status
```

## 일반적인 함정 (및 피하는 방법)

### 함정: 프로필 / 상태 디렉토리 불일치

이전 게이트웨이가 프로필 (또는 `OPENCLAW_STATE_DIR`)을 사용했는데 새 게이트웨이가 다른 경로를 사용하면 다음 증상이 나타납니다:

- 구성 변경이 적용되지 않음
- 채널 누락 / 로그아웃됨
- 세션 기록이 비어 있음

수정: 마이그레이션한 것과 동일한 프로필/상태 디렉토리로 게이트웨이/서비스를 실행한 다음:

```bash
openclaw doctor
```

### 함정: `openclaw.json`만 복사

`openclaw.json`만으로는 충분하지 않습니다. 많은 프로바이더가 다음에 상태를 저장합니다:

- `$OPENCLAW_STATE_DIR/credentials/`
- `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

항상 전체 `$OPENCLAW_STATE_DIR` 폴더를 마이그레이션하세요.

### 함정: 권한 / 소유권

root로 복사하거나 사용자를 전환한 경우, 게이트웨이가 자격 증명/세션을 읽지 못할 수 있습니다.

수정: 상태 디렉토리 + 작업 공간의 소유자가 게이트웨이를 실행하는 사용자인지 확인하세요.

### 함정: 원격/로컬 모드 간 마이그레이션

- 인터페이스 (WebUI/TUI)가 **원격** 게이트웨이를 가리키면, 원격 호스트가 세션 저장소 + 작업 공간을 소유합니다.
- 노트북을 마이그레이션해도 원격 게이트웨이의 상태는 이동되지 않습니다.

원격 모드인 경우 **게이트웨이 호스트**를 마이그레이션하세요.

### 함정: 백업의 비밀

`$OPENCLAW_STATE_DIR`에는 비밀 (API 키, OAuth 토큰, WhatsApp 자격 증명)이 포함되어 있습니다. 백업을 프로덕션 비밀처럼 취급하세요:

- 암호화하여 저장
- 안전하지 않은 채널로 전송 피하기
- 유출이 의심되면 키 교체

## 검증 체크리스트

새 컴퓨터에서 확인:

- `openclaw status`에 게이트웨이가 실행 중으로 표시
- 채널이 여전히 연결됨 (예: WhatsApp 재페어링 불필요)
- 대시보드가 열리고 기존 세션 표시
- 작업 공간 파일 (메모리, 구성)이 존재

## 관련 항목

- [Doctor](/gateway/doctor)
- [게이트웨이 문제 해결](/gateway/troubleshooting)
- [OpenClaw는 데이터를 어디에 저장하나요?](/help/faq#where-does-openclaw-store-its-data)
