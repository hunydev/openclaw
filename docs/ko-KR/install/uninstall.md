---
read_when:
  - 컴퓨터에서 OpenClaw를 제거하고 싶을 때
  - 제거 후에도 게이트웨이 서비스가 계속 실행될 때
summary: OpenClaw 완전 제거 (CLI, 서비스, 상태, 작업 공간)
title: 제거
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 6673a755c5e1f90a807dd8ac92a774cff6d1bc97d125c75e8bf72a40e952a777
  source_path: install/uninstall.md
  workflow: 15
---

# 제거

두 가지 방법:

- **간편 방법**: `openclaw`가 아직 설치되어 있을 때.
- **수동 서비스 제거**: CLI가 삭제되었지만 서비스가 계속 실행될 때.

## 간편 방법 (CLI가 아직 설치됨)

권장: 내장 제거 프로그램 사용:

```bash
openclaw uninstall
```

비대화형 모드 (자동화 / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

수동 단계 (동일 효과):

1. 게이트웨이 서비스 중지:

```bash
openclaw gateway stop
```

2. 게이트웨이 서비스 제거 (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. 상태 및 구성 삭제:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

`OPENCLAW_CONFIG_PATH`를 상태 디렉토리 외부의 커스텀 위치로 설정했다면 해당 파일도 삭제하세요.

4. 작업 공간 삭제 (선택 사항, 에이전트 파일 제거):

```bash
rm -rf ~/.openclaw/workspace
```

5. CLI 설치 제거 (사용한 방법 선택):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. macOS 앱을 설치한 경우:

```bash
rm -rf /Applications/OpenClaw.app
```

참고:

- 프로필 (`--profile` / `OPENCLAW_PROFILE`)을 사용한 경우, 각 상태 디렉토리에 대해 3단계를 반복하세요 (기본값 `~/.openclaw-<profile>`).
- 원격 모드에서는 상태 디렉토리가 **게이트웨이 호스트**에 있으므로 거기에서도 1-4단계를 수행해야 합니다.

## 수동 서비스 제거 (CLI 미설치)

게이트웨이 서비스가 계속 실행되지만 `openclaw`가 더 이상 존재하지 않을 때 사용하세요.

### macOS (launchd)

기본 레이블은 `bot.molt.gateway`입니다 (또는 `bot.molt.<profile>`; 레거시 `com.openclaw.*`가 여전히 존재할 수 있음):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

프로필을 사용한 경우 레이블과 plist 이름을 `bot.molt.<profile>`로 교체하세요. 레거시 `com.openclaw.*` plist 파일이 있으면 함께 제거하세요.

### Linux (systemd 사용자 단위)

기본 단위 이름은 `openclaw-gateway.service`입니다 (또는 `openclaw-gateway-<profile>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (예약된 작업)

기본 작업 이름은 `OpenClaw Gateway`입니다 (또는 `OpenClaw Gateway (<profile>)`).
작업 스크립트는 상태 디렉토리 아래에 있습니다.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

프로필을 사용한 경우 해당 작업 이름과 `~\.openclaw-<profile>\gateway.cmd`를 삭제하세요.

## 일반 설치 vs 소스 체크아웃

### 일반 설치 (install.sh / npm / pnpm / bun)

`https://openclaw.ai/install.sh` 또는 `install.ps1`을 사용했다면, CLI는 `npm install -g openclaw@latest`로 설치되었습니다.
`npm rm -g openclaw`로 제거하세요 (다른 방식을 사용했다면 `pnpm remove -g` / `bun remove -g`).

### 소스 체크아웃 (git clone)

저장소 체크아웃에서 실행한 경우 (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1. 저장소를 삭제하기 **전에** 먼저 게이트웨이 서비스를 제거하세요 (위의 간편 방법 또는 수동 서비스 제거 사용).
2. 저장소 디렉토리 삭제.
3. 위와 같이 상태 및 작업 공간 제거.
