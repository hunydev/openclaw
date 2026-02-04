---
read_when:
  - 새로운 어시스턴트 인스턴스 온보딩
  - 보안/권한 영향 검토
summary: 안전 주의사항을 포함한 OpenClaw 개인 어시스턴트 설정 가이드
title: 개인 어시스턴트 설정
x-i18n:
  generated_at: "2026-02-03T00:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 2763668c053abe34ea72c40d1306d3d1143099c58b1e3ef91c2e5a20cb2769e0
  source_path: start/openclaw.md
  workflow: 15
---

# OpenClaw로 개인 어시스턴트 만들기

OpenClaw는 **Pi** 에이전트를 위한 WhatsApp + Telegram + Discord + iMessage Gateway입니다. 플러그인을 통해 Mattermost도 지원합니다. 이 가이드는 "개인 어시스턴트" 설정을 다룹니다: 항상 작동하는 에이전트 역할을 하는 전용 WhatsApp 번호를 설정하는 방법입니다.

## ⚠️ 안전이 최우선입니다

에이전트에게 다음과 같은 권한을 부여하게 됩니다:

- 컴퓨터에서 명령어 실행 (Pi 도구 설정에 따라 다름)
- 워크스페이스의 파일 읽기/쓰기
- WhatsApp/Telegram/Discord/Mattermost(플러그인)를 통한 메시지 전송

보수적인 설정으로 시작하세요:

- 항상 `channels.whatsapp.allowFrom`을 설정하세요 (개인 Mac에서 전 세계에 개방된 상태로 실행하지 마세요).
- 어시스턴트용 전용 WhatsApp 번호를 사용하세요.
- Heartbeat는 기본적으로 30분마다 실행됩니다. 설정을 신뢰할 수 있을 때까지 `agents.defaults.heartbeat.every: "0m"`으로 비활성화하세요.

## 사전 요구사항

- Node **22+**
- OpenClaw가 PATH에 있어야 함 (권장: 전역 설치)
- 어시스턴트용 두 번째 전화번호 (SIM/eSIM/선불 SIM)

```bash
npm install -g openclaw@latest
# 또는: pnpm add -g openclaw@latest
```

소스에서 설치 (개발용):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 첫 실행 시 UI 의존성 자동 설치
pnpm build
pnpm link --global
```

## 두 대의 폰 설정 (권장)

다음과 같은 구성이 필요합니다:

```
내 폰 (개인용)                  두 번째 폰 (어시스턴트)
┌─────────────────┐           ┌─────────────────┐
│  내 WhatsApp    │  ──────▶  │  어시스턴트 WA  │
│  +1-555-YOU     │  메시지   │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ QR로 연결
                                       ▼
                              ┌─────────────────┐
                              │  내 Mac         │
                              │  (openclaw)      │
                              │    Pi 에이전트  │
                              └─────────────────┘
```

개인 WhatsApp을 OpenClaw에 연결하면, 당신에게 오는 모든 메시지가 "에이전트 입력"이 됩니다. 이건 보통 원하는 게 아닙니다.

## 5분 빠른 시작

1. WhatsApp Web 페어링 (QR 코드 표시; 어시스턴트 폰으로 스캔):

```bash
openclaw channels login
```

2. Gateway 시작 (계속 실행 상태로 유지):

```bash
openclaw gateway --port 18789
```

3. `~/.openclaw/openclaw.json`에 최소 설정 파일 생성:

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

이제 허용 목록에 있는 폰에서 어시스턴트 번호로 메시지를 보내세요.

온보딩이 완료되면 Gateway 토큰이 포함된 대시보드가 자동으로 열리고 토큰화된 링크가 출력됩니다. 나중에 다시 열려면: `openclaw dashboard`.

## 에이전트에게 워크스페이스 제공하기 (AGENTS)

OpenClaw는 워크스페이스 디렉토리에서 작동 지침과 "메모리"를 읽습니다.

기본적으로 OpenClaw는 `~/.openclaw/workspace`를 에이전트 워크스페이스로 사용하며, 설정/첫 에이전트 실행 시 자동으로 생성됩니다 (초기 `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` 포함). `BOOTSTRAP.md`는 워크스페이스가 완전히 새로 생성될 때만 만들어집니다 (삭제 후 다시 나타나지 않습니다).

팁: 이 폴더를 OpenClaw의 "메모리"로 생각하고 git 저장소로 만드세요 (가능하면 비공개). 그러면 `AGENTS.md` + 메모리 파일들이 백업됩니다. git이 설치되어 있으면 새 워크스페이스는 자동으로 초기화됩니다.

```bash
openclaw setup
```

전체 워크스페이스 구조 + 백업 가이드: [에이전트 워크스페이스](/concepts/agent-workspace)
메모리 워크플로우: [메모리](/concepts/memory)

선택사항: `agents.defaults.workspace`로 다른 워크스페이스를 선택할 수 있습니다 (`~` 지원).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

이미 저장소에서 워크스페이스 파일을 제공하고 있다면, 부트스트랩 파일 생성을 완전히 비활성화할 수 있습니다:

```json5
{
  agent: {
    skipBootstrap: true,
  },
}
```

## "어시스턴트"로 만드는 설정

OpenClaw는 기본적으로 좋은 어시스턴트 설정을 제공하지만, 보통 다음을 조정하고 싶을 것입니다:

- `SOUL.md`의 페르소나/지침
- thinking 기본값 (필요한 경우)
- heartbeat (신뢰할 수 있게 된 후)

예시:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 0으로 시작; 나중에 활성화.
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## 세션과 메모리

- 세션 파일: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
- 세션 메타데이터 (토큰 사용량, 마지막 라우트 등): `~/.openclaw/agents/<agentId>/sessions/sessions.json` (레거시: `~/.openclaw/sessions/sessions.json`)
- `/new` 또는 `/reset`은 해당 채팅의 새 세션을 시작합니다 (`resetTriggers`로 설정 가능). 단독으로 보내면 에이전트가 리셋 확인을 위해 짧은 인사로 응답합니다.
- `/compact [instructions]`는 세션 컨텍스트를 압축하고 남은 컨텍스트 예산을 보고합니다.

## Heartbeat (능동 모드)

기본적으로 OpenClaw는 30분마다 다음 프롬프트로 heartbeat를 실행합니다:
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
`agents.defaults.heartbeat.every: "0m"`으로 비활성화할 수 있습니다.

- `HEARTBEAT.md`가 존재하지만 실질적으로 비어있는 경우 (빈 줄과 `# Heading` 같은 마크다운 헤더만 있는 경우), OpenClaw는 API 호출을 절약하기 위해 heartbeat 실행을 건너뜁니다.
- 파일이 없으면 heartbeat가 여전히 실행되고 모델이 무엇을 할지 결정합니다.
- 에이전트가 `HEARTBEAT_OK`로 응답하면 (선택적으로 짧은 패딩 포함; `agents.defaults.heartbeat.ackMaxChars` 참조), OpenClaw는 해당 heartbeat의 아웃바운드 전송을 억제합니다.
- Heartbeat는 전체 에이전트 턴을 실행합니다 — 더 짧은 간격은 더 많은 토큰을 소모합니다.

```json5
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## 미디어 입출력

인바운드 첨부파일 (이미지/오디오/문서)은 템플릿을 통해 명령어에 전달될 수 있습니다:

- `{{MediaPath}}` (로컬 임시 파일 경로)
- `{{MediaUrl}}` (의사 URL)
- `{{Transcript}}` (오디오 전사가 활성화된 경우)

에이전트의 아웃바운드 첨부파일: `MEDIA:<path-or-url>`을 별도의 줄에 포함합니다 (공백 없이). 예시:

```
Here's the screenshot.
MEDIA:https://example.com/screenshot.png
```

OpenClaw는 이를 추출하여 텍스트와 함께 미디어로 전송합니다.

## 운영 체크리스트

```bash
openclaw status          # 로컬 상태 (자격증명, 세션, 대기 중인 이벤트)
openclaw status --all    # 전체 진단 (읽기 전용, 복사 가능)
openclaw status --deep   # Gateway 상태 프로브 추가 (Telegram + Discord)
openclaw health --json   # Gateway 상태 스냅샷 (WS)
```

로그는 `/tmp/openclaw/`에 저장됩니다 (기본: `openclaw-YYYY-MM-DD.log`).

## 다음 단계

- 웹 채팅: [웹 채팅](/web/webchat)
- Gateway 운영: [Gateway 운영 가이드](/gateway)
- 크론 + 웨이크업: [크론 작업](/automation/cron-jobs)
- macOS 메뉴바 앱: [OpenClaw macOS 앱](/platforms/macos)
- iOS 노드 앱: [iOS 앱](/platforms/ios)
- Android 노드 앱: [Android 앱](/platforms/android)
- Windows 상태: [Windows (WSL2)](/platforms/windows)
- Linux 상태: [Linux 앱](/platforms/linux)
- 보안: [보안](/gateway/security)
