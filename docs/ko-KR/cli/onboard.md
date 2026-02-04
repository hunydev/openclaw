---
read_when:
  - Gateway, 작업 공간, 인증, 채널 및 Skills의 안내 설정을 원할 때
summary: "`openclaw onboard`(대화형 온보딩 마법사)의 CLI 레퍼런스"
title: onboard
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw onboard`

대화형 온보딩 마법사(로컬 또는 원격 Gateway 설정).

관련 내용:

- 마법사 가이드: [온보딩](/start/onboarding)

## 예시

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

플로우 설명:

- `quickstart`: 최소 프롬프트, Gateway 토큰 자동 생성.
- `manual`: 완전한 포트/바인딩/인증 프롬프트(`advanced`의 별칭).
- 채팅을 시작하는 가장 빠른 방법: `openclaw dashboard`(콘솔 UI, 채널 설정 불필요).
