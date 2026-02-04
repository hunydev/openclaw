---
read_when:
  - 온보딩 마법사 실행 또는 설정 시
  - 새 기기 설정 시
summary: CLI 온보딩 마법사 - Gateway, 워크스페이스, 채널 및 스킬의 단계별 설정 가이드
title: 온보딩 마법사
x-i18n:
  generated_at: "2026-02-03T00:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 571302dcf63a0c700cab6b54964e524d75d98315d3b35fafe7232d2ce8199e83
  source_path: start/wizard.md
  workflow: 9
---

# 온보딩 마법사 (CLI)

온보딩 마법사는 macOS, Linux, 또는 Windows(WSL2 권장)에서 OpenClaw를 설정하는 **권장 방법**입니다. 로컬 Gateway 또는 원격 Gateway 연결, 채널, 스킬, 워크스페이스 기본값을 하나의 안내 흐름으로 구성합니다.

주요 진입점:

```bash
openclaw onboard
```

가장 빠른 첫 대화: Control UI를 엽니다(채널 설정 불필요). `openclaw dashboard`를 실행하고 브라우저에서 대화하세요. 문서: [대시보드](/web/dashboard).

후속 재설정:

```bash
openclaw configure
```

권장: Brave Search API 키를 설정하면 에이전트가 `web_search`를 사용할 수 있습니다(`web_fetch`는 키 없이도 작동). 가장 쉬운 방법: `openclaw configure --section web`을 실행하면 `tools.web.search.apiKey`가 저장됩니다. 문서: [웹 도구](/tools/web).

## 빠른 시작 vs 고급 모드

마법사는 **빠른 시작**(기본값) 또는 **고급**(전체 제어) 모드로 시작합니다.

**빠른 시작**은 기본값을 유지합니다:

- 로컬 Gateway (loopback)
- 기본 워크스페이스(또는 기존 워크스페이스)
- Gateway 포트 **18789**
- Gateway 인증 **Token**(loopback에서도 자동 생성)
- Tailscale 노출 **Off**
- Telegram + WhatsApp DM은 기본적으로 **allowlist**(전화번호 입력 요청)

**고급**은 모든 단계를 표시합니다(모드, 워크스페이스, Gateway, 채널, 데몬, 스킬).

## 마법사의 기능

**로컬 모드(기본)**에서 안내하는 항목:

- 모델/인증(OpenAI Code (Codex) 구독 OAuth, Anthropic API 키(권장) 또는 setup-token(붙여넣기), MiniMax/GLM/Moonshot/AI Gateway 옵션)
- 워크스페이스 위치 + 부트스트랩 파일
- Gateway 설정(포트/바인드/인증/Tailscale)
- 제공자(Telegram, WhatsApp, Discord, Google Chat, Mattermost(플러그인), Signal)
- 데몬 설치(LaunchAgent / systemd 사용자 유닛)
- 상태 점검
- 스킬(권장)

**원격 모드**는 로컬 클라이언트가 다른 위치의 Gateway에 연결하도록만 구성합니다. 원격 호스트에서는 아무것도 설치하거나 변경하지 **않습니다**.

격리된 에이전트(별도의 워크스페이스 + 세션 + 인증)를 추가하려면:

```bash
openclaw agents add <name>
```

팁: `--json`은 비대화형 모드를 의미하지 **않습니다**. 스크립트에는 `--non-interactive`(및 `--workspace`)를 사용하세요.

## 흐름 상세(로컬)

1. **기존 설정 감지**
   - `~/.openclaw/openclaw.json`이 존재하면 **유지 / 수정 / 초기화** 중 선택합니다.
   - 마법사를 다시 실행해도 명시적으로 **초기화**를 선택하거나 `--reset`을 전달하지 않는 한 아무것도 삭제하지 **않습니다**.
   - 설정이 유효하지 않거나 레거시 키가 포함된 경우, 마법사가 중지되고 계속하기 전에 `openclaw doctor`를 실행하라고 요청합니다.
   - 초기화는 `trash`를 사용합니다(`rm` 사용 안 함). 범위 선택 가능:
     - 설정만
     - 설정 + 자격 증명 + 세션
     - 전체 초기화(워크스페이스도 제거)

2. **모델/인증**
   - **Anthropic API 키(권장)**: `ANTHROPIC_API_KEY`가 있으면 사용하거나 키를 입력받아 데몬용으로 저장합니다.
   - **Anthropic OAuth (Claude Code CLI)**: macOS에서는 Keychain 항목 "Claude Code-credentials"를 확인합니다(launchd 시작 시 차단되지 않도록 "항상 허용" 선택). Linux/Windows에서는 `~/.claude/.credentials.json`이 있으면 재사용합니다.
   - **Anthropic 토큰(setup-token 붙여넣기)**: 아무 기기에서나 `claude setup-token`을 실행한 후 토큰을 붙여넣습니다(이름 지정 가능, 빈칸 = 기본값).
   - **OpenAI Code (Codex) 구독 (Codex CLI)**: `~/.codex/auth.json`이 존재하면 마법사가 재사용할 수 있습니다.
   - **OpenAI Code (Codex) 구독 (OAuth)**: 브라우저 흐름. `code#state`를 붙여넣습니다.
     - 모델이 설정되지 않았거나 `openai/*`인 경우 `agents.defaults.model`을 `openai-codex/gpt-5.2`로 설정합니다.
   - **OpenAI API 키**: `OPENAI_API_KEY`가 있으면 사용하거나 키를 입력받아 `~/.openclaw/.env`에 저장합니다(launchd가 읽을 수 있도록).
   - **OpenCode Zen(다중 모델 프록시)**: `OPENCODE_API_KEY`(또는 `OPENCODE_ZEN_API_KEY`)를 입력받습니다(https://opencode.ai/auth 에서 발급).
   - **API 키**: 키를 저장해 줍니다.
   - **Vercel AI Gateway(다중 모델 프록시)**: `AI_GATEWAY_API_KEY`를 입력받습니다.
   - 자세한 내용: [Vercel AI Gateway](/providers/vercel-ai-gateway)
   - **MiniMax M2.1**: 설정이 자동으로 기록됩니다.
   - 자세한 내용: [MiniMax](/providers/minimax)
   - **Synthetic(Anthropic 호환)**: `SYNTHETIC_API_KEY`를 입력받습니다.
   - 자세한 내용: [Synthetic](/providers/synthetic)
   - **Moonshot (Kimi K2)**: 설정이 자동으로 기록됩니다.
   - **Kimi Coding**: 설정이 자동으로 기록됩니다.
   - 자세한 내용: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
   - **건너뛰기**: 인증을 아직 구성하지 않습니다.
   - 감지된 옵션에서 기본 모델을 선택하거나 제공자/모델을 수동으로 입력합니다.
   - 마법사가 모델 점검을 실행하고, 구성된 모델이 알 수 없거나 인증이 누락된 경우 경고합니다.

- OAuth 자격 증명은 `~/.openclaw/credentials/oauth.json`에 저장됩니다. 인증 프로필은 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`에 저장됩니다(API 키 + OAuth).
- 자세한 내용: [/concepts/oauth](/concepts/oauth)

3. **워크스페이스**
   - 기본값 `~/.openclaw/workspace`(구성 가능).
   - 에이전트 부트스트랩 의식에 필요한 워크스페이스 파일을 생성합니다.
   - 전체 워크스페이스 레이아웃 + 백업 가이드: [에이전트 워크스페이스](/concepts/agent-workspace)

4. **Gateway**
   - 포트, 바인드, 인증 모드, Tailscale 노출.
   - 인증 권장 사항: loopback에서도 **Token**을 유지하여 로컬 WS 클라이언트가 인증해야 합니다.
   - 모든 로컬 프로세스를 완전히 신뢰하는 경우에만 인증을 비활성화하세요.
   - non-loopback 바인드는 여전히 인증이 필요합니다.

5. **채널**
   - [WhatsApp](/channels/whatsapp): 선택적 QR 로그인.
   - [Telegram](/channels/telegram): 봇 토큰.
   - [Discord](/channels/discord): 봇 토큰.
   - [Google Chat](/channels/googlechat): 서비스 계정 JSON + 웹훅 대상.
   - [Mattermost](/channels/mattermost) (플러그인): 봇 토큰 + 기본 URL.
   - [Signal](/channels/signal): 선택적 `signal-cli` 설치 + 계정 설정.
   - [iMessage](/channels/imessage): 로컬 `imsg` CLI 경로 + DB 접근.
   - DM 보안: 기본값은 페어링입니다. 첫 DM은 코드를 전송합니다. `openclaw pairing approve <channel> <code>`로 승인하거나 allowlist를 사용하세요.

6. **데몬 설치**
   - macOS: LaunchAgent
     - 로그인된 사용자 세션이 필요합니다. headless의 경우 사용자 정의 LaunchDaemon을 사용하세요(제공되지 않음).
   - Linux(및 WSL2를 통한 Windows): systemd 사용자 유닛
     - 마법사가 `loginctl enable-linger <user>`를 통해 linger를 활성화하여 로그아웃 후에도 Gateway가 실행됩니다.
     - sudo를 요청할 수 있습니다(`/var/lib/systemd/linger` 쓰기). 먼저 sudo 없이 시도합니다.
   - **런타임 선택:** Node(권장, WhatsApp/Telegram에 필요). Bun은 **권장하지 않습니다**.

7. **상태 점검**
   - Gateway를 시작하고(필요한 경우) `openclaw health`를 실행합니다.
   - 팁: `openclaw status --deep`은 상태 출력에 Gateway 상태 점검을 추가합니다(도달 가능한 Gateway 필요).

8. **스킬(권장)**
   - 사용 가능한 스킬을 읽고 요구 사항을 확인합니다.
   - Node 관리자를 선택합니다: **npm / pnpm**(bun 권장하지 않음).
   - 선택적 의존성을 설치합니다(일부는 macOS에서 Homebrew 사용).

9. **완료**
   - 요약 + 다음 단계(추가 기능을 위한 iOS/Android/macOS 앱 포함).

- GUI가 감지되지 않으면, 마법사가 브라우저를 여는 대신 Control UI용 SSH 포트 포워딩 안내를 출력합니다.
- Control UI 에셋이 없으면, 마법사가 빌드를 시도합니다. 대안은 `pnpm ui:build`입니다(UI 의존성 자동 설치).

## 원격 모드

원격 모드는 로컬 클라이언트가 다른 위치의 Gateway에 연결하도록 구성합니다.

설정할 항목:

- 원격 Gateway URL (`ws://...`)
- 원격 Gateway가 인증을 요구하면 토큰(권장)

참고 사항:

- 원격 설치나 데몬 변경은 수행되지 않습니다.
- Gateway가 loopback 전용이면 SSH 터널링 또는 tailnet을 사용하세요.
- 검색 힌트:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)

## 다른 에이전트 추가

`openclaw agents add <name>`을 사용하여 자체 워크스페이스, 세션, 인증 프로필을 가진 별도의 에이전트를 생성합니다. `--workspace` 없이 실행하면 마법사가 시작됩니다.

설정되는 항목:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

참고 사항:

- 기본 워크스페이스는 `~/.openclaw/workspace-<agentId>` 형식을 따릅니다.
- 인바운드 메시지를 라우팅하려면 `bindings`를 추가하세요(마법사가 이를 수행할 수 있음).
- 비대화형 플래그: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## 비대화형 모드

온보딩을 자동화하거나 스크립트화하려면 `--non-interactive`를 사용하세요:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

기계 판독 가능한 요약을 원하면 `--json`을 추가하세요.

Gemini 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Z.AI 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Vercel AI Gateway 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Moonshot 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Synthetic 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

OpenCode Zen 예시:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

에이전트 추가(비대화형) 예시:

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway 마법사 RPC

Gateway는 RPC를 통해 마법사 흐름을 노출합니다(`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). 클라이언트(macOS 앱, Control UI)는 온보딩 로직을 다시 구현하지 않고도 단계를 렌더링할 수 있습니다.

## Signal 설정 (signal-cli)

마법사가 GitHub 릴리스에서 `signal-cli`를 설치할 수 있습니다:

- 적절한 릴리스 에셋을 다운로드합니다.
- `~/.openclaw/tools/signal-cli/<version>/`에 저장합니다.
- 설정에 `channels.signal.cliPath`를 기록합니다.

참고 사항:

- JVM 빌드는 **Java 21**이 필요합니다.
- 네이티브 빌드가 있으면 우선 사용됩니다.
- Windows는 WSL2를 사용합니다. signal-cli 설치는 WSL 내부에서 Linux 흐름을 따릅니다.

## 마법사가 기록하는 내용

`~/.openclaw/openclaw.json`의 일반적인 필드:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers`(Minimax 선택 시)
- `gateway.*`(모드, 바인드, 인증, Tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- 채널 allowlist(Slack/Discord/Matrix/Microsoft Teams) - 프롬프트 중 동의 시 적용(가능하면 이름이 ID로 해석됨).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add`는 `agents.list[]`와 선택적 `bindings`를 기록합니다.

WhatsApp 자격 증명은 `~/.openclaw/credentials/whatsapp/<accountId>/`에 저장됩니다. 세션은 `~/.openclaw/agents/<agentId>/sessions/`에 저장됩니다.

일부 채널은 플러그인으로 제공됩니다. 온보딩 중 채널을 선택하면 마법사가 설정하기 전에 먼저 설치를 요청합니다(npm 또는 로컬 경로).

## 관련 문서

- macOS 앱 온보딩: [온보딩](/start/onboarding)
- 설정 참조: [Gateway 설정](/gateway/configuration)
- 제공자: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [iMessage](/channels/imessage)
- 스킬: [스킬](/tools/skills), [스킬 설정](/tools/skills-config)
