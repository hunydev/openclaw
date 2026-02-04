---
read_when:
  - 여러 격리된 에이전트(작업 공간 + 라우팅 + 인증)가 필요할 때
summary: "`openclaw agents`의 CLI 레퍼런스(나열/추가/삭제/신원 설정)"
title: agents
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw agents`

격리된 에이전트(작업 공간 + 인증 + 라우팅) 관리.

관련 내용:

- 멀티 에이전트 라우팅: [멀티 에이전트 라우팅](/concepts/multi-agent)
- 에이전트 작업 공간: [에이전트 작업 공간](/concepts/agent-workspace)

## 예시

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## 신원 파일

각 에이전트 작업 공간은 작업 공간 루트에 `IDENTITY.md`를 포함할 수 있습니다:

- 예시 경로: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity`는 작업 공간 루트에서 읽습니다(또는 명시적으로 지정된 `--identity-file`에서)

아바타 경로는 작업 공간 루트를 기준으로 해석됩니다.

## 신원 설정

`set-identity`는 `agents.list[].identity`에 필드를 씁니다:

- `name`
- `theme`
- `emoji`
- `avatar`(작업 공간 상대 경로, http(s) URL 또는 data URI)
