---
read_when:
  - 대화형으로 자격 증명, 장치 또는 에이전트 기본값을 조정하고 싶을 때
summary: "`openclaw configure`의 CLI 레퍼런스(대화형 구성 프롬프트)"
title: configure
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw configure`

자격 증명, 장치 및 에이전트 기본 구성을 설정하기 위한 대화형 프롬프트.

참고: **모델** 섹션에는 이제 `agents.defaults.models` 허용 목록을 설정하기 위한
다중 선택이 포함됩니다(모델 선택기 및 `/model`에서 표시할 모델 결정).

팁: 하위 명령 없이 `openclaw config`를 실행하면 동일한 마법사가 열립니다. 비대화형 편집에는
`openclaw config get|set|unset`을 사용하세요.

관련 내용:

- Gateway 구성 레퍼런스: [구성](/gateway/configuration)
- Config CLI: [Config](/cli/config)

참고 사항:

- Gateway가 실행될 위치를 선택할 때 `gateway.mode`가 항상 업데이트됩니다. 이것만 수정하면 되는 경우 다른 섹션을 구성하지 않고 "계속"을 선택할 수 있습니다.
- 채널 지향 서비스(Slack/Discord/Matrix/Microsoft Teams)는 설정 중 채널/룸 허용 목록을 구성하라는 프롬프트를 표시합니다. 이름 또는 ID를 입력할 수 있습니다; 마법사는 가능한 경우 이름을 ID로 해석합니다.

## 예시

```bash
openclaw configure
openclaw configure --section models --section channels
```
