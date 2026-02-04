---
read_when:
  - 비대화형으로 구성을 읽거나 편집하고 싶을 때
summary: "`openclaw config`의 CLI 레퍼런스(구성 값 get/set/unset)"
title: config
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw config`

구성 도우미: 경로로 값을 get/set/unset합니다. 하위 명령 없이 실행하면 구성 마법사가 열립니다(`openclaw configure`와 동일).

## 예시

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```

## 경로

경로는 점 또는 대괄호 표기법을 사용합니다:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

에이전트 목록 인덱스를 사용하여 특정 에이전트를 지정합니다:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## 값

값은 가능하면 JSON5로 파싱됩니다; 그렇지 않으면 문자열로 처리됩니다.
JSON5 파싱을 강제하려면 `--json`을 사용하세요.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

편집 후 Gateway를 재시작하세요.
