---
read_when:
  - doctor 마이그레이션 추가 또는 수정 시
  - 호환성 깨는 구성 변경 도입 시
summary: Doctor 명령 - 상태 검사, 구성 마이그레이션 및 수정 단계
title: Doctor
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: df7b25f60fd08d508f4c6abfc8e7e06f29bd4bbb34c3320397f47eb72c8de83f
  source_path: gateway/doctor.md
  workflow: 14
---

# Doctor

`openclaw doctor`는 OpenClaw의 수정 + 마이그레이션 도구입니다. 오래된 구성/상태를 수정하고, 상태를 검사하며, 실행 가능한 수정 단계를 제공합니다.

## 빠른 시작

```bash
openclaw doctor
```

### 헤드리스 / 자동화

```bash
openclaw doctor --yes
```

프롬프트 없이 기본값 수락 (해당되는 경우 재시작/서비스/샌드박스 수정 단계 포함).

```bash
openclaw doctor --repair
```

프롬프트 없이 권장 수정 적용 (안전한 경우 수정 + 재시작 수행).

```bash
openclaw doctor --repair --force
```

공격적인 수정도 적용 (커스텀 supervisor 구성 덮어쓰기).

```bash
openclaw doctor --non-interactive
```

프롬프트 없이 실행, 안전한 마이그레이션만 적용 (구성 정규화 + 디스크 상태 마이그레이션). 사람의 확인이 필요한 재시작/서비스/샌드박스 작업은 건너뜀.
레거시 상태 마이그레이션 감지 시 자동 실행됩니다.

```bash
openclaw doctor --deep
```

추가 Gateway 설치를 위해 시스템 서비스 스캔 (launchd/systemd/schtasks).

쓰기 전에 변경 사항을 보려면 먼저 구성 파일을 열어보세요:

```bash
cat ~/.openclaw/openclaw.json
```

## 기능 개요

- 선택적 git 설치 사전 검사 업데이트 (대화형 모드에서만).
- UI 프로토콜 신선도 검사 (프로토콜 스키마 업데이트 시 제어 UI 재빌드).
- 상태 검사 + 재시작 프롬프트.
- Skills 상태 요약 (사용 가능/누락/차단됨).
- 레거시 값의 구성 정규화.
- OpenCode Zen 프로바이더 재정의 경고 (`models.providers.opencode`).
- 레거시 디스크 상태 마이그레이션 (세션/에이전트 디렉토리/WhatsApp 인증).
- 상태 무결성 및 권한 검사 (세션, 트랜스크립트, 상태 디렉토리).
- 로컬 실행 시 구성 파일 권한 검사 (chmod 600).
- 모델 인증 상태: OAuth 만료 확인, 만료 임박 토큰 새로 고침, 인증 구성의 쿨다운/비활성화 상태 보고.
- 추가 작업 공간 디렉토리 감지 (`~/openclaw`).
- 샌드박스 활성화 시 샌드박스 이미지 수정.
- 레거시 서비스 마이그레이션 및 추가 Gateway 감지.
- Gateway 런타임 검사 (서비스 설치됨 but 실행 안 됨; 캐시된 launchd 레이블).
- 채널 상태 경고 (실행 중인 Gateway에서 프로브).
- Supervisor 구성 감사 (launchd/systemd/schtasks) 및 선택적 수정.
- Gateway 런타임 모범 사례 검사 (Node vs Bun, 버전 관리자 경로).
- Gateway 포트 충돌 진단 (기본값 `18789`).
- 열린 DM 정책의 보안 경고.
- `gateway.auth.token` 미설정 시 Gateway 인증 경고 (로컬 모드; 토큰 생성 제공).
- Linux에서 systemd linger 검사.
- 소스 설치 검사 (pnpm workspace 불일치, UI 자산 누락, tsx 바이너리 누락).
- 업데이트된 구성 + 마법사 메타데이터 기록.

## 상세 동작 및 원리

### 0) 선택적 업데이트 (git 설치)

git 체크아웃이고 doctor가 대화형 모드에서 실행 중이면 doctor 실행 전에 업데이트 (fetch/rebase/build)를 제안합니다.

### 1) 구성 정규화

구성에 레거시 값 형식이 포함된 경우 (예: 채널별 재정의 없는 `messages.ackReaction`) doctor가 현재 스키마로 정규화합니다.

### 2) 레거시 구성 키 마이그레이션

구성에 더 이상 사용되지 않는 키가 포함된 경우 다른 명령은 실행을 거부하고 `openclaw doctor` 실행을 요청합니다.

Doctor는:

- 발견된 레거시 키를 설명합니다.
- 적용하는 마이그레이션을 표시합니다.
- 업데이트된 스키마로 `~/.openclaw/openclaw.json`을 다시 씁니다.

Gateway는 시작 시 레거시 구성 형식을 감지하면 자동으로 doctor 마이그레이션을 실행하므로 수동으로 doctor를 실행하지 않아도 오래된 구성이 수정됩니다.

현재 마이그레이션:

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → 최상위 `bindings`
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

### 2b) OpenCode Zen 프로바이더 재정의

`models.providers.opencode` (또는 `opencode-zen`)를 수동으로 추가했다면 `@mariozechner/pi-ai` 내장 OpenCode Zen 디렉토리를 재정의합니다. 이로 인해 모든 모델이 단일 API로 강제되거나 비용이 0이 될 수 있습니다. Doctor는 재정의를 제거하고 모델별 API 라우팅 + 비용을 복원하도록 경고합니다.

### 3) 레거시 상태 마이그레이션 (디스크 레이아웃)

Doctor는 이전 버전의 디스크 레이아웃을 현재 구조로 마이그레이션할 수 있습니다:

- 세션 저장소 + 트랜스크립트:
  - `~/.openclaw/sessions/`에서 `~/.openclaw/agents/<agentId>/sessions/`로
- 에이전트 디렉토리:
  - `~/.openclaw/agent/`에서 `~/.openclaw/agents/<agentId>/agent/`로
- WhatsApp 인증 상태 (Baileys):
  - 레거시 `~/.openclaw/credentials/*.json`에서 (`oauth.json` 제외)
  - `~/.openclaw/credentials/whatsapp/<accountId>/...`로 (기본 계정 ID: `default`)

이러한 마이그레이션은 최선의 노력으로 수행되며 멱등성이 있습니다; doctor가 레거시 폴더를 백업으로 유지할 때 경고합니다. Gateway/CLI도 시작 시 레거시 세션 + 에이전트 디렉토리를 자동 마이그레이션하여 히스토리/인증/모델이 수동으로 doctor를 실행하지 않아도 에이전트별 경로에 배치됩니다. WhatsApp 인증은 `openclaw doctor`를 통해서만 마이그레이션됩니다.

### 4) 상태 무결성 검사 (세션 지속성, 라우팅 및 보안)

상태 디렉토리는 운영의 핵심입니다. 사라지면 세션, 자격 증명, 로그 및 구성을 잃게 됩니다 (다른 곳에 백업이 없는 한).

Doctor 검사:

- **상태 디렉토리 누락**: 치명적 상태 손실 경고, 디렉토리 재생성 제안, 손실된 데이터는 복구 불가 알림.
- **상태 디렉토리 권한**: 쓰기 가능성 확인; 권한 수정 옵션 제공 (소유자/그룹 불일치 감지 시 `chown` 힌트).
- **세션 디렉토리 누락**: `sessions/` 및 세션 저장소 디렉토리는 히스토리 지속 및 `ENOENT` 크래시 방지에 필수.
- **트랜스크립트 불일치**: 최근 세션 항목에 트랜스크립트 파일이 누락된 경우 경고.
- **메인 세션 "단일 행 JSONL"**: 메인 트랜스크립트가 한 줄뿐일 때 플래그 (히스토리가 누적되지 않음).
- **여러 상태 디렉토리**: 여러 `~/.openclaw` 폴더가 다른 홈 디렉토리에 존재하거나 `OPENCLAW_STATE_DIR`이 다른 곳을 가리킬 때 경고 (설치 간 히스토리가 분산될 수 있음).
- **원격 모드 알림**: `gateway.mode=remote`면 doctor가 원격 호스트에서 실행하라고 알림 (상태가 거기에 저장됨).
- **구성 파일 권한**: `~/.openclaw/openclaw.json`이 그룹/기타 사용자에게 읽기 가능할 때 경고하고 `600`으로 조이는 옵션 제공.

### 5) 모델 인증 상태 (OAuth 만료)

Doctor는 인증 저장소의 OAuth 구성을 검사하고, 토큰이 만료 임박/이미 만료되면 경고하며, 안전할 때 새로 고침합니다. Anthropic Claude Code 구성이 오래되면 `claude setup-token` 실행 (또는 setup-token 붙여넣기)을 제안합니다. 새로 고침 프롬프트는 대화형 실행 (TTY)에서만 나타남; `--non-interactive`는 새로 고침 시도를 건너뜁니다.

Doctor는 다음 이유로 일시적으로 사용 불가한 인증 구성도 보고합니다:

- 단기 쿨다운 (속도 제한/타임아웃/인증 실패)
- 장기 비활성화 (청구/크레딧 실패)

### 6) Hooks 모델 검증

`hooks.gmail.model`이 설정되면 doctor가 디렉토리 및 허용 목록에 대해 모델 참조를 검증하고 해석 불가 또는 금지된 경우 경고합니다.

### 7) 샌드박스 이미지 수정

샌드박스가 활성화되면 doctor가 Docker 이미지를 검사하고 현재 이미지가 누락된 경우 빌드하거나 레거시 이름으로 전환하는 옵션을 제공합니다.

### 8) Gateway 서비스 마이그레이션 및 정리 힌트

Doctor는 레거시 Gateway 서비스 (launchd/systemd/schtasks)를 감지하고 삭제 후 현재 Gateway 포트로 OpenClaw 서비스를 설치하는 옵션을 제공합니다. 추가적인 Gateway 유사 서비스도 스캔하고 정리 힌트를 출력합니다. 프로필로 명명된 OpenClaw Gateway 서비스는 일급 시민으로 취급되어 "추가"로 플래그되지 않습니다.

### 9) 보안 경고

프로바이더가 허용 목록 없이 DM에 열려 있거나 정책이 위험하게 구성된 경우 doctor가 경고합니다.

### 10) systemd linger (Linux)

systemd 사용자 서비스로 실행 중이면 doctor가 linger 활성화를 확인하여 Gateway가 로그아웃 후에도 계속 실행되도록 합니다.

### 11) Skills 상태

Doctor는 현재 작업 공간의 사용 가능/누락/차단된 Skills 빠른 요약을 출력합니다.

### 12) Gateway 인증 검사 (로컬 토큰)

로컬 Gateway에 `gateway.auth`가 없으면 doctor가 경고하고 토큰 생성 옵션을 제공합니다. 자동화에서 토큰 생성을 강제하려면 `openclaw doctor --generate-gateway-token` 사용.

### 13) Gateway 상태 검사 + 재시작

Doctor는 상태 검사를 실행하고 Gateway가 비정상적으로 보이면 재시작 옵션을 제공합니다.

### 14) 채널 상태 경고

Gateway가 정상이면 doctor가 채널 상태 프로브를 실행하고 경고 및 제안된 수정 사항을 보고합니다.

### 15) Supervisor 구성 감사 + 수정

Doctor는 설치된 supervisor 구성 (launchd/systemd/schtasks)에서 누락되거나 오래된 기본값 (예: systemd의 network-online 의존성 및 재시작 지연)을 검사합니다. 불일치 발견 시 업데이트를 권장하고 서비스 파일/태스크를 현재 기본값으로 다시 쓸 수 있습니다.

설명:

- `openclaw doctor`는 supervisor 구성을 다시 쓰기 전에 확인을 요청합니다.
- `openclaw doctor --yes`는 기본 수정 프롬프트를 수락합니다.
- `openclaw doctor --repair`는 프롬프트 없이 권장 수정을 적용합니다.
- `openclaw doctor --repair --force`는 커스텀 supervisor 구성을 덮어씁니다.
- 완전 다시 쓰기를 강제하려면 항상 `openclaw gateway install --force` 사용 가능.

### 16) Gateway 런타임 + 포트 진단

Doctor는 서비스 런타임 (PID, 마지막 종료 상태)을 검사하고 서비스가 설치되었지만 실제로 실행 중이지 않으면 경고합니다. Gateway 포트 (기본값 `18789`)의 포트 충돌도 검사하고 가능한 원인 (Gateway 이미 실행 중, SSH 터널)을 보고합니다.

### 17) Gateway 런타임 모범 사례

Gateway 서비스가 Bun 또는 버전 관리자 관리 Node 경로 (`nvm`, `fnm`, `volta`, `asdf` 등)에서 실행 중일 때 doctor가 경고합니다. WhatsApp + Telegram 채널은 Node가 필요하며 버전 관리자 경로는 업그레이드 후 실효될 수 있습니다 (서비스가 셸 초기화 파일을 로드하지 않기 때문). Doctor는 시스템 Node 설치가 가능할 때 마이그레이션 옵션을 제공합니다 (Homebrew/apt/choco).

### 18) 구성 쓰기 + 마법사 메타데이터

Doctor는 모든 구성 변경을 영구화하고 마법사 메타데이터를 기록하여 doctor 실행을 표시합니다.

### 19) 작업 공간 힌트 (백업 + 메모리 시스템)

작업 공간 메모리 시스템이 누락된 경우 doctor가 추가를 제안하고 작업 공간이 아직 git 관리되지 않으면 백업 힌트를 출력합니다.

작업 공간 구조 및 git 백업 (프라이빗 GitHub 또는 GitLab 권장)의 전체 가이드는 [/concepts/agent-workspace](/concepts/agent-workspace)를 참조하세요.
