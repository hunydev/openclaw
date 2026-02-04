---
read_when:
  - 실행 중인 Gateway의 상태를 빠르게 확인하고 싶을 때
summary: "`openclaw health`의 CLI 레퍼런스(RPC를 통한 Gateway 상태 엔드포인트)"
title: health
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw health`

실행 중인 Gateway에서 상태를 가져옵니다.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

참고:

- `--verbose`는 실시간 탐색을 실행하고 여러 계정이 구성된 경우 각 계정의 소요 시간을 출력합니다.
- 여러 에이전트가 구성된 경우 출력에 각 에이전트의 세션 저장소 정보가 포함됩니다.
