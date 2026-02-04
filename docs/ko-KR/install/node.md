---
read_when:
  - OpenClaw를 설치했지만 `openclaw`가 "command not found"일 때
  - 새 컴퓨터에서 Node.js/npm 구성 중
  - npm install -g ...가 권한 또는 PATH 문제로 실패할 때
summary: Node.js + npm 설치 상태 확인 - 버전, PATH 및 전역 설치
title: Node.js + npm (PATH 설치 상태 확인)
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9f6d83be362e3e148ddf07d47e57c51679c22687263d3b5131cccbef2e37c598
  source_path: install/node.md
  workflow: 15
---

# Node.js + npm (PATH 설치 상태 확인)

OpenClaw의 런타임 기준 요구 사항은 **Node 22+**입니다.

`npm install -g openclaw@latest`를 실행할 수 있지만 이후 `openclaw: command not found`가 표시된다면, 거의 항상 **PATH** 문제입니다: npm이 전역 바이너리를 저장하는 디렉토리가 셸의 PATH에 없습니다.

## 빠른 진단

실행:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

`$(npm prefix -g)/bin` (macOS/Linux) 또는 `$(npm prefix -g)` (Windows)가 `echo "$PATH"` 출력에 **없으면**, 셸이 전역 npm 바이너리 (`openclaw` 포함)를 찾을 수 없습니다.

## 수정: npm의 전역 bin 디렉토리를 PATH에 추가

1. 전역 npm prefix 찾기:

```bash
npm prefix -g
```

2. 셸 시작 파일에 전역 npm bin 디렉토리 추가:

- zsh: `~/.zshrc`
- bash: `~/.bashrc`

예시 (`npm prefix -g` 출력 경로로 교체):

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

그런 다음 **새 터미널**을 열거나 (zsh에서 `rehash` / bash에서 `hash -r` 실행).

Windows에서는 `npm prefix -g` 출력을 PATH에 추가하세요.

## 수정: `sudo npm install -g` / 권한 오류 방지 (Linux)

`npm install -g ...`가 `EACCES`로 실패하면, npm의 전역 prefix를 사용자 쓰기 가능 디렉토리로 전환하세요:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

`export PATH=...` 줄을 셸 시작 파일에 영구화하세요.

## 권장 Node 설치 방법

다음 조건을 만족하는 Node/npm 설치는 문제가 적습니다:

- Node를 최신 상태로 유지 (22+)
- 전역 npm bin 디렉토리가 안정적이고 새 셸에서 PATH에 있음

일반적인 선택:

- macOS: Homebrew (`brew install node`) 또는 버전 관리자
- Linux: 선호하는 버전 관리자, 또는 Node 22+를 제공하는 배포판 지원 방식
- Windows: 공식 Node 설치 프로그램, `winget` 또는 Windows Node 버전 관리자

버전 관리자 (nvm/fnm/asdf 등)를 사용하는 경우, 일상적으로 사용하는 셸 (zsh 또는 bash)에서 초기화되었는지 확인하세요. 그래야 설치 프로그램 실행 시 설정된 PATH가 적용됩니다.
