---
read_when:
  - 처음부터 시작하는 초기 설정
  - 설치 → 온보딩 → 첫 메시지 전송까지의 가장 빠른 경로를 원할 때
summary: 시작 가이드: 처음부터 첫 메시지까지 (위저드, 인증, 채널, 페어링)
title: 시작하기
x-i18n:
  generated_at: "2026-02-03T00:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: d0ebc83c10efc569eaf6fb32368a29ef75a373f15da61f3499621462f08aff63
  source_path: start/getting-started.md
  workflow: 9
---

# 시작하기

> 이 페이지 및 기타 한국어 문서는 자동 번역 파이프라인을 통해 생성되었습니다. 번역 문제를 발견하시면 [#6995](https://github.com/openclaw/openclaw/issues/6995)에 피드백을 남겨주세요.

목표: **처음** → **첫 번째 채팅 성공** (합리적인 기본 설정)까지 최대한 빠르게 도달합니다.

가장 빠른 채팅 방법: Control UI를 엽니다 (채널 설정 불필요). `openclaw dashboard`를 실행하고 브라우저에서 채팅하거나, Gateway 호스트에서 `http://127.0.0.1:18789/`를 엽니다.
문서: [Dashboard](/web/dashboard) 및 [Control UI](/web/control-ui).

권장 경로: **CLI 온보딩 위저드** (`openclaw onboard`)를 사용하세요. 다음을 설정합니다:

- 모델/인증 (OAuth 권장)
- Gateway 설정
- 채널 (WhatsApp/Telegram/Discord/Mattermost (플러그인)/...)
- 페어링 기본값 (안전한 DM)
- 워크스페이스 부트스트랩 + Skills
- 선택적 백그라운드 서비스

더 자세한 참조 페이지가 필요하면 다음으로 이동하세요: [위저드](/start/wizard), [설정](/start/setup), [페어링](/start/pairing), [보안](/gateway/security).

샌드박스 참고: `agents.defaults.sandbox.mode: "non-main"`은 `session.mainKey` (기본값 `"main"`)를 사용하므로 그룹/채널 세션은 샌드박스 처리됩니다. 메인 에이전트가 항상 호스트에서 실행되도록 하려면 명시적인 에이전트별 오버라이드를 설정하세요:

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

## 0) 사전 요구 사항

- Node `>=22`
- `pnpm` (선택 사항; 소스에서 빌드하는 경우 권장)
- **권장:** 웹 검색을 위한 Brave Search API 키. 가장 쉬운 방법:
  `openclaw configure --section web` (`tools.web.search.apiKey`에 저장됨).
  [Web tools](/tools/web) 참조.

macOS: 앱을 빌드할 계획이라면 Xcode / CLT를 설치하세요. CLI + Gateway만 사용한다면 Node만으로 충분합니다.
Windows: **WSL2** 사용 (Ubuntu 권장). WSL2를 강력히 권장합니다. 네이티브 Windows는 테스트되지 않았고, 문제가 더 많으며, 도구 호환성이 떨어집니다. 먼저 WSL2를 설치한 다음 WSL 내에서 Linux 단계를 실행하세요. [Windows (WSL2)](/platforms/windows) 참조.

## 1) CLI 설치 (권장)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

설치 옵션 (설치 방법, 비대화형, GitHub에서 설치): [Install](/install).

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

대안 (전역 설치):

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

## 2) 온보딩 위저드 실행 (및 서비스 설치)

```bash
openclaw onboard --install-daemon
```

선택할 내용:

- **로컬 vs 원격** Gateway
- **인증**: OpenAI Code (Codex) 구독 (OAuth) 또는 API 키. Anthropic의 경우 API 키를 권장합니다. `claude setup-token`도 지원됩니다.
- **프로바이더**: WhatsApp QR 로그인, Telegram/Discord 봇 토큰, Mattermost 플러그인 토큰 등.
- **데몬**: 백그라운드 설치 (launchd/systemd; WSL2는 systemd 사용)
  - **런타임**: Node (권장; WhatsApp/Telegram 필수). Bun은 **권장하지 않습니다**.
- **Gateway 토큰**: 위저드가 기본적으로 생성하며 (loopback에서도) `gateway.auth.token`에 저장합니다.

위저드 문서: [위저드](/start/wizard)

### 인증: 저장 위치 (중요)

- **권장 Anthropic 경로:** API 키 설정 (위저드가 서비스용으로 저장 가능). `claude setup-token`: Claude Code 자격 증명을 재사용할 수 있습니다.

- OAuth 자격 증명 (레거시 가져오기): `~/.openclaw/credentials/oauth.json`
- 인증 프로필 (OAuth + API 키): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

헤드리스/서버 팁: 먼저 일반 머신에서 OAuth를 완료한 다음 `oauth.json`을 Gateway 호스트로 복사하세요.

## 3) Gateway 시작

온보딩 중 서비스를 설치했다면 Gateway가 이미 실행 중이어야 합니다:

```bash
openclaw gateway status
```

수동 실행 (포그라운드):

```bash
openclaw gateway --port 18789 --verbose
```

Dashboard (로컬 loopback): `http://127.0.0.1:18789/`
토큰이 구성된 경우 Control UI 설정에 붙여넣으세요 (`connect.params.auth.token`으로 저장됨).

⚠️ **Bun 경고 (WhatsApp + Telegram):** Bun은 이러한 채널에서 알려진 문제가 있습니다. WhatsApp 또는 Telegram을 사용하는 경우 **Node**로 Gateway를 실행하세요.

## 3.5) 빠른 확인 (2분)

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

## 4) 페어링 + 첫 번째 채팅 인터페이스 연결

### WhatsApp (QR 로그인)

```bash
openclaw channels login
```

WhatsApp → 설정 → 연결된 기기에서 스캔합니다.

WhatsApp 문서: [WhatsApp](/channels/whatsapp)

### Telegram / Discord / 기타

위저드가 토큰/설정을 작성할 수 있습니다. 수동 구성을 선호한다면 다음부터 시작하세요:

- Telegram: [Telegram](/channels/telegram)
- Discord: [Discord](/channels/discord)
- Mattermost (플러그인): [Mattermost](/channels/mattermost)

**Telegram DM 팁:** 첫 번째 DM은 페어링 코드를 반환합니다. 승인하세요 (다음 단계 참조), 그렇지 않으면 봇이 응답하지 않습니다.

## 5) DM 안전 (페어링 승인)

기본 정책: 알 수 없는 DM은 짧은 코드를 받고 메시지는 승인될 때까지 처리되지 않습니다.
첫 번째 DM에 응답이 없으면 페어링을 승인하세요:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

페어링 문서: [페어링](/start/pairing)

## 소스에서 설치 (개발)

OpenClaw 자체를 개발하는 경우 소스에서 실행하세요:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 첫 실행 시 UI 의존성 자동 설치
pnpm build
openclaw onboard --install-daemon
```

전역 설치가 아직 없다면 리포지토리에서 `pnpm openclaw ...`로 온보딩 단계를 실행하세요.
`pnpm build`는 A2UI 자산도 번들링합니다. 해당 단계만 실행해야 한다면 `pnpm canvas:a2ui:bundle`을 사용하세요.

Gateway (이 리포지토리에서):

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## 7) 엔드투엔드 확인

새 터미널에서 테스트 메시지를 보냅니다:

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

`openclaw health`에서 "인증이 구성되지 않음"이 표시되면 위저드로 돌아가서 OAuth/키 인증을 설정하세요 — 인증 없이는 에이전트가 응답할 수 없습니다.

팁: `openclaw status --all`은 복사해서 붙여넣기 좋은, 읽기 전용 디버그 리포트입니다.
