---
read_when:
  - 기본 모델을 변경하거나 제공자 인증 상태를 보고 싶을 때
  - 사용 가능한 모델/제공자를 스캔하고 인증 프로필을 디버깅하고 싶을 때
summary: "`openclaw models`의 CLI 레퍼런스(status/list/set/scan, 별칭, 폴백, 인증)"
title: models
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# `openclaw models`

모델 검색, 스캔 및 구성(기본 모델, 폴백, 인증 프로필).

관련 내용:

- 제공자 + 모델: [모델](/providers/models)
- 제공자 인증 설정: [빠른 시작](/start/getting-started)

## 일반 명령

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```
