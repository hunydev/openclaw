---
read_when:
  - 새 컴퓨터에서 설정할 때
  - 개인 설정을 유지하면서 최신 버전을 사용하고 싶을 때
summary: 설정 가이드: 업데이트를 유지하면서 맞춤 OpenClaw 설정 관리하기
title: 설정
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: b7f4bd657d0df4feb5035c9f5ee727f9c67b991e9cedfc7768f99d010553fa01
  source_path: start/setup.md
  workflow: 15
---

# 설정

최종 업데이트: 2026-01-01

## 요약

- **개인화 설정은 저장소 외부에 보관:** `~/.openclaw/workspace` (워크스페이스) + `~/.openclaw/openclaw.json` (설정).
- **안정적인 워크플로우:** macOS 앱을 설치하고, 내장 Gateway를 실행합니다.
- **최신 버전 워크플로우:** `pnpm gateway:watch`로 Gateway를 직접 실행한 후, macOS 앱을 Local 모드로 연결합니다.

## 사전 요구 사항 (소스 빌드)

- Node `>=22`
- `pnpm`
- Docker (선택 사항; 컨테이너화된 설정/E2E 테스트용 — [Docker](/install/docker) 참조)

## 개인화 전략 (업데이트해도 설정이 유지되도록)

"내 방식으로 100% 맞춤 설정"하면서 쉽게 업데이트하려면, 커스터마이징 내용을 다음 위치에 보관하세요:

- **설정:** `~/.openclaw/openclaw.json` (JSON/JSON5 형식)
- **워크스페이스:** `~/.openclaw/workspace` (Skills, 프롬프트, 메모리; 비공개 git 저장소로 관리 권장)

최초 한 번 초기화:

```bash
openclaw setup
```

이 저장소 내에서는 로컬 CLI 엔트리를 사용하세요:

```bash
openclaw setup
```

전역 설치가 아직 안 되어 있다면 `pnpm openclaw setup`으로 실행하세요.

## 안정적인 워크플로우 (macOS 앱 우선)

1. **OpenClaw.app** 설치 및 실행 (메뉴 바).
2. 온보딩/권한 체크리스트 완료 (TCC 권한 요청).
3. Gateway가 **Local** 모드이고 실행 중인지 확인 (앱이 관리).
4. 채팅 채널 연결 (예: WhatsApp):

```bash
openclaw channels login
```

5. 설치 확인:

```bash
openclaw health
```

빌드에서 온보딩을 사용할 수 없는 경우:

- `openclaw setup` 실행 후, `openclaw channels login`, 그 다음 Gateway를 수동으로 시작 (`openclaw gateway`).

## 최신 버전 워크플로우 (터미널에서 Gateway 실행)

목표: TypeScript Gateway 개발, 핫 리로드 적용, macOS 앱 UI 연결 유지.

### 0) (선택) macOS 앱도 소스에서 실행

macOS 앱도 최신 버전으로 사용하고 싶다면:

```bash
./scripts/restart-mac.sh
```

### 1) 개발용 Gateway 시작

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch`는 Gateway를 watch 모드로 실행하며, TypeScript 파일 변경 시 자동으로 리로드됩니다.

### 2) macOS 앱을 실행 중인 Gateway에 연결

**OpenClaw.app**에서:

- Connection Mode: **Local**
  앱이 설정된 포트에서 실행 중인 Gateway에 연결됩니다.

### 3) 확인

- 앱 내 Gateway 상태가 **"Using existing gateway …"**로 표시되어야 합니다.
- 또는 CLI로 확인:

```bash
openclaw health
```

### 주의 사항

- **포트 오류:** Gateway WS 기본값은 `ws://127.0.0.1:18789`입니다. 앱과 CLI가 동일한 포트를 사용하는지 확인하세요.
- **상태 저장 위치:**
  - 자격 증명: `~/.openclaw/credentials/`
  - 세션: `~/.openclaw/agents/<agentId>/sessions/`
  - 로그: `/tmp/openclaw/`

## 자격 증명 저장 위치

인증 디버깅이나 백업 대상 결정 시 참고하세요:

- **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Telegram 봇 토큰**: 설정/환경 변수 또는 `channels.telegram.tokenFile`
- **Discord 봇 토큰**: 설정/환경 변수 (토큰 파일 미지원)
- **Slack 토큰**: 설정/환경 변수 (`channels.slack.*`)
- **페어링 허용 목록**: `~/.openclaw/credentials/<channel>-allowFrom.json`
- **모델 인증 프로필**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **레거시 OAuth 가져오기**: `~/.openclaw/credentials/oauth.json`
  자세한 내용: [보안](/gateway/security#credential-storage-map).

## 업데이트 (설정 유지하면서)

- `~/.openclaw/workspace`와 `~/.openclaw/`는 "개인 데이터"로 취급하세요. 개인 프롬프트/설정을 `openclaw` 저장소에 넣지 마세요.
- 소스 업데이트: `git pull` + `pnpm install` (lockfile 변경 시) + `pnpm gateway:watch` 계속 사용.

## Linux (systemd 사용자 서비스)

Linux 설치는 systemd **사용자** 서비스를 사용합니다. 기본적으로 systemd는 로그아웃/유휴 시 사용자 서비스를 중지하며, 이로 인해 Gateway가 종료됩니다. 온보딩에서 lingering을 활성화하려고 시도합니다 (sudo 비밀번호 요청 가능). 아직 비활성화 상태라면 다음을 실행하세요:

```bash
sudo loginctl enable-linger $USER
```

상시 가동 또는 다중 사용자 서버의 경우, 사용자 서비스 대신 **시스템** 서비스를 고려하세요 (lingering 불필요). systemd 관련 내용은 [Gateway 운영 가이드](/gateway)를 참조하세요.

## 관련 문서

- [Gateway 운영 가이드](/gateway) (플래그, 프로세스 관리, 포트)
- [Gateway 설정](/gateway/configuration) (설정 스키마 + 예제)
- [Discord](/channels/discord) 및 [Telegram](/channels/telegram) (응답 태그 + replyToMode 설정)
- [OpenClaw 어시스턴트 설정](/start/openclaw)
- [macOS 앱](/platforms/macos) (Gateway 라이프사이클)
