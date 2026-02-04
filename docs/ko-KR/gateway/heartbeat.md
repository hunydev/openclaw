---
read_when:
  - 하트비트 빈도 또는 메시지 내용 조정 시
  - 하트비트와 cron 중 선택 시
summary: 하트비트 폴링 메시지 및 알림 규칙
title: 하트비트
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 18b017066aa2c41811b985564dd389834906f4576e85b576fb357a0eff482e69
  source_path: gateway/heartbeat.md
  workflow: 14
---

# 하트비트 (Gateway)

> **하트비트 또는 cron?** [Cron vs 하트비트](/automation/cron-vs-heartbeat)에서 언제 어떤 것을 사용해야 하는지 확인하세요.

하트비트는 메인 세션에서 **정기적인 에이전트 대화 턴**을 실행하여 모델이 사용자를 방해하지 않으면서 주의가 필요한 사항을 사전에 알릴 수 있게 합니다.

## 빠른 시작 (초보자)

1. 하트비트를 활성화된 상태로 유지 (기본 간격 `30m`, Anthropic OAuth/setup-token 모드에서 `1h`), 또는 사용자 정의 빈도 설정.
2. 에이전트 작업 공간에 간단한 `HEARTBEAT.md` 체크리스트 생성 (선택 사항이지만 권장).
3. 하트비트 메시지 전송 위치 결정 (기본값 `target: "last"`).
4. 선택 사항: 투명성을 위해 하트비트 추론 내용 전달 활성화.
5. 선택 사항: 하트비트를 활성 시간대 (현지 시간)로 제한.

구성 예제:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // 선택 사항: 별도의 `Reasoning:` 메시지도 전달
      },
    },
  },
}
```

## 기본값

- 간격: `30m` (Anthropic OAuth/setup-token 인증 모드 감지 시 `1h`). `agents.defaults.heartbeat.every` 또는 에이전트별 `agents.list[].heartbeat.every`로 설정; `0m`으로 비활성화.
- 프롬프트 본문 (`agents.defaults.heartbeat.prompt`로 구성 가능):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- 하트비트 프롬프트는 사용자 메시지로 **그대로** 전송됩니다. 시스템 프롬프트에 "Heartbeat" 섹션이 포함되며 실행이 내부적으로 하트비트로 표시됩니다.
- 활성 시간 (`heartbeat.activeHours`)은 구성된 시간대에 따라 검사됩니다. 창 밖에서는 다음 창 내 시점까지 하트비트가 건너뛰어집니다.

## 하트비트 프롬프트의 목적

기본 프롬프트는 의도적으로 넓게 설계되었습니다:

- **백그라운드 작업**: "Consider outstanding tasks"는 에이전트가 할 일 (받은 편지함, 캘린더, 알림, 대기 중인 작업)을 확인하고 긴급한 것을 표시하도록 합니다.
- **사용자 배려**: "Checkup sometimes on your human during day time"은 가끔 가벼운 "도움이 필요하세요?" 메시지를 보내도록 하지만, 구성된 현지 시간대를 사용하여 야간 방해를 방지합니다 ([/concepts/timezone](/concepts/timezone) 참조).

하트비트가 매우 구체적인 작업을 수행하길 원하면 (예: "Gmail PubSub 통계 확인" 또는 "Gateway 상태 검증"), `agents.defaults.heartbeat.prompt` (또는 `agents.list[].heartbeat.prompt`)를 사용자 정의 내용으로 설정하세요 (그대로 전송됨).

## 응답 규칙

- 주의가 필요한 것이 없으면 **`HEARTBEAT_OK`**로 응답.
- 하트비트 실행 중 `HEARTBEAT_OK`가 응답의 **시작 또는 끝**에 나타나면 OpenClaw가 이를 확인으로 처리합니다. 표시가 제거되고 남은 내용이 **≤ `ackMaxChars`** (기본값: 300)이면 전체 응답이 삭제됩니다.
- `HEARTBEAT_OK`가 응답의 **중간**에 나타나면 특별 처리 없음.
- 알림의 경우 `HEARTBEAT_OK`를 **포함하지 마세요**; 알림 텍스트만 반환하세요.

하트비트 외부에서 메시지 시작 또는 끝의 잘못된 `HEARTBEAT_OK`는 제거되고 로깅됩니다; `HEARTBEAT_OK`만 포함된 메시지는 삭제됩니다.

## 구성

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 기본값: 30m (0m으로 비활성화)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // 기본값: false (가용 시 별도 Reasoning: 메시지 전달)
        target: "last", // last | none | <채널 id> (코어 또는 플러그인, 예: "bluebubbles")
        to: "+15551234567", // 선택적 채널별 오버라이드
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // HEARTBEAT_OK 후 허용되는 최대 문자 수
      },
    },
  },
}
```

### 범위 및 우선순위

- `agents.defaults.heartbeat`는 전역 하트비트 동작을 설정합니다.
- `agents.list[].heartbeat`는 그 위에 병합됩니다; 어떤 에이전트에 `heartbeat` 블록이 있으면 **해당 에이전트만** 하트비트를 실행합니다.
- `channels.defaults.heartbeat`는 모든 채널에 대한 가시성 기본값을 설정합니다.
- `channels.<channel>.heartbeat`는 채널 기본값을 오버라이드합니다.
- `channels.<channel>.accounts.<id>.heartbeat` (멀티 계정 채널)는 채널별 설정을 오버라이드합니다.

### 에이전트별 하트비트

`agents.list[]` 항목 중 하나에 `heartbeat` 블록이 포함되면 **해당 에이전트만** 하트비트를 실행합니다. 에이전트별 블록은 `agents.defaults.heartbeat` 위에 병합됩니다 (공유 기본값을 한 번 설정한 후 에이전트별로 오버라이드 가능).

예제: 두 에이전트, 두 번째만 하트비트 실행.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### 필드 설명

- `every`: 하트비트 간격 (기간 문자열; 기본 단위 = 분).
- `model`: 하트비트 실행을 위한 선택적 모델 오버라이드 (`provider/model`).
- `includeReasoning`: 활성화 시 가용할 때 `Reasoning:` 접두사가 붙은 별도 메시지도 전달 (`/reasoning on` 형식과 동일).
- `session`: 하트비트 실행을 위한 선택적 세션 키.
  - `main` (기본값): 에이전트 메인 세션.
  - 명시적 세션 키 (`openclaw sessions --json` 또는 [세션 CLI](/cli/sessions)에서 복사).
  - 세션 키 형식: [세션](/concepts/session) 및 [그룹](/concepts/groups) 참조.
- `target`:
  - `last` (기본값): 가장 최근 사용된 외부 채널로 전달.
  - 명시적 채널: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  - `none`: 하트비트 실행하지만 외부로 전달 **안 함**.
- `to`: 선택적 수신자 오버라이드 (채널별 ID, 예: WhatsApp의 E.164 형식 또는 Telegram 채팅 ID).
- `prompt`: 기본 프롬프트 본문 오버라이드 (병합 안 함).
- `ackMaxChars`: 전달 전 `HEARTBEAT_OK` 후 허용되는 최대 문자 수.

## 전달 동작

- 하트비트는 기본적으로 에이전트의 메인 세션에서 실행 (`agent:<id>:<mainKey>`), 또는 `session.scope = "global"`일 때 `global`. `session` 설정으로 특정 채널 세션 (Discord/WhatsApp 등)으로 오버라이드.
- `session`은 실행 컨텍스트에만 영향; 전달은 `target`과 `to`로 제어.
- 특정 채널/수신자에게 전달하려면 `target` + `to` 설정. `target: "last"` 사용 시 해당 세션의 가장 최근 외부 채널 사용.
- 메인 큐가 바쁘면 하트비트가 건너뛰어지고 나중에 재시도됩니다.
- `target`이 외부 대상으로 해석되지 않으면 실행은 하지만 아웃바운드 메시지는 전송 안 함.
- 하트비트만 포함된 응답은 세션을 활성 상태로 **유지하지 않습니다**; `updatedAt`이 복원되어 유휴 만료 동작이 정상 작동합니다.

## 가시성 제어

기본적으로 `HEARTBEAT_OK` 확인은 숨겨지고 알림 내용은 전달됩니다. 채널별 또는 계정별로 조정할 수 있습니다:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # HEARTBEAT_OK 숨김 (기본값)
      showAlerts: true # 알림 메시지 표시 (기본값)
      useIndicator: true # 인디케이터 이벤트 전송 (기본값)
  telegram:
    heartbeat:
      showOk: true # Telegram에서 OK 확인 표시
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # 해당 계정의 알림 전달 억제
```

우선순위: 계정별 → 채널별 → 채널 기본값 → 내장 기본값.

### 각 플래그의 역할

- `showOk`: 모델이 OK만 포함된 응답을 반환할 때 `HEARTBEAT_OK` 확인 전송.
- `showAlerts`: 모델이 비-OK 응답을 반환할 때 알림 내용 전송.
- `useIndicator`: UI 상태 표시 패널용 인디케이터 이벤트 전송.

**세 가지 모두** false이면 OpenClaw가 하트비트 실행을 완전히 건너뜁니다 (모델 호출 안 함).

### 채널별 vs 계정별 예제

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 모든 Slack 계정
    accounts:
      ops:
        heartbeat:
          showAlerts: false # ops 계정만 알림 억제
  telegram:
    heartbeat:
      showOk: true
```

### 일반적인 패턴

| 목표                               | 구성                                                                                     |
| ---------------------------------- | ---------------------------------------------------------------------------------------- |
| 기본 동작 (무음 OK, 알림 켜짐)     | _(구성 필요 없음)_                                                                       |
| 완전 무음 (메시지 없음, 인디케이터 없음) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| 인디케이터만 (메시지 없음)         | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }`  |
| 한 채널에서만 OK 표시              | `channels.telegram.heartbeat: { showOk: true }`                                          |

## HEARTBEAT.md (선택 사항)

작업 공간에 `HEARTBEAT.md` 파일이 있으면 기본 프롬프트가 에이전트에게 이를 읽도록 지시합니다. 이것을 "하트비트 체크리스트"로 생각하세요: 짧고 안정적이며 30분마다 안전하게 포함될 수 있어야 합니다.

`HEARTBEAT.md`가 존재하지만 실제로 비어 있으면 (빈 줄과 `# Heading` 같은 마크다운 제목만 포함) OpenClaw는 API 호출을 절약하기 위해 하트비트 실행을 건너뜁니다. 파일이 없으면 하트비트는 여전히 실행되며 모델이 무엇을 할지 결정합니다.

프롬프트 비대화를 피하기 위해 내용을 짧게 유지하세요 (간단한 체크리스트 또는 알림).

`HEARTBEAT.md` 예제:

```md
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it's daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### 에이전트가 HEARTBEAT.md를 업데이트할 수 있나요?

네—그렇게 지시하면 됩니다.

`HEARTBEAT.md`는 에이전트 작업 공간의 일반 파일이므로 일반 채팅에서 에이전트에게 지시할 수 있습니다:

- "매일 캘린더 확인을 추가하도록 `HEARTBEAT.md`를 업데이트해줘."
- "받은 편지함 후속 조치에 집중하도록 `HEARTBEAT.md`를 더 짧게 다시 작성해줘."

이 업데이트가 사전에 발생하길 원하면 하트비트 프롬프트에 명시적 지시를 포함할 수 있습니다. 예: "체크리스트가 오래되면 더 나은 내용으로 HEARTBEAT.md를 업데이트해줘."

안전 팁: `HEARTBEAT.md`에 민감한 정보 (API 키, 전화번호, 비공개 토큰)를 넣지 마세요—프롬프트 컨텍스트의 일부가 됩니다.

## 수동 깨우기 (온디맨드)

시스템 이벤트를 큐에 넣고 하트비트를 즉시 트리거할 수 있습니다:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

여러 에이전트에 `heartbeat`가 구성되어 있으면 수동 깨우기가 각 에이전트의 하트비트를 즉시 실행합니다.

다음 예약된 시점까지 기다리려면 `--mode next-heartbeat` 사용.

## 추론 내용 전달 (선택 사항)

기본적으로 하트비트는 최종 "응답" 페이로드만 전달합니다.

투명성이 필요하면 활성화하세요:

- `agents.defaults.heartbeat.includeReasoning: true`

활성화되면 하트비트는 `Reasoning:` 접두사가 붙은 별도 메시지도 전달합니다 (`/reasoning on` 형식과 동일). 에이전트가 여러 세션/코드베이스를 관리하고 왜 연락하기로 결정했는지 이해하고 싶을 때 유용합니다—하지만 예상보다 더 많은 내부 세부 정보가 노출될 수 있습니다. 그룹 채팅에서는 비활성화 상태로 유지하는 것이 좋습니다.

## 비용 인식

하트비트는 전체 에이전트 대화 턴을 실행합니다. 더 짧은 간격은 더 많은 토큰을 소비합니다. `HEARTBEAT.md`를 짧게 유지하고 내부 상태 업데이트만 필요하면 더 저렴한 `model` 또는 `target: "none"` 사용을 고려하세요.
