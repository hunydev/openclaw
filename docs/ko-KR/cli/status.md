---
read_when:
  - 채널 상태 및 최근 세션 수신자를 빠르게 진단하고 싶을 때
  - 디버깅용 붙여넣기 가능한 전체 상태를 원할 때
summary: "`openclaw status`(진단, 탐색, 사용량 스냅샷)의 CLI 레퍼런스"
title: status
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw status`

채널 및 세션의 진단 정보.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

참고:

- `--deep`은 실시간 탐색을 실행합니다(WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
- 여러 에이전트가 구성된 경우 출력에 각 에이전트의 세션 저장소가 포함됩니다.
- 개요에는 Gateway 및 노드 호스트 서비스의 설치/실행 상태가 포함됩니다(사용 가능한 경우).
- 개요에는 업데이트 채널 및 git SHA가 포함됩니다(소스 체크아웃의 경우).
- 업데이트 정보가 개요에 표시됩니다; 업데이트가 사용 가능하면 상태에서 `openclaw update` 실행을 제안합니다([업데이트](/install/updating) 참조).
