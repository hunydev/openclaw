---
read_when:
  - 모델이나 사용자에게 타임스탬프가 표시되는 방식을 변경하려는 경우
  - 메시지나 시스템 프롬프트 출력의 시간 형식 문제를 디버깅하려는 경우
summary: 봉투, 프롬프트, 도구 및 커넥터에서의 날짜와 시간 처리
title: 날짜와 시간
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 날짜와 시간

OpenClaw는 기본적으로 **호스트 로컬 시간을 전송 타임스탬프**로 사용하고, **시스템 프롬프트에서만 사용자 시간대**를 사용합니다.
제공자 타임스탬프는 보존되므로 도구는 원래 의미를 유지합니다(현재 시간은 `session_status`를 통해 사용 가능).

## 메시지 봉투(기본값: 로컬 시간)

인바운드 메시지에는 타임스탬프(분 단위 정밀도)가 추가됩니다:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

이 봉투 타임스탬프는 제공자 시간대와 관계없이 **기본적으로 호스트 로컬 시간**입니다.

이 동작을 재정의할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA 시간대
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

- `envelopeTimezone: "utc"`는 UTC를 사용합니다.
- `envelopeTimezone: "local"`은 호스트 시간대를 사용합니다.
- `envelopeTimezone: "user"`는 `agents.defaults.userTimezone`을 사용합니다(호스트 시간대로 폴백).
- 명시적 IANA 시간대(예: `"America/Chicago"`)로 고정 시간대를 지정합니다.
- `envelopeTimestamp: "off"`는 봉투 헤더에서 절대 타임스탬프를 제거합니다.
- `envelopeElapsed: "off"`는 경과 시간 접미사(`+2m` 스타일)를 제거합니다.

### 예시

**로컬 시간(기본값):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**사용자 시간대:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**경과 시간 활성화:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

## 시스템 프롬프트: 현재 날짜와 시간

사용자 시간대를 알고 있으면 시스템 프롬프트에 **현재 날짜와 시간** 섹션이 포함되며, 프롬프트 캐싱 안정성을 위해 **시간대만**(시계/시간 형식 없이) 포함됩니다:

```
Time zone: America/Chicago
```

에이전트가 현재 시간을 얻어야 할 때는 `session_status` 도구를 사용하세요. 상태 카드에 타임스탬프 줄이 포함되어 있습니다.

## 시스템 이벤트 줄(기본값: 로컬 시간)

에이전트 컨텍스트에 삽입되는 대기 중인 시스템 이벤트는 메시지 봉투와 동일한 시간대 선택(기본값: 호스트 로컬 시간)을 사용하는 타임스탬프 접두사가 붙습니다.

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

### 사용자 시간대 및 형식 구성

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

- `userTimezone`은 프롬프트 컨텍스트에서 **사용자 로컬 시간대**를 설정합니다.
- `timeFormat`은 프롬프트에서 **12시간/24시간 표시 형식**을 제어합니다. `auto`는 OS 설정을 따릅니다.

## 시간 형식 감지(auto)

`timeFormat: "auto"`일 때 OpenClaw는 OS 설정(macOS/Windows)을 확인하고 지역 형식으로 폴백합니다. 감지된 값은 반복적인 시스템 호출을 피하기 위해 **프로세스별로 캐시**됩니다.

## 도구 페이로드 + 커넥터(원시 제공자 시간 + 정규화된 필드)

채널 도구는 **제공자 원시 타임스탬프**를 반환하며 일관성을 위해 정규화된 필드가 추가됩니다:

- `timestampMs`: 에포크 밀리초(UTC)
- `timestampUtc`: ISO 8601 UTC 문자열

원시 제공자 필드는 보존되며 데이터 손실이 없습니다.

- Slack: API의 에포크 유사 문자열
- Discord: UTC ISO 타임스탬프
- Telegram/WhatsApp: 제공자별 숫자/ISO 타임스탬프

로컬 시간이 필요하면 알려진 시간대를 사용하여 다운스트림에서 변환하세요.

## 관련 문서

- [시스템 프롬프트](/concepts/system-prompt)
- [시간대](/concepts/timezone)
- [메시지](/concepts/messages)
