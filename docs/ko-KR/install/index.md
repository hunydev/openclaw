---
read_when:
  - OpenClaw 설치
  - GitHub에서 설치하고 싶을 때
summary: OpenClaw 설치 (권장 설치 프로그램, 전역 설치 또는 소스에서 설치)
title: 설치
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: b26f48c116c26c163ee0090fb4c3e29622951bd427ecaeccba7641d97cfdf17a
  source_path: install/index.md
  workflow: 14
---

# 설치

특별한 이유가 없다면 설치 프로그램을 사용하세요. CLI를 설정하고 온보딩을 실행합니다.

## 빠른 설치 (권장)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

다음 단계 (온보딩을 건너뛴 경우):

```bash
openclaw onboard --install-daemon
```

## 시스템 요구 사항

- **Node >=22**
- macOS, Linux 또는 WSL2를 통한 Windows
- `pnpm`은 소스에서 빌드할 때만 필요

## 설치 방법 선택

### 1) 설치 프로그램 스크립트 (권장)

npm을 통해 `openclaw`를 전역 설치하고 온보딩을 실행합니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

설치 프로그램 인수:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
```

자세한 내용: [설치 프로그램 내부 구조](/install/installer).

비대화형 (온보딩 건너뛰기):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
```

### 2) 전역 설치 (수동)

Node가 이미 설치되어 있다면:

```bash
npm install -g openclaw@latest
```

전역 libvips가 이미 설치되어 있고 (macOS에서는 주로 Homebrew를 통해) `sharp` 설치가 실패하면, 사전 빌드된 바이너리를 강제로 사용하세요:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

`sharp: Please add node-gyp to your dependencies`가 표시되면 빌드 도구를 설치하거나 (macOS: Xcode CLT + `npm install -g node-gyp`), 위의 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 우회 방법으로 네이티브 빌드를 건너뛸 수 있습니다.

또는 pnpm 사용:

```bash
pnpm add -g openclaw@latest
pnpm approve-builds -g                # openclaw, node-llama-cpp, sharp 등 승인
pnpm add -g openclaw@latest           # postinstall 스크립트 실행을 위해 재실행
```

pnpm은 빌드 스크립트가 있는 패키지에 대해 명시적 승인을 요구합니다. 첫 설치 시 "Ignored build scripts" 경고가 표시되면 `pnpm approve-builds -g`를 실행하고 나열된 패키지를 선택한 다음, postinstall 스크립트를 실행하기 위해 설치를 재실행하세요.

그런 다음:

```bash
openclaw onboard --install-daemon
```

### 3) 소스에서 설치 (기여자/개발 용도)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 첫 실행 시 자동으로 UI 의존성 설치
pnpm build
openclaw onboard --install-daemon
```

팁: 전역 설치가 아직 안 되어 있다면 `pnpm openclaw ...`로 저장소 명령을 실행할 수 있습니다.

### 4) 기타 설치 옵션

- Docker: [Docker](/install/docker)
- Nix: [Nix](/install/nix)
- Ansible: [Ansible](/install/ansible)
- Bun (CLI 전용): [Bun](/install/bun)

## 설치 후

- 온보딩 실행: `openclaw onboard --install-daemon`
- 빠른 확인: `openclaw doctor`
- 게이트웨이 상태 확인: `openclaw status` + `openclaw health`
- 대시보드 열기: `openclaw dashboard`

## 설치 방식: npm vs git (설치 프로그램)

설치 프로그램은 두 가지 방식을 지원합니다:

- `npm` (기본값): `npm install -g openclaw@latest`
- `git`: GitHub에서 클론/빌드 후 소스 체크아웃에서 실행

### CLI 인수

```bash
# 명시적으로 npm 사용
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method npm

# GitHub에서 설치 (소스 체크아웃)
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

일반 인수:

- `--install-method npm|git`
- `--git-dir <path>` (기본값: `~/openclaw`)
- `--no-git-update` (기존 체크아웃 사용 시 `git pull` 건너뛰기)
- `--no-prompt` (프롬프트 비활성화; CI/자동화에 필수)
- `--dry-run` (실행할 작업 출력; 변경 없음)
- `--no-onboard` (온보딩 건너뛰기)

### 환경 변수

동등한 환경 변수 (자동화용):

- `OPENCLAW_INSTALL_METHOD=git|npm`
- `OPENCLAW_GIT_DIR=...`
- `OPENCLAW_GIT_UPDATE=0|1`
- `OPENCLAW_NO_PROMPT=1`
- `OPENCLAW_DRY_RUN=1`
- `OPENCLAW_NO_ONBOARD=1`
- `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` (기본값: `1`; `sharp`가 시스템 libvips로 컴파일하는 것 방지)

## 문제 해결: `openclaw`를 찾을 수 없음 (PATH 문제)

빠른 진단:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

`$(npm prefix -g)/bin` (macOS/Linux) 또는 `$(npm prefix -g)` (Windows)가 `echo "$PATH"` 출력에 **없으면**, 셸이 전역 npm 바이너리 (`openclaw` 포함)를 찾을 수 없습니다.

수정: 셸 시작 파일에 추가 (zsh: `~/.zshrc`, bash: `~/.bashrc`):

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Windows에서는 `npm prefix -g` 출력을 PATH에 추가하세요.

그런 다음 새 터미널을 열거나 (zsh에서 `rehash` / bash에서 `hash -r` 실행).

## 업데이트 / 제거

- 업데이트: [업데이트](/install/updating)
- 새 컴퓨터로 마이그레이션: [마이그레이션](/install/migrating)
- 제거: [제거](/install/uninstall)
