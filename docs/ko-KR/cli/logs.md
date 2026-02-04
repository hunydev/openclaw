---
read_when:
  - 원격으로 Gateway 로그를 봐야 할 때(SSH 불필요)
  - 도구용 JSON 형식의 로그 라인이 필요할 때
summary: RPC를 통해 Gateway 로그를 보는 `openclaw logs` CLI 레퍼런스
title: logs
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw logs`

RPC를 통해 Gateway 파일 로그를 실시간으로 봅니다(원격 모드 지원).

관련 내용:

- 로그 개요: [로깅](/logging)

## 예시

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
