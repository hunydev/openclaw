---
read_when:
  - 스크립트에서 에이전트 턴 한 번을 실행하고 싶을 때(선택적으로 응답 배달)
summary: "`openclaw agent`의 CLI 레퍼런스(Gateway를 통해 에이전트 턴 한 번 전송)"
title: agent
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw agent`

Gateway를 통해 에이전트 턴 한 번 실행(임베디드 실행에는 `--local` 사용).
`--agent <id>`를 사용하여 구성된 에이전트를 직접 지정합니다.

관련 내용:

- 에이전트 전송 도구: [에이전트 전송](/tools/agent-send)

## 예시

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
