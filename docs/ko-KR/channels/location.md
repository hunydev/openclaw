---
read_when:
  - 채널 위치 해석 추가 또는 수정 시
  - 에이전트 프롬프트 또는 도구에서 위치 컨텍스트 필드 사용 시
summary: 인바운드 채널 위치 해석(Telegram + WhatsApp) 및 컨텍스트 필드
title: 채널 위치 해석
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 5602ef105c3da7e47497bfed8fc343dd8d7f3c019ff7e423a08b25092c5a1837
  source_path: channels/location.md
  workflow: 14
---

# 채널 위치 해석

OpenClaw는 채팅 채널에서 공유된 위치를 다음과 같이 정규화합니다:

- 인바운드 메시지 본문에 첨부된 사람이 읽을 수 있는 텍스트, 그리고
- 자동 응답 컨텍스트 페이로드의 구조화된 필드.

현재 지원:

- **Telegram**(위치 핀 + 장소 + 실시간 위치)
- **WhatsApp**(locationMessage + liveLocationMessage)
- **Matrix**(`m.location` + `geo_uri`)

## 텍스트 형식

위치는 괄호 없이 친숙한 행 형식으로 표시됩니다:

- 핀:
  - `📍 48.858844, 2.294351 ±12m`
- 명명된 장소:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- 실시간 공유:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

채널에 캡션/코멘트가 포함된 경우 다음 줄에 추가됩니다:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## 컨텍스트 필드

위치 정보가 있는 경우 다음 필드가 `ctx`에 추가됩니다:

- `LocationLat`(숫자)
- `LocationLon`(숫자)
- `LocationAccuracy`(숫자, 미터; 선택적)
- `LocationName`(문자열; 선택적)
- `LocationAddress`(문자열; 선택적)
- `LocationSource`(`pin | place | live`)
- `LocationIsLive`(부울)

## 채널 참고 사항

- **Telegram**: 장소는 `LocationName/LocationAddress`에 매핑됨; 실시간 위치는 `live_period` 사용.
- **WhatsApp**: `locationMessage.comment`와 `liveLocationMessage.caption`이 캡션 줄로 추가됨.
- **Matrix**: `geo_uri`가 핀 위치로 해석됨; 고도 무시, `LocationIsLive`는 항상 false.
