---
read_when:
  - 채널이 연결되었지만 메시지가 흐르지 않을 때
  - 채널 설정 오류 문제 해결(인텐트, 권한, 프라이버시 모드)
summary: 채널별 문제 해결 빠른 참조(Discord/Telegram/WhatsApp)
title: 채널 문제 해결
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 6542ee86b3e50929caeaab127642d135dfbc0d8a44876ec2df0fff15bf57cd63
  source_path: channels/troubleshooting.md
  workflow: 14
---

# 채널 문제 해결

먼저 실행:

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe`는 일반적인 채널 설정 오류를 감지할 때 경고를 출력하고 소규모 라이브 검사(자격 증명, 부분 권한/멤버십)를 포함합니다.

## 채널

- Discord: [/channels/discord#troubleshooting](/channels/discord#troubleshooting)
- Telegram: [/channels/telegram#troubleshooting](/channels/telegram#troubleshooting)
- WhatsApp: [/channels/whatsapp#troubleshooting-quick](/channels/whatsapp#troubleshooting-quick)

## Telegram 빠른 수정

- 로그에 `HttpError: Network request for 'sendMessage' failed` 또는 `sendChatAction` 표시 → IPv6 DNS 확인. `api.telegram.org`가 IPv6로 우선 해석되고 호스트에 IPv6 아웃바운드 연결이 없으면 IPv4를 강제하거나 IPv6를 활성화하세요. [/channels/telegram#troubleshooting](/channels/telegram#troubleshooting) 참조.
- 로그에 `setMyCommands failed` 표시 → `api.telegram.org`에 대한 아웃바운드 HTTPS 및 DNS 도달 가능성 확인(제한적인 VPS 또는 프록시 환경에서 흔함).
