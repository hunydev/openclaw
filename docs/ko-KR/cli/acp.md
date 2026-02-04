---
read_when:
  - ACP 기반 IDE 통합을 설정할 때
  - Gateway로의 ACP 세션 라우팅을 디버깅할 때
summary: IDE 통합을 위한 ACP 브릿지 실행
title: acp
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# acp

OpenClaw Gateway와 통신하는 ACP(Agent Client Protocol) 브릿지를 실행합니다.

이 명령은 stdio를 통해 ACP 프로토콜을 사용하여 IDE와 통신하고 WebSocket을 통해 프롬프트를 Gateway로 전달합니다. ACP 세션을 Gateway 세션 키에 매핑합니다.

## 사용법

```bash
openclaw acp

# 원격 Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# 기존 세션 키에 연결
openclaw acp --session agent:main:main

# 레이블로 연결(이미 존재해야 함)
openclaw acp --session-label "support inbox"

# 첫 번째 프롬프트 전에 세션 키 재설정
openclaw acp --session agent:main:main --reset-session
```

## ACP 클라이언트(디버그)

IDE 없이 브릿지의 설치 무결성 검사를 위해 내장 ACP 클라이언트를 사용합니다.
ACP 브릿지를 시작하고 대화형으로 프롬프트를 입력할 수 있습니다.

```bash