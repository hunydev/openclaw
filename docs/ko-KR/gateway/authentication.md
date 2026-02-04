---
read_when:
  - 모델 인증 또는 OAuth 만료 문제 디버깅 시
  - 인증 또는 자격 증명 저장소 관련 문서 작성 시
summary: 모델 인증 - OAuth, API 키 및 setup-token
title: 인증
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 66fa2c64ff374c9cfcdb4e7a951b0d164d512295e65513eb682f12191b75e557
  source_path: gateway/authentication.md
  workflow: 14
---

# 인증

OpenClaw는 모델 프로바이더에 대해 OAuth 및 API 키를 통한 인증을 지원합니다. Anthropic 계정의 경우 **API 키**를 권장합니다. Claude 구독 액세스의 경우 `claude setup-token`으로 생성된 장기 유효 토큰을 사용하세요.

전체 OAuth 플로우 및 저장소 레이아웃은 [/concepts/oauth](/concepts/oauth)를 참조하세요.

## 권장 Anthropic 설정 (API 키)

Anthropic을 직접 사용하는 경우 API 키를 사용하세요.

1. Anthropic 콘솔에서 API 키 생성.
2. **Gateway 호스트** (`openclaw gateway`를 실행하는 컴퓨터)에 배치.

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Gateway가 systemd/launchd 하에서 실행되는 경우 데몬이 읽을 수 있도록 `~/.openclaw/.env`에 키를 넣는 것을 권장:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

그런 다음 데몬 (또는 Gateway 프로세스)을 재시작하고 다시 확인:

```bash
openclaw models status
openclaw doctor
```

환경 변수를 직접 관리하지 않으려면 온보딩 마법사가 데몬용 API 키를 저장할 수 있습니다: `openclaw onboard`.

환경 변수 상속에 대한 자세한 내용은 [도움말](/help) 참조 (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic: setup-token (구독 인증)

Anthropic의 경우 **API 키**가 권장됩니다. Claude 구독을 사용하는 경우 setup-token 플로우도 지원됩니다. **Gateway 호스트**에서 실행:

```bash
claude setup-token
```

그런 다음 OpenClaw에 붙여넣기:

```bash
openclaw models auth setup-token --provider anthropic
```

토큰이 다른 컴퓨터에서 생성된 경우 수동으로 붙여넣기:

```bash
openclaw models auth paste-token --provider anthropic
```

다음과 같은 Anthropic 오류가 표시되면:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

...대신 Anthropic API 키를 사용하세요.

수동 토큰 입력 (모든 프로바이더에 적용; `auth-profiles.json`에 기록하고 구성 업데이트):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

자동화에 적합한 검사 (만료/누락 시 종료 코드 `1`, 곧 만료 시 `2`):

```bash
openclaw models status --check
```

선택적 운영 스크립트 (systemd/Termux) 문서: [/automation/auth-monitoring](/automation/auth-monitoring)

> `claude setup-token`은 대화형 TTY가 필요합니다.

## 모델 인증 상태 확인

```bash
openclaw models status
openclaw doctor
```

## 사용할 자격 증명 제어

### 세션별 (채팅 명령)

`/model <alias-or-id>@<profileId>`를 사용하여 현재 세션에 특정 프로바이더 자격 증명 지정 (예시 구성 ID: `anthropic:default`, `anthropic:work`).

`/model` (또는 `/model list`)을 사용하여 컴팩트 선택기 열기; `/model status`로 전체 뷰 (후보 + 다음 인증 구성 및 구성된 프로바이더 엔드포인트 세부 정보) 확인.

### 에이전트별 (CLI 재정의)

에이전트에 대해 명시적 인증 구성 순서 재정의 설정 (해당 에이전트의 `auth-profiles.json`에 저장):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

특정 에이전트 지정에는 `--agent <id>` 사용; 생략하면 구성된 기본 에이전트 사용.

## 문제 해결

### "No credentials found"

Anthropic 토큰 구성이 누락된 경우 **Gateway 호스트**에서 `claude setup-token`을 실행한 다음 다시 확인:

```bash
openclaw models status
```

### 토큰이 곧 만료/이미 만료

`openclaw models status`를 실행하여 어떤 구성이 곧 만료되는지 확인. 구성이 누락된 경우 `claude setup-token`을 다시 실행하고 토큰을 다시 붙여넣기.

## 요구 사항

- Claude Max 또는 Pro 구독 (`claude setup-token`용)
- Claude Code CLI 설치됨 (`claude` 명령 사용 가능)
