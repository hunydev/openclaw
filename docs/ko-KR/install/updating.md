---
read_when:
  - OpenClaw 업데이트
  - 업데이트 후 문제 발생 시
summary: OpenClaw 안전하게 업데이트 (전역 설치 또는 소스 설치), 롤백 전략
title: 업데이트
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 612b2519cf3e4a2c2d0f01575c3fa75ab1c88a6fed9e59477bf27395beda03c1
  source_path: install/updating.md
  workflow: 15
---

# 업데이트

OpenClaw는 빠르게 반복됩니다 ("1.0"에 도달하지 않음). 업데이트를 인프라 릴리스처럼 취급하세요: 업데이트 → 검사 실행 → 재시작 (또는 자동 재시작하는 `openclaw update` 사용) → 확인.

## 권장 방법: 웹사이트 설치 프로그램 재실행 (제자리 업그레이드)

**선호** 업데이트 경로는 웹사이트의 설치 프로그램을 재실행하는 것입니다. 기존 설치를 감지하고, 제자리에서 업그레이드하며, 필요시 `openclaw doctor`를 실행합니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

참고:

- 온보딩 마법사를 다시 실행하지 않으려면 `--no-onboard`를 추가하세요.
- **소스 설치**의 경우:
  ```bash
  curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
  ```
  설치 프로그램은 저장소 작업 공간이 깨끗할 때**만** `git pull --rebase`를 수행합니다.
- **전역 설치**의 경우, 스크립트 내부에서 `npm install -g openclaw@latest`를 사용합니다.
- 호환성 참고: `openclaw`는 여전히 호환성 shim으로 사용 가능합니다.

## 업데이트 전

- 설치 방법 파악: **전역 설치** (npm/pnpm) 또는 **소스 설치** (git clone).
- 게이트웨이 실행 방법 파악: **포그라운드 터미널** 또는 **관리 서비스** (launchd/systemd).
- 커스텀 구성 백업:
  - 구성: `~/.openclaw/openclaw.json`
  - 자격 증명: `~/.openclaw/credentials/`
  - 작업 공간: `~/.openclaw/workspace`

## 업데이트 (전역 설치)

전역 설치 (하나 선택):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

게이트웨이 런타임으로 Bun 사용은 **권장하지 않습니다** (WhatsApp/Telegram 버그 있음).

업데이트 채널 전환 (git + npm 설치):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

일회성 지정 태그/버전 설치에는 `--tag <dist-tag|version>`을 사용하세요.

채널 의미 및 릴리스 노트는 [개발 채널](/install/development-channels)을 참조하세요.

참고: npm 설치에서 게이트웨이 시작 시 업데이트 힌트를 기록합니다 (현재 채널 태그 확인). `update.checkOnStart: false`로 비활성화하세요.

그런 다음:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

참고:

- 게이트웨이가 서비스로 실행 중이면, PID를 직접 종료하는 대신 `openclaw gateway restart` 권장.
- 특정 버전에 고정했다면 아래 "롤백/버전 고정"을 참조하세요.

## 업데이트 (`openclaw update`)

**소스 설치** (git checkout)의 경우 권장:

```bash
openclaw update
```

비교적 안전한 업데이트 플로우를 수행합니다:

- 작업 공간이 깨끗해야 함.
- 선택한 채널로 전환 (태그 또는 브랜치).
- 구성된 업스트림에서 풀 및 리베이스 (dev 채널).
- 의존성 설치, 빌드, 대시보드 UI 빌드, `openclaw doctor` 실행.
- 기본적으로 게이트웨이 재시작 (`--no-restart`로 건너뛰기).

**npm/pnpm**으로 설치한 경우 (git 메타데이터 없음), `openclaw update`가 패키지 관리자를 통해 업데이트를 시도합니다. 설치 방법을 감지할 수 없으면 "업데이트 (전역 설치)"를 사용하세요.

## 업데이트 (대시보드 UI / RPC)

대시보드 UI는 **업데이트 및 재시작** 기능 (RPC: `update.run`)을 제공합니다. 이는:

1. `openclaw update`와 동일한 소스 업데이트 플로우 실행 (git checkout만).
2. 재시작 센티넬 파일 및 구조화된 보고서 (stdout/stderr 끝부분) 작성.
3. 게이트웨이 재시작 및 가장 최근 활성 세션에 보고서 전송.

리베이스가 실패하면 게이트웨이가 중단되고 업데이트 적용 없이 재시작됩니다.

## 업데이트 (소스 설치)

저장소 체크아웃 디렉토리에서:

권장 방법:

```bash
openclaw update
```

수동 방법 (대략 동등):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # 첫 실행 시 자동으로 UI 의존성 설치
openclaw doctor
openclaw health
```

참고:

- 패키지된 `openclaw` 바이너리 ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs))를 실행하거나 Node로 `dist/`를 실행할 때 `pnpm build`가 중요합니다.
- 전역 설치 없이 저장소 체크아웃에서 실행한다면 `pnpm openclaw ...`로 CLI 명령을 실행하세요.
- TypeScript에서 직접 실행 (`pnpm openclaw ...`)하면 일반적으로 재빌드가 필요 없지만, **구성 마이그레이션은 여전히 적용됨** → doctor 실행.
- 전역 설치와 git 설치 간 전환이 쉽습니다: 다른 방식을 설치하고 `openclaw doctor`를 실행하면 게이트웨이 서비스 진입점이 현재 설치로 다시 작성됩니다.

## 필수: `openclaw doctor`

Doctor는 "안전한 업데이트" 명령입니다. 의도적으로 단순합니다: 수정 + 마이그레이션 + 경고.

참고: **소스 설치** (git checkout)를 사용하는 경우, `openclaw doctor`가 먼저 `openclaw update` 실행을 제안합니다.

일반적으로 수행하는 작업:

- 더 이상 사용되지 않는 구성 키 / 레거시 구성 파일 위치 마이그레이션.
- DM 정책 감사 및 위험한 "열린" 설정에 대해 경고.
- 게이트웨이 상태 확인 및 재시작 제안.
- 레거시 게이트웨이 서비스 (launchd/systemd; 레거시 schtasks)를 현재 OpenClaw 서비스로 감지 및 마이그레이션.
- Linux에서 systemd 사용자 lingering 보장 (로그아웃 후에도 게이트웨이 계속 실행).

자세한 내용: [Doctor](/gateway/doctor)

## 게이트웨이 시작/중지/재시작

CLI (모든 OS):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

서비스 관리 사용 시:

- macOS launchd (앱 번들 LaunchAgent): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (`bot.molt.<profile>` 사용; 레거시 `com.openclaw.*` 여전히 작동)
- Linux systemd 사용자 서비스: `systemctl --user restart openclaw-gateway[-<profile>].service`
- Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  - `launchctl`/`systemctl`은 서비스가 설치된 경우에만 작동; 그렇지 않으면 `openclaw gateway install` 실행.

운영 매뉴얼 및 전체 서비스 레이블: [게이트웨이 운영 매뉴얼](/gateway)

## 롤백/버전 고정 (문제 발생 시)

### 버전 고정 (전역 설치)

알려진 작동 버전 설치 (`<version>`을 마지막으로 작동하던 버전으로 교체):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

팁: 현재 게시된 버전을 보려면 `npm view openclaw version`을 실행하세요.

그런 다음 재시작하고 doctor 재실행:

```bash
openclaw doctor
openclaw gateway restart
```

### 버전 고정 (소스 설치) 날짜 기준

특정 날짜의 커밋 선택 (예: "2026-01-01 기준 main 브랜치 상태"):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

그런 다음 의존성 재설치 및 재시작:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

나중에 최신으로 돌아가려면:

```bash
git checkout main
git pull
```

## 막혔을 때

- `openclaw doctor`를 다시 실행하고 출력을 주의 깊게 읽으세요 (보통 수정 방법을 알려줍니다).
- 참조: [문제 해결](/gateway/troubleshooting)
- Discord에서 질문: https://discord.gg/clawd
