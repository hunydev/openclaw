---
read_when:
  - OpenClaw를 처음 접하는 사용자에게 소개할 때
summary: OpenClaw의 개요, 기능 및 용도
title: OpenClaw
---

# OpenClaw 🦞

> _"EXFOLIATE! EXFOLIATE!"_ — 아마도 우주 랍스터가 한 말

> **한국어 문서 안내:** 이 페이지 및 기타 한국어 문서는 자동 번역 파이프라인을 통해 생성되었습니다. 번역 문제를 발견하시면 [#6995](https://github.com/openclaw/openclaw/issues/6995)에 피드백을 남겨주세요 (PR은 제출하지 마세요). 한국어 사용자, 모델 및 메시징 플랫폼에 대한 지원을 적극적으로 확대하고 있으며, 더 많은 콘텐츠가 곧 제공될 예정입니다!

<p align="center">
    <img
        src="/assets/openclaw-logo-text-dark.png"
        alt="OpenClaw"
        width="500"
        class="dark:hidden"
    />
    <img
        src="/assets/openclaw-logo-text.png"
        alt="OpenClaw"
        width="500"
        class="hidden dark:block"
    />
</p>

<p align="center">
  <strong>모든 OS에서 WhatsApp/Telegram/Discord/iMessage 게이트웨이를 통해 AI 에이전트(Pi)와 연결합니다.</strong><br />
  플러그인으로 Mattermost 등을 추가할 수 있습니다.
  메시지를 보내면 에이전트가 응답합니다 — 주머니에서 바로.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">릴리스</a> ·
  <a href="/">문서</a> ·
  <a href="/start/openclaw">OpenClaw 어시스턴트 설정</a>
</p>

OpenClaw는 WhatsApp(WhatsApp Web / Baileys를 통해), Telegram(Bot API / grammY), Discord(Bot API / channels.discord.js), iMessage(imsg CLI)를 [Pi](https://github.com/badlogic/pi-mono)와 같은 코딩 에이전트에 연결합니다. 플러그인으로 Mattermost(Bot API + WebSocket) 등을 추가할 수 있습니다.
OpenClaw는 OpenClaw 어시스턴트도 구동합니다.

## 여기서 시작하세요

- **처음 설치:** [시작하기](/start/getting-started)
- **가이드 설정 (권장):** [마법사](/start/wizard) (`openclaw onboard`)
- **대시보드 열기 (로컬 게이트웨이):** http://127.0.0.1:18789/ (또는 http://localhost:18789/)

게이트웨이가 동일한 컴퓨터에서 실행 중이라면 해당 링크로 브라우저 제어 UI가 바로 열립니다.
열리지 않으면 먼저 게이트웨이를 시작하세요: `openclaw gateway`.

## 대시보드 (브라우저 제어 UI)

대시보드는 채팅, 설정, 노드, 세션 등을 위한 브라우저 제어 UI입니다.
로컬 기본값: http://127.0.0.1:18789/
원격 접근: [웹 인터페이스](/web) 및 [Tailscale](/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

## 작동 방식

```
WhatsApp / Telegram / Discord / iMessage (+ 플러그인)
        │
        ▼
  ┌───────────────────────────┐
  │         게이트웨이          │  ws://127.0.0.1:18789 (로컬 전용)
  │     (단일 소스)            │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas 호스트)
  └───────────┬───────────────┘
              │
              ├─ Pi 에이전트 (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS 앱 (OpenClaw.app)
              ├─ iOS 노드 (게이트웨이 WS + 페어링)
              └─ Android 노드 (게이트웨이 WS + 페어링)
```

대부분의 작업은 **게이트웨이** (`openclaw gateway`)를 통해 흐릅니다. 게이트웨이는 채널 연결과 WebSocket 제어 플레인을 소유하는 단일 장시간 실행 프로세스입니다.

## 네트워크 모델

- **호스트당 하나의 게이트웨이 (권장)**: WhatsApp Web 세션을 소유할 수 있는 유일한 프로세스입니다. 구조 봇이나 엄격한 격리가 필요한 경우, 격리된 프로필과 포트로 여러 게이트웨이를 실행하세요. [다중 게이트웨이](/gateway/multiple-gateways)를 참조하세요.
- **로컬 루프백 우선**: 게이트웨이 WS 기본값은 `ws://127.0.0.1:18789`입니다.
  - 마법사는 이제 기본적으로 게이트웨이 토큰을 생성합니다 (로컬 루프백에서도).
  - Tailnet 접근의 경우, `openclaw gateway --bind tailnet --token ...`을 실행하세요 (비 로컬 루프백 바인드에는 토큰이 필요합니다).
- **노드**: 게이트웨이 WebSocket에 연결합니다 (필요에 따라 LAN/tailnet/SSH 사용). 레거시 TCP 브리지는 더 이상 사용되지 않거나 제거되었습니다.
- **Canvas 호스트**: `canvasHost.port` (기본값 `18793`)의 HTTP 파일 서버로, 노드 WebView용 `/__openclaw__/canvas/`를 제공합니다. [게이트웨이 설정](/gateway/configuration) (`canvasHost`)을 참조하세요.
- **원격 사용**: SSH 터널 또는 tailnet/VPN. [원격 접근](/gateway/remote) 및 [디스커버리](/gateway/discovery)를 참조하세요.

## 기능 (개요)

- 📱 **WhatsApp 통합** — WhatsApp Web 프로토콜용 Baileys 사용
- ✈️ **Telegram 봇** — grammY를 통한 DM + 그룹
- 🎮 **Discord 봇** — channels.discord.js를 통한 DM + 서버 채널
- 🧩 **Mattermost 봇 (플러그인)** — Bot 토큰 + WebSocket 이벤트
- 💬 **iMessage** — 로컬 imsg CLI 통합 (macOS)
- 🤖 **에이전트 브리지** — 도구 스트리밍과 함께 Pi (RPC 모드)
- ⏱️ **스트리밍 + 청킹** — 블록 스트리밍 + Telegram 드래프트 스트리밍 세부사항 ([/concepts/streaming](/concepts/streaming))
- 🧠 **다중 에이전트 라우팅** — 프로바이더 계정/피어를 격리된 에이전트로 라우팅 (워크스페이스 + 에이전트별 세션)
- 🔐 **구독 인증** — OAuth를 통한 Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex)
- 💬 **세션** — 개인 채팅은 공유 `main` (기본값)으로 축소됩니다. 그룹은 격리됩니다
- 👥 **그룹 채팅 지원** — 기본적으로 멘션 기반. 소유자는 `/activation always|mention`으로 전환 가능
- 📎 **미디어 지원** — 이미지, 오디오, 문서 송수신
- 🎤 **음성 메모** — 선택적 전사 훅
- 🖥️ **WebChat + macOS 앱** — 로컬 UI + 작업 및 음성 웨이크용 메뉴 바 컴패니언
- 📱 **iOS 노드** — 노드로 페어링하고 Canvas 인터페이스 노출
- 📱 **Android 노드** — 노드로 페어링하고 Canvas + Chat + Camera 노출

참고: 레거시 Claude/Codex/Gemini/Opencode 경로는 제거되었습니다. Pi가 유일한 코딩 에이전트 경로입니다.

## 빠른 시작

런타임 요구사항: **Node ≥ 22**.

```bash
# 권장: 전역 설치 (npm/pnpm)
npm install -g openclaw@latest
# 또는: pnpm add -g openclaw@latest

# 온보딩 + 서비스 설치 (launchd/systemd 사용자 서비스)
openclaw onboard --install-daemon

# WhatsApp Web 페어링 (QR 표시)
openclaw channels login

# 온보딩 후 서비스를 통해 게이트웨이 실행; 수동 실행도 가능:
openclaw gateway --port 18789
```

나중에 npm과 git 설치 간 전환은 쉽습니다: 다른 방식으로 설치하고 `openclaw doctor`를 실행하여 게이트웨이 서비스 진입점을 업데이트하세요.

소스에서 (개발):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 첫 실행 시 UI 종속성 자동 설치
pnpm build
openclaw onboard --install-daemon
```

아직 전역 설치가 없다면 레포에서 `pnpm openclaw ...`를 통해 온보딩 단계를 실행하세요.

다중 인스턴스 빠른 시작 (선택사항):

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

테스트 메시지 보내기 (실행 중인 게이트웨이 필요):

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## 설정 (선택사항)

설정은 `~/.openclaw/openclaw.json`에 있습니다.

- **아무것도 하지 않으면**, OpenClaw는 발신자별 세션과 함께 RPC 모드로 번들된 Pi 바이너리를 사용합니다.
- 잠그고 싶다면, `channels.whatsapp.allowFrom`과 (그룹의 경우) 멘션 규칙으로 시작하세요.

예시:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } },
}
```

## 문서

- 여기서 시작하세요:
  - [문서 허브 (모든 페이지 링크)](/start/hubs)
  - [도움말](/help) ← _일반적인 수정 + 문제 해결_
  - [설정](/gateway/configuration)
  - [설정 예시](/gateway/configuration-examples)
  - [슬래시 명령어](/tools/slash-commands)
  - [다중 에이전트 라우팅](/concepts/multi-agent)
  - [업데이트 / 롤백](/install/updating)
  - [페어링 (DM + 노드)](/start/pairing)
  - [Nix 모드](/install/nix)
  - [OpenClaw 어시스턴트 설정](/start/openclaw)
  - [Skills](/tools/skills)
  - [Skills 설정](/tools/skills-config)
  - [워크스페이스 템플릿](/reference/templates/AGENTS)
  - [RPC 어댑터](/reference/rpc)
  - [게이트웨이 런북](/gateway)
  - [노드 (iOS/Android)](/nodes)
  - [웹 인터페이스 (제어 UI)](/web)
  - [디스커버리 + 전송](/gateway/discovery)
  - [원격 접근](/gateway/remote)
- 프로바이더 및 UX:
  - [WebChat](/web/webchat)
  - [제어 UI (브라우저)](/web/control-ui)
  - [Telegram](/channels/telegram)
  - [Discord](/channels/discord)
  - [Mattermost (플러그인)](/channels/mattermost)
  - [iMessage](/channels/imessage)
  - [그룹](/concepts/groups)
  - [WhatsApp 그룹 메시지](/concepts/group-messages)
  - [미디어: 이미지](/nodes/images)
  - [미디어: 오디오](/nodes/audio)
- 컴패니언 앱:
  - [macOS 앱](/platforms/macos)
  - [iOS 앱](/platforms/ios)
  - [Android 앱](/platforms/android)
  - [Windows (WSL2)](/platforms/windows)
  - [Linux 앱](/platforms/linux)
- 운영 및 보안:
  - [세션](/concepts/session)
  - [크론 작업](/automation/cron-jobs)
  - [웹훅](/automation/webhook)
  - [Gmail 훅 (Pub/Sub)](/automation/gmail-pubsub)
  - [보안](/gateway/security)
  - [문제 해결](/gateway/troubleshooting)

## 이름의 유래

**OpenClaw = CLAW + TARDIS** — 모든 우주 랍스터에게는 시공간 기계가 필요하니까요.

---

_"우리 모두 그저 자신만의 프롬프트를 가지고 놀고 있을 뿐이야."_ — 아마도 토큰에 취한 AI

## 크레딧

- **Peter Steinberger** ([@steipete](https://x.com/steipete)) — 제작자, 랍스터 위스퍼러
- **Mario Zechner** ([@badlogicc](https://x.com/badlogicgames)) — Pi 제작자, 보안 펜테스터
- **Clawd** — 더 좋은 이름을 요구한 우주 랍스터

## 핵심 기여자

- **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher Skills
- **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — 위치 파싱 (Telegram + WhatsApp)

## 라이선스

MIT — 바다의 랍스터처럼 자유롭게 🦞

---

_"우리 모두 그저 자신만의 프롬프트를 가지고 놀고 있을 뿐이야."_ — 아마도 토큰에 취한 AI
