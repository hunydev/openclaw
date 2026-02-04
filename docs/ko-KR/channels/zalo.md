---
read_when:
  - Zalo 기능 또는 webhook 작업 시
summary: Zalo 봇 지원 상태, 기능 및 설정
title: Zalo
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 0311d932349f96412b712970b5d37329b91929bf3020536edf3ca0ff464373c0
  source_path: channels/zalo.md
  workflow: 14
---

# Zalo(Bot API)

상태: 실험적. DM만 지원; Zalo 문서에 따르면 그룹 기능은 곧 출시 예정.

## 플러그인 필요

Zalo는 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

- CLI를 통해 설치: `openclaw plugins install @openclaw/zalo`
- 또는 온보딩에서 **Zalo**를 선택하고 설치 프롬프트 확인
- 자세한 내용: [플러그인](/plugin)

## 빠른 설정(초보자용)

1. Zalo 플러그인 설치:
   - 소스 체크아웃에서: `openclaw plugins install ./extensions/zalo`
   - npm에서(게시된 경우): `openclaw plugins install @openclaw/zalo`
   - 또는 온보딩에서 **Zalo**를 선택하고 설치 프롬프트 확인
2. 토큰 설정:
   - 환경 변수: `ZALO_BOT_TOKEN=...`
   - 또는 설정: `channels.zalo.botToken: "..."`.
3. Gateway 재시작(또는 온보딩 완료).
4. DM 접근은 기본적으로 페어링 모드; 첫 연락 시 페어링 코드 승인.

최소 설정:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## 소개

Zalo는 베트남 시장을 대상으로 한 인스턴트 메시징 앱입니다. Bot API를 통해 Gateway가 1:1 대화용 봇을 실행할 수 있습니다.
고객 서비스나 메시지를 Zalo로 확정적으로 라우팅해야 하는 알림 시나리오에 적합합니다.

- Gateway가 관리하는 Zalo Bot API 채널.
- 확정적 라우팅: 응답은 항상 Zalo로 돌아감; 모델이 채널을 선택하지 않음.
- DM은 에이전트의 메인 세션을 공유.
- 그룹은 아직 지원되지 않음(Zalo 문서에 "곧 출시"로 표시됨).

## 설정(빠른 경로)

### 1) 봇 토큰 생성(Zalo Bot Platform)

1. **https://bot.zaloplatforms.com**으로 이동하여 로그인.
2. 새 봇을 만들고 설정 구성.
3. 봇 토큰 복사(형식: `12345689:abc-xyz`).

### 2) 토큰 설정(환경 변수 또는 설정)

예시:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

환경 변수 옵션: `ZALO_BOT_TOKEN=...`(기본 계정만).

다중 계정: `channels.zalo.accounts`를 사용하여 각 계정에 토큰과 선택적 `name` 설정.

3. Gateway 재시작. 토큰이 해석되면(환경 변수 또는 설정을 통해) Zalo가 시작됩니다.
4. DM 접근은 기본적으로 페어링 모드. 봇이 처음 연락받으면 페어링 코드를 승인하세요.

## 작동 방식(동작)

- 인바운드 메시지는 미디어 플레이스홀더와 함께 공유 채널 엔벨로프로 정규화됨.
- 응답은 항상 동일한 Zalo 채팅으로 라우팅됨.
- 기본적으로 롱 폴링 사용; `channels.zalo.webhookUrl`로 webhook 모드 활성화 가능.

## 제한

- 아웃바운드 텍스트는 2000자로 청크 분할(Zalo API 제한).
- 미디어 다운로드/업로드는 `channels.zalo.mediaMaxMb`로 제한됨(기본값 5).
- 2000자 제한으로 인해 스트리밍이 의미 없어 기본적으로 비활성화됨.

## 접근 제어(DM)

### DM 접근

- 기본값: `channels.zalo.dmPolicy = "pairing"`. 알 수 없는 발신자는 페어링 코드를 받음; 승인 전까지 메시지 무시(페어링 코드 1시간 후 만료).
- 승인 방법:
  - `openclaw pairing list zalo`
  - `openclaw pairing approve zalo <CODE>`
- 페어링이 기본 토큰 교환. 자세한 내용: [페어링](/start/pairing)
- `channels.zalo.allowFrom`은 숫자 사용자 ID를 허용(사용자 이름 조회 미지원).

## 롱 폴링 vs webhook

- 기본값: 롱 폴링(공용 URL 불필요).
- Webhook 모드: `channels.zalo.webhookUrl`과 `channels.zalo.webhookSecret` 설정.
  - Webhook 시크릿은 8-256자여야 함.
  - Webhook URL은 HTTPS여야 함.
  - Zalo는 검증을 위해 `X-Bot-Api-Secret-Token` 헤더와 함께 이벤트 전송.
  - Gateway HTTP는 `channels.zalo.webhookPath`에서 webhook 요청 처리(기본값은 webhook URL 경로).

**참고:** Zalo API 문서에 따르면 getUpdates(폴링)와 webhook은 상호 배타적입니다.

## 지원되는 메시지 타입

- **텍스트 메시지**: 완전 지원, 2000자로 청크 분할.
- **이미지 메시지**: 인바운드 이미지 다운로드 및 처리; `sendPhoto`를 통해 이미지 전송.
- **스티커**: 로깅되지만 완전히 처리되지 않음(에이전트 응답 없음).
- **지원되지 않는 타입**: 로깅됨(예: 보호된 사용자의 메시지).

## 기능

| 기능          | 상태                            |
| ------------- | ------------------------------- |
| DM            | ✅ 지원됨                       |
| 그룹          | ❌ 곧 출시(Zalo 문서에 따름)    |
| 미디어(이미지)| ✅ 지원됨                       |
| 이모지 반응   | ❌ 지원 안 됨                   |
| 토픽          | ❌ 지원 안 됨                   |
| 투표          | ❌ 지원 안 됨                   |
| 네이티브 명령 | ❌ 지원 안 됨                   |
| 스트리밍      | ⚠️ 비활성화됨(2000자 제한)     |

## 전달 대상(CLI/크론)

- 대상으로 채팅 ID 사용.
