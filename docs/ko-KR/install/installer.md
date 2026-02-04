---
read_when:
  - openclaw.ai/install.sh의 작동 방식을 알고 싶을 때
  - 설치 자동화 (CI / 헤드리스 환경)
  - GitHub 체크아웃에서 설치하고 싶을 때
summary: 설치 프로그램 스크립트 작동 방식 (install.sh + install-cli.sh), 인수 및 자동화
title: 설치 프로그램 내부 구조
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9e0a19ecb5da0a395030e1ccf0d4bedf16b83946b3432c5399d448fe5d298391
  source_path: install/installer.md
  workflow: 14
---

# 설치 프로그램 내부 구조

OpenClaw는 두 개의 설치 프로그램 스크립트를 제공합니다 (`openclaw.ai`에서 호스팅):

- `https://openclaw.ai/install.sh` — "권장" 설치 프로그램 (기본값은 전역 npm 설치; GitHub 체크아웃에서도 설치 가능)
- `https://openclaw.ai/install-cli.sh` — root 권한 불필요 CLI 설치 프로그램 (독립 Node가 있는 prefix 디렉토리에 설치)
- `https://openclaw.ai/install.ps1` — Windows PowerShell 설치 프로그램 (기본값 npm; 선택적 git 설치)

현재 인수/동작 확인:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
```

Windows (PowerShell) 도움말:

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

설치 프로그램이 완료되었지만 새 터미널에서 `openclaw`를 찾을 수 없다면, 대부분 Node/npm PATH 문제입니다. 참조: [설치](/install#nodejs--npm-path-sanity).

## install.sh (권장)

기능 개요:

- OS 감지 (macOS / Linux / WSL).
- Node.js **22+** 확인 (macOS는 Homebrew; Linux는 NodeSource).
- 설치 방식 선택:
  - `npm` (기본값): `npm install -g openclaw@latest`
  - `git`: 소스 체크아웃 클론/빌드 후 래퍼 스크립트 설치
- Linux에서: 필요시 npm prefix를 `~/.npm-global`로 전환하여 전역 npm 권한 오류 방지.
- 기존 설치 업그레이드 시: `openclaw doctor --non-interactive` 실행 (최선의 노력).
- git 설치 시: 설치/업데이트 후 `openclaw doctor --non-interactive` 실행 (최선의 노력).
- 기본적으로 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 설정하여 `sharp` 네이티브 설치 문제 완화 (시스템 libvips 컴파일 방지).

`sharp`가 전역 설치된 libvips에 연결되길 *원하는* 경우 (또는 디버깅 중이라면):

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.ai/install.sh | bash
```

### 발견 가능성 / "git 설치" 힌트

**기존 OpenClaw 소스 체크아웃 디렉토리 내에서** 설치 프로그램을 실행하면 (`package.json` + `pnpm-workspace.yaml`로 감지), 다음을 묻습니다:

- 이 체크아웃 업데이트 및 사용 (`git`)
- 또는 전역 npm 설치로 마이그레이션 (`npm`)

비대화형 컨텍스트 (TTY 없음 / `--no-prompt`)에서는 `--install-method git|npm`을 전달하거나 (`OPENCLAW_INSTALL_METHOD` 설정) 해야 하며, 그렇지 않으면 스크립트가 종료 코드 `2`로 종료됩니다.

### Git이 필요한 이유

`--install-method git` 경로 (클론 / 풀)에는 Git이 필요합니다.

`npm` 설치의 경우 Git이 *일반적으로* 필요하지 않지만, 일부 환경에서는 여전히 필요합니다 (예: git URL로 패키지나 의존성을 가져올 때). 설치 프로그램은 현재 새 배포판에서 `spawn git ENOENT` 오류를 방지하기 위해 Git 존재를 확인합니다.

### 새 Linux에서 npm이 `EACCES`를 보고하는 이유

일부 Linux 설정에서 (특히 시스템 패키지 관리자 또는 NodeSource를 통해 Node 설치 후), npm의 전역 prefix가 root 소유 위치를 가리킵니다. 이때 `npm install -g ...`가 `EACCES` / `mkdir` 권한 오류를 보고합니다.

`install.sh`는 다음으로 prefix를 전환하여 이를 완화합니다:

- `~/.npm-global` (존재하면 `~/.bashrc` / `~/.zshrc`의 `PATH`에 추가)

## install-cli.sh (root 권한 불필요 CLI 설치 프로그램)

이 스크립트는 `openclaw`를 prefix 디렉토리 (기본값: `~/.openclaw`)에 설치하고, 해당 prefix 아래에 전용 Node 런타임을 설치하므로 시스템 Node/npm을 건드리지 않으려는 컴퓨터에서 사용할 수 있습니다.

도움말:

```bash
curl -fsSL https://openclaw.ai/install-cli.sh | bash -s -- --help
```

## install.ps1 (Windows PowerShell)

기능 개요:

- Node.js **22+** 확인 (winget/Chocolatey/Scoop 또는 수동 설치).
- 설치 방식 선택:
  - `npm` (기본값): `npm install -g openclaw@latest`
  - `git`: 소스 체크아웃 클론/빌드 후 래퍼 스크립트 설치
- 업그레이드 및 git 설치 시 `openclaw doctor --non-interactive` 실행 (최선의 노력).

예시:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

환경 변수:

- `OPENCLAW_INSTALL_METHOD=git|npm`
- `OPENCLAW_GIT_DIR=...`

Git 요구 사항:

`-InstallMethod git`을 선택했지만 Git이 설치되어 있지 않으면, 설치 프로그램이 Git for Windows 링크 (`https://git-scm.com/download/win`)를 출력하고 종료합니다.

일반적인 Windows 문제:

- **npm error spawn git / ENOENT**: Git for Windows를 설치하고 PowerShell을 다시 열어 설치 프로그램을 재실행하세요.
- **"openclaw"은(는) 인식할 수 없는 명령입니다**: npm 전역 bin 폴더가 PATH에 없습니다. 대부분의 시스템에서 `%AppData%\\npm`입니다. `npm config get prefix`를 실행하고 `\\bin`을 PATH에 추가한 다음 PowerShell을 다시 열 수도 있습니다.
