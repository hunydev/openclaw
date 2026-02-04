---
read_when:
  - WhatsApp/웹 채널 동작 또는 수신함 라우팅 처리 시
summary: WhatsApp(웹 채널) 통합: 로그인, 수신함, 응답, 미디어 및 운영
title: WhatsApp
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# WhatsApp(웹 채널)

상태: Baileys를 통한 WhatsApp Web만 지원됩니다. Gateway가 세션을 소유합니다.

## 빠른 설정(시작하기)

1. 가능하면 **별도의 전화번호** 사용(권장).
2. `~/.openclaw/openclaw.json`에서 WhatsApp 구성.
3. `openclaw channels login`을 실행하여 QR 코드 스캔(연결된 기기).
4. Gateway 시작.

최소 구성:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## 목표

- 단일 Gateway 프로세스에서 여러 WhatsApp 계정 지원(다중 계정).
- 결정론적 라우팅: 응답은 WhatsApp으로 돌아가며, 모델 라우팅 없음.
- 모델이 인용 응답을 이해하는 데 충분한 컨텍스트 제공.

## 구성 쓰기

기본적으로 WhatsApp은 `/config set|unset`으로 트리거되는 구성 업데이트 쓰기를 허용합니다(`commands.config: true` 필요).

비활성화 방법:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## 아키텍처(책임 분담)

- **Gateway**가 Baileys 소켓과 수신함 루프를 소유합니다.
- **CLI / macOS 앱**은 Gateway와 통신하며, Baileys를 직접 사용하지 않습니다.
- **활성 리스너**는 아웃바운드 전송에 필요합니다. 그렇지 않으면 전송이 빠르게 실패합니다.

## 전화번호 얻기(두 가지 모드)

WhatsApp은 인증을 위해 실제 전화번호가 필요합니다. VoIP 및 가상 번호는 일반적으로 차단됩니다. WhatsApp에서 OpenClaw를 실행하는 두 가지 지원 방법:

### 전용 번호(권장)

OpenClaw용 **별도의 전화번호** 사용. 최고의 UX, 깔끔한 라우팅, 셀프채팅 문제 없음. 이상적인 설정: **여분/오래된 Android 폰 + eSIM**. Wi-Fi와 전원에 연결된 상태로 유지하고 QR 코드로 연결.

**WhatsApp Business:** 같은 기기에서 다른 번호로 WhatsApp Business를 사용할 수 있습니다. 개인 WhatsApp을 분리하기에 좋습니다 — WhatsApp Business를 설치하고 거기에 OpenClaw 번호를 등록하세요.

**예제 구성(전용 번호, 단일 사용자 허용 목록):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**페어링 모드(선택):**
허용 목록 대신 페어링을 사용하려면 `channels.whatsapp.dmPolicy`를 `pairing`으로 설정하세요. 알 수 없는 발신자는 페어링 코드를 받습니다. 다음으로 승인:
`openclaw pairing approve whatsapp <code>`

### 개인 번호(대안)

빠른 대안: **자신의 번호**에서 OpenClaw 실행. 자신에게 메시지 보내기(WhatsApp "나에게 메시지")로 연락처를 방해하지 않고 테스트. 설정 및 실험 중에는 기본 폰에서 인증 코드를 읽어야 합니다. **셀프채팅 모드를 활성화해야 합니다.**
마법사가 개인 WhatsApp 번호를 물을 때, 어시스턴트 번호가 아닌 메시지를 보내는 데 사용할 전화번호(소유자/발신자)를 입력하세요.

**예제 구성(개인 번호, 셀프채팅):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

`messages.responsePrefix`가 설정되면 셀프채팅 응답은 기본적으로 `[{identity.name}]`(그렇지 않으면 `[openclaw]`)을 사용합니다.
`messages.responsePrefix`가 설정되지 않으면 기본값이 사용됩니다. 명시적으로 설정하여 사용자 정의하거나 비활성화(`""`로 제거).

### 번호 얻기 팁

- **본국 eSIM**, 거주 국가의 이동통신사에서(가장 신뢰할 수 있음)
  - 오스트리아: [hot.at](https://www.hot.at)
  - 영국: [giffgaff](https://www.giffgaff.com) — 무료 SIM, 계약 없음
- **선불 SIM 카드** — 저렴함, 인증 SMS 한 번만 받으면 됨

**피해야 할 것:** TextNow, Google Voice, 대부분의 "무료 SMS" 서비스 — WhatsApp이 적극적으로 차단합니다.

**팁:** 번호는 인증 SMS 한 번만 받으면 됩니다. 그 후 WhatsApp Web 세션은 `creds.json`으로 유지됩니다.

## Twilio를 사용하지 않는 이유?

- 초기 OpenClaw 버전은 Twilio의 WhatsApp Business 통합을 지원했습니다.
- WhatsApp Business 번호는 개인 어시스턴트에 적합하지 않습니다.
- Meta는 24시간 응답 창을 강제합니다. 지난 24시간 이내에 응답하지 않으면 비즈니스 번호는 새 메시지를 시작할 수 없습니다.
- 고빈도 또는 "빈번한" 사용은 공격적인 차단을 트리거합니다. 비즈니스 계정은 개인 어시스턴트 스타일의 많은 메시지에 적합하지 않기 때문입니다.
- 결과: 전달이 불안정하고 자주 차단되어 지원이 제거되었습니다.

## 로그인 + 자격 증명

- 로그인 명령: `openclaw channels login`(연결된 기기를 통해 QR 코드 스캔).
- 다중 계정 로그인: `openclaw channels login --account <id>`(`<id>` = `accountId`).
- 기본 계정(`--account` 생략 시): 존재하면 `default`, 그렇지 않으면 첫 번째 구성된 계정 ID(정렬됨).
- 자격 증명은 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`에 저장됩니다.
- 백업 복사본은 `creds.json.bak`(손상 시 복원)에 있습니다.
- 레거시 호환성: 초기 설치는 Baileys 파일을 `~/.openclaw/credentials/`에 직접 저장했습니다.
- 로그아웃: `openclaw channels logout`(또는 `--account <id>`)은 WhatsApp 인증 상태를 삭제합니다(공유 `oauth.json`은 유지).
- 로그아웃된 소켓 => 재연결 오류 프롬프트.

## 인바운드 흐름(DM + 그룹)

- WhatsApp 이벤트는 `messages.upsert`(Baileys)에서 옵니다.
- 수신함 리스너는 테스트/재시작에서 이벤트 핸들러 누적을 방지하기 위해 종료 시 바인딩 해제됩니다.
- 상태/브로드캐스트 채팅은 무시됩니다.
- DM은 E.164 형식을 사용하고, 그룹은 그룹 JID를 사용합니다.
- **DM 정책**: `channels.whatsapp.dmPolicy`가 DM 접근을 제어합니다(기본값: `pairing`).
  - 페어링: 알 수 없는 발신자는 페어링 코드를 받습니다(`openclaw pairing approve whatsapp <code>`로 승인. 코드는 1시간 후 만료).
  - 열기: `channels.whatsapp.allowFrom`에 `"*"`가 필요합니다.
  - 연결된 WhatsApp 번호는 암시적으로 신뢰되므로 자기 메시지는 `channels.whatsapp.dmPolicy` 및 `channels.whatsapp.allowFrom` 검사를 건너뜁니다.

### 개인 번호 모드(대안)

**개인 WhatsApp 번호**에서 OpenClaw를 실행하는 경우 `channels.whatsapp.selfChatMode`를 활성화합니다(위의 예제 구성 참조).

동작:

- 아웃바운드 DM 메시지는 페어링 응답을 트리거하지 않습니다(연락처 스팸 방지).
- 알 수 없는 발신자의 인바운드 메시지는 여전히 `channels.whatsapp.dmPolicy`를 따릅니다.
- 셀프채팅 모드(allowFrom에 자신의 번호 포함)는 자동 읽음 확인을 피하고 멘션 JID를 무시합니다.
- 비셀프채팅 DM은 읽음 확인을 보냅니다.

## 읽음 확인

기본적으로 Gateway는 인바운드 WhatsApp 메시지를 수락하면 읽음으로 표시합니다(파란색 체크).

전역 비활성화:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

계정별 비활성화:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

참고:

- 셀프채팅 모드는 항상 읽음 확인을 건너뜁니다.

## WhatsApp FAQ: 메시지 전송 + 페어링

**WhatsApp을 연결하면 OpenClaw가 무작위 연락처에게 메시지를 보내나요?**
아니요. 기본 DM 정책은 **페어링**이므로 알 수 없는 발신자는 페어링 코드만 받고 메시지는 **처리되지 않습니다**. OpenClaw는 받은 채팅이나 명시적으로 트리거한 전송(에이전트/CLI)에만 응답합니다.

**WhatsApp에서 페어링은 어떻게 작동하나요?**
페어링은 알 수 없는 발신자에 대한 DM 게이트입니다:

- 새 발신자의 첫 DM 메시지는 단축 코드를 반환합니다(메시지는 처리되지 않음).
- 다음으로 승인: `openclaw pairing approve whatsapp <code>`(`openclaw pairing list whatsapp`로 나열).
- 코드는 1시간 후 만료됩니다. 채널당 최대 3개의 대기 중인 요청이 있습니다.

**여러 사람이 동일한 WhatsApp 번호에서 다른 OpenClaw 인스턴스를 사용할 수 있나요?**
예, `bindings`를 사용하여 각 발신자를 다른 에이전트로 라우팅합니다(peer `kind: "dm"`, 발신자 E.164 예: `+15551234567`). 응답은 여전히 **동일한 WhatsApp 계정**에서 오며, DM은 각 에이전트의 기본 세션으로 축소되므로 **사람당 하나의 에이전트**를 사용하세요. DM 접근 제어(`dmPolicy`/`allowFrom`)는 각 WhatsApp 계정 수준에서 전역입니다. [다중 에이전트 라우팅](/concepts/multi-agent) 참조.

**마법사가 왜 내 전화번호를 묻나요?**
마법사는 이를 사용하여 **허용 목록/소유자**를 설정하여 자신의 DM 메시지를 허용합니다. 자동 전송에 사용되지 않습니다. 개인 WhatsApp 번호에서 실행하는 경우 동일한 번호를 사용하고 `channels.whatsapp.selfChatMode`를 활성화합니다.

## 메시지 정규화(모델이 보는 것)

- `Body`는 현재 메시지 본문과 그 봉투입니다.
- 인용 응답 컨텍스트는 **항상 첨부**됩니다:
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
- 응답 메타데이터도 설정됩니다:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = 인용 본문 또는 미디어 자리 표시자
  - `ReplyToSender` = E.164(알려진 경우)
- 미디어만 있는 인바운드 메시지는 자리 표시자를 사용합니다:
  - `<media:image|video|audio|document|sticker>`

## 그룹 채팅

- 그룹은 `agent:<agentId>:whatsapp:group:<jid>` 세션에 매핑됩니다.
- 그룹 정책: `channels.whatsapp.groupPolicy = open|disabled|allowlist`(기본값 `allowlist`).
- 활성화 모드:
  - `mention`(기본값): @멘션 또는 정규식 일치 필요.
  - `always`: 항상 트리거.
- `/activation mention|always`는 소유자 전용이며 독립적인 메시지로 전송해야 합니다.
- 소유자 = `channels.whatsapp.allowFrom`(설정되지 않은 경우 자체 E.164).
- **기록 주입**(대기 중만):
  - 최근 *처리되지 않은* 메시지(기본값 50개)가 다음에 삽입됩니다:
    `[Chat messages since your last reply - for context]`(이미 세션에 있는 메시지는 다시 주입되지 않음)
  - 현재 메시지는:
    `[Current message - respond to this]`
  - 발신자 접미사가 추가됩니다: `[from: Name (+E164)]`
- 그룹 메타데이터는 5분간 캐시됩니다(주제 + 참여자).

## 응답 전달(스레드)

- WhatsApp Web은 표준 메시지를 보냅니다(현재 Gateway에 인용 응답 스레딩 없음).
- 이 채널에서는 응답 태그가 무시됩니다.

## 확인 반응(메시지 수신 시 자동 반응)

WhatsApp은 봇이 응답을 생성하기 전에 메시지를 수신하는 즉시 자동으로 이모지 반응을 보낼 수 있습니다. 이는 사용자에게 메시지가 수신되었음을 즉시 피드백합니다.

**구성:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**옵션:**

- `emoji`(문자열): 확인용 이모지(예: "👀", "✅", "📨"). 비어 있거나 생략 = 기능 비활성화.
- `direct`(불리언, 기본값: `true`): DM에서 반응 전송.
- `group`(문자열, 기본값: `"mentions"`): 그룹 채팅 동작:
  - `"always"`: 모든 그룹 메시지에 반응(@멘션 없어도)
  - `"mentions"`: 봇이 @멘션될 때만 반응
  - `"never"`: 그룹에서 절대 반응하지 않음

**계정별 재정의:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**동작 참고:**

- 반응은 메시지 수신 시 **즉시** 전송되며, 타이핑 표시기나 봇 응답 전입니다.
- `requireMention: false`(활성화 모드: always)인 그룹에서 `group: "mentions"`는 모든 메시지에 반응합니다(@멘션만이 아님).
- 발사 후 망각: 반응 실패는 기록되지만 봇 응답을 차단하지 않습니다.
- 그룹 반응에는 참여자 JID가 자동으로 포함됩니다.
- WhatsApp은 `messages.ackReaction`을 무시합니다. 대신 `channels.whatsapp.ackReaction`을 사용하세요.

## 에이전트 도구(반응)

- 도구: `whatsapp`, `react` 작업(`chatJid`, `messageId`, `emoji`, 선택적 `remove`).
- 선택: `participant`(그룹 발신자), `fromMe`(자신의 메시지에 반응), `accountId`(다중 계정).
- 반응 제거 의미: [/tools/reactions](/tools/reactions) 참조.
- 도구 게이팅: `channels.whatsapp.actions.reactions`(기본값: 활성화).

## 제한

- 아웃바운드 텍스트는 `channels.whatsapp.textChunkLimit`로 청킹됩니다(기본값 4000).
- 선택적 줄바꿈 청킹: `channels.whatsapp.chunkMode="newline"`을 설정하면 빈 줄(단락 경계)에서 분할한 후 길이별 청킹.
- 인바운드 미디어 저장은 `channels.whatsapp.mediaMaxMb`로 제한됩니다(기본값 50 MB).
- 아웃바운드 미디어 항목은 `agents.defaults.mediaMaxMb`로 제한됩니다(기본값 5 MB).

## 아웃바운드 전송(텍스트 + 미디어)

- 활성 웹 리스너 사용. Gateway가 실행 중이 아니면 오류.
- 텍스트 청킹: 메시지당 최대 4k(`channels.whatsapp.textChunkLimit`로 구성 가능, 선택적 `channels.whatsapp.chunkMode`).
- 미디어:
  - 이미지/비디오/오디오/문서 지원.
  - 오디오는 PTT로 전송됩니다. `audio/ogg` => `audio/ogg; codecs=opus`.
  - 첫 번째 미디어 항목만 캡션이 달립니다.
  - 미디어 가져오기는 HTTP(S) 및 로컬 경로 지원.
  - 동적 GIF: WhatsApp은 인라인 루프 재생을 위해 `gifPlayback: true`인 MP4를 기대합니다.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send` 파라미터에 `gifPlayback: true` 포함

## 음성 메시지(PTT 오디오)

WhatsApp은 오디오를 **음성 메시지**(PTT 버블)로 보냅니다.

- 최적: OGG/Opus. OpenClaw가 `audio/ogg`를 `audio/ogg; codecs=opus`로 다시 작성합니다.
- WhatsApp은 `[[audio_as_voice]]`를 무시합니다(오디오는 이미 음성 메시지로 전송됨).

## 미디어 제한 + 최적화

- 기본 아웃바운드 제한: 5 MB(미디어 항목당).
- 재정의: `agents.defaults.mediaMaxMb`.
- 이미지는 제한 내에 맞도록 자동으로 JPEG로 최적화됩니다(스케일링 + 품질 스캔).
- 초과 미디어 => 오류. 미디어 응답은 텍스트 경고로 폴백.

## 하트비트

- **Gateway 하트비트**는 연결 건강 상태를 기록합니다(`web.heartbeatSeconds`, 기본값 60초).
- **에이전트 하트비트**는 에이전트별로 구성하거나(`agents.list[].heartbeat`)
  `agents.defaults.heartbeat`를 통해 전역으로 구성할 수 있습니다(에이전트별 항목이 설정되지 않은 경우 폴백).
  - 구성된 하트비트 프롬프트 사용(기본값: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + `HEARTBEAT_OK` 건너뛰기 동작.
  - 전달은 기본적으로 마지막 사용된 채널(또는 구성된 대상)로.

## 재연결 동작

- 백오프: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- maxAttempts에 도달하면 웹 모니터가 중지됩니다(강등).
- 로그아웃됨 => 중지하고 재연결 요청.

## 구성 요약

- `channels.whatsapp.dmPolicy`(DM 정책: pairing/allowlist/open/disabled).
- `channels.whatsapp.selfChatMode`(같은 번호 설정. 봇이 개인 WhatsApp 번호 사용).
- `channels.whatsapp.allowFrom`(DM 허용 목록). WhatsApp은 E.164 전화번호 사용(사용자 이름 없음).
- `channels.whatsapp.mediaMaxMb`(인바운드 미디어 저장 제한).
- `channels.whatsapp.ackReaction`(메시지 수신 시 자동 반응: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*`(계정별 설정 + 선택적 `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb`(계정별 인바운드 미디어 제한).
- `channels.whatsapp.accounts.<accountId>.ackReaction`(계정별 확인 반응 재정의).
- `channels.whatsapp.groupAllowFrom`(그룹 발신자 허용 목록).
- `channels.whatsapp.groupPolicy`(그룹 정책).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit`(그룹 기록 컨텍스트. `0`이면 비활성화).
- `channels.whatsapp.dmHistoryLimit`(DM 기록 제한, 사용자 턴 수). 사용자별 재정의: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups`(그룹 허용 목록 + 멘션 게이트 기본값. `"*"`로 모두 허용)
- `channels.whatsapp.actions.reactions`(WhatsApp 도구 반응 게이트).
- `agents.list[].groupChat.mentionPatterns`(또는 `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix`(인바운드 접두사. 계정별: `channels.whatsapp.accounts.<accountId>.messagePrefix`. 더 이상 사용되지 않음: `messages.messagePrefix`)
- `messages.responsePrefix`(아웃바운드 접두사)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model`(선택적 재정의)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*`(에이전트별 재정의)
- `session.*`(scope, idle, store, mainKey)
- `web.enabled`(false이면 채널 시작 비활성화)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## 로그 + 문제 해결

- 서브시스템: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- 로그 파일: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`(구성 가능).
- 문제 해결 가이드: [Gateway 문제 해결](/gateway/troubleshooting).

## 문제 해결(빠른)

**연결되지 않음 / QR 로그인 필요**

- 증상: `channels status`가 `linked: false`를 표시하거나 "연결되지 않음" 경고.
- 해결: Gateway 호스트에서 `openclaw channels login`을 실행하고 QR 코드 스캔(WhatsApp → 설정 → 연결된 기기).

**연결됨 but 연결 끊김 / 재연결 루프**

- 증상: `channels status`가 `running, disconnected`를 표시하거나 "연결됨 but 연결 끊김" 경고.
- 해결: `openclaw doctor`(또는 Gateway 재시작). 문제가 지속되면 `channels login`으로 재연결하고 `openclaw logs --follow`를 확인.

**Bun 런타임**

- **Bun은 권장하지 않습니다**. WhatsApp(Baileys)과 Telegram은 Bun에서 불안정합니다.
  Gateway 실행에는 **Node**를 사용하세요.(시작 가이드 런타임 참고 참조.)
