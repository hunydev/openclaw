---
read_when:
  - cron 작업 생성 없이 시스템 이벤트를 큐에 넣고 싶을 때
  - 하트비트를 활성화하거나 비활성화해야 할 때
  - 시스템 프레즌스 항목을 확인하고 싶을 때
summary: "`openclaw system`의 CLI 레퍼런스(시스템 이벤트, 하트비트, 프레즌스)"
title: system
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw system`

Gateway의 시스템 수준 도우미: 시스템 이벤트 큐, 하트비트 제어 및 프레즌스 보기.

## 일반 명령

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

**메인** 세션에서 시스템 이벤트를 큐에 넣습니다. 다음 하트비트에서 프롬프트에 `System:` 줄로 주입됩니다. `--mode now`를 사용하면 즉시 하트비트를 트리거하고; `next-heartbeat`는 다음 예약된 하트비트 주기를 기다립니다.

플래그:

- `--text <text>`: 필수 시스템 이벤트 텍스트.
- `--mode <mode>`: `now` 또는 `next-heartbeat`(기본값).
- `--json`: 기계 판독 가능한 출력.
