---
read_when:
  - macOS 온보딩 도우미 설계 시
  - 인증 또는 신원 설정 구현 시
summary: OpenClaw 첫 실행 온보딩 흐름 (macOS 앱)
title: 온보딩
x-i18n:
  generated_at: "2026-02-03T00:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 3c2364d1b292bfab2ad1a037dea7dfee086fdb1a88253cf3565b9d352d1ad645
  source_path: start/onboarding.md
  workflow: 15
---

# 온보딩 (macOS 앱)

이 문서는 **현재** 첫 실행 온보딩 흐름을 설명합니다. 목표는 매끄러운 "0일차" 경험을 제공하는 것입니다: Gateway 실행 위치 선택, 인증 연결, 마법사 실행, 에이전트 자체 부트스트랩 완료.

## 페이지 순서 (현재)

1. 환영 + 보안 안내
2. **Gateway 선택** (로컬 / 원격 / 나중에 설정)
3. **인증 (Anthropic OAuth)** — 로컬 전용
4. **설정 마법사** (Gateway 기반)
5. **권한** (TCC 프롬프트)
6. **CLI** (선택 사항)
7. **온보딩 채팅** (전용 세션)
8. 준비 완료

## 1) 환영 + 보안 안내

표시된 보안 안내를 읽고 그에 따라 결정하세요.

## 2) 로컬 vs 원격

**Gateway**가 어디서 실행되나요?

- **로컬 (이 Mac):** 온보딩에서 OAuth 흐름을 실행하고 로컬에 자격 증명을 저장할 수 있습니다.
- **원격 (SSH/Tailnet 경유):** 온보딩에서 로컬로 OAuth를 실행하지 **않습니다**; 자격 증명은 Gateway 호스트에 있어야 합니다.
- **나중에 설정:** 설정을 건너뛰고 앱을 미구성 상태로 둡니다.

Gateway 인증 팁:

- 마법사는 이제 loopback에도 **토큰**을 생성하므로, 로컬 WS 클라이언트도 반드시 인증해야 합니다.
- 인증을 비활성화하면 모든 로컬 프로세스가 연결할 수 있습니다; 완전히 신뢰할 수 있는 머신에서만 이 옵션을 사용하세요.
- 다중 머신 접근이나 loopback이 아닌 바인딩에는 **토큰**을 사용하세요.

## 3) 로컬 전용 인증 (Anthropic OAuth)

macOS 앱은 Anthropic OAuth (Claude Pro/Max)를 지원합니다. 흐름은 다음과 같습니다:

- 브라우저에서 OAuth (PKCE) 열기
- 사용자에게 `code#state` 값을 붙여넣도록 요청
- 자격 증명을 `~/.openclaw/credentials/oauth.json`에 저장

다른 제공자 (OpenAI, 커스텀 API)는 현재 환경 변수나 설정 파일을 통해 구성합니다.

## 4) 설정 마법사 (Gateway 기반)

앱은 CLI와 동일한 설정 마법사를 실행할 수 있습니다. 이를 통해 온보딩이 Gateway 측 동작과 동기화되고, SwiftUI에서 로직을 중복 구현하지 않아도 됩니다.

## 5) 권한

온보딩에서 다음에 필요한 TCC 권한을 요청합니다:

- 알림
- 손쉬운 사용
- 화면 기록
- 마이크 / 음성 인식
- 자동화 (AppleScript)

## 6) CLI (선택 사항)

앱은 npm/pnpm을 통해 전역 `openclaw` CLI를 설치할 수 있어, 터미널 워크플로우와 launchd 작업이 바로 사용 가능합니다.

## 7) 온보딩 채팅 (전용 세션)

설정 완료 후, 앱은 전용 온보딩 채팅 세션을 열어 에이전트가 자기소개를 하고 다음 단계를 안내합니다. 이를 통해 첫 실행 안내와 일반 대화가 분리됩니다.

## 에이전트 부트스트랩 의식

첫 에이전트 실행 시, OpenClaw는 워크스페이스를 부트스트랩합니다 (기본값 `~/.openclaw/workspace`):

- `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md` 시드 파일 생성
- 짧은 Q&A 의식 실행 (한 번에 하나의 질문)
- 신원 및 선호도를 `IDENTITY.md`, `USER.md`, `SOUL.md`에 저장
- 완료 시 `BOOTSTRAP.md` 제거로 한 번만 실행되도록 보장

## 선택 사항: Gmail 훅 (수동)

Gmail Pub/Sub 설정은 현재 수동 단계입니다. 다음을 사용하세요:

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

자세한 내용은 [/automation/gmail-pubsub](/automation/gmail-pubsub)를 참조하세요.

## 원격 모드 참고 사항

Gateway가 다른 머신에서 실행될 때, 자격 증명과 워크스페이스 파일은 **해당 호스트에** 있습니다. 원격 모드에서 OAuth가 필요하면 Gateway 호스트에 다음을 생성하세요:

- `~/.openclaw/credentials/oauth.json`
- `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
