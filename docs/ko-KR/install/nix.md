---
read_when:
  - 재현 가능하고 롤백 가능한 설치를 원할 때
  - 이미 Nix/NixOS/Home Manager를 사용 중일 때
  - 모든 것을 고정하고 선언적으로 관리하고 싶을 때
summary: Nix를 사용한 선언적 OpenClaw 설치
title: Nix
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: f1452194cfdd74613b5b3ab90b0d506eaea2d16b147497987710d6ad658312ba
  source_path: install/nix.md
  workflow: 14
---

# Nix 설치

Nix로 OpenClaw를 실행하는 권장 방법은 **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — 즉시 사용 가능한 Home Manager 모듈입니다.

## 빠른 시작

다음을 AI 에이전트 (Claude, Cursor 등)에게 붙여넣으세요:

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 전체 가이드: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw 저장소가 Nix 설치의 권위 있는 출처입니다. 이 페이지는 간략한 개요일 뿐입니다.

## 얻게 되는 것

- 게이트웨이 + macOS 앱 + 도구 (whisper, spotify, cameras) — 모두 고정 버전
- 재시작 후에도 유지되는 Launchd 서비스
- 선언적 구성이 있는 플러그인 시스템
- 즉각적인 롤백: `home-manager switch --rollback`

---

## Nix 모드 런타임 동작

`OPENCLAW_NIX_MODE=1`이 설정되면 (nix-openclaw가 자동 설정):

OpenClaw는 구성을 결정론적으로 만들고 자동 설치 플로우를 비활성화하는 **Nix 모드**를 지원합니다.
다음 환경 변수를 내보내 활성화:

```bash
OPENCLAW_NIX_MODE=1
```

macOS에서 GUI 앱은 자동으로 셸 환경 변수를 상속받지 않습니다.
defaults를 통해 Nix 모드를 활성화할 수도 있습니다:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### 구성 + 상태 경로

OpenClaw는 `OPENCLAW_CONFIG_PATH`에서 JSON5 구성을 읽고 가변 데이터를 `OPENCLAW_STATE_DIR`에 저장합니다.

- `OPENCLAW_STATE_DIR` (기본값: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (기본값: `$OPENCLAW_STATE_DIR/openclaw.json`)

Nix 아래에서 실행할 때 런타임 상태와 구성이
불변 저장소로 들어가지 않도록 이 경로를 Nix 관리 위치로 명시적으로 설정하세요.

### Nix 모드에서의 런타임 동작

- 자동 설치 및 자체 변경 플로우 비활성화
- 누락된 의존성은 Nix 특정 수정 제안 표시
- 활성화되면 UI에 읽기 전용 Nix 모드 배너 표시

## 패키징 참고 (macOS)

macOS 패키징 플로우에는 안정적인 Info.plist 템플릿이 필요합니다:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh)는 이 템플릿을 앱 번들에 복사하고 동적 필드를
패치합니다 (bundle ID, 버전/빌드 번호, Git SHA, Sparkle 키). 이를 통해 plist가 SwiftPM
패키징 및 Nix 빌드에서 결정론적으로 유지됩니다 (전체 Xcode 도구 체인에 의존하지 않음).

## 관련 항목
