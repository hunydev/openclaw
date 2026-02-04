---
read_when:
  - Matrix 채널 기능 개발 시
summary: Matrix 지원 상태, 기능 및 구성
title: Matrix
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Matrix(플러그인)

Matrix는 개방형 탈중앙화 메시징 프로토콜입니다. OpenClaw는 Matrix **사용자**로 모든 홈서버에 연결하므로, 봇용 Matrix 계정을 만들어야 합니다. 로그인하면 봇에게 직접 DM을 보내거나 룸(Matrix의 "그룹")에 초대할 수 있습니다. Beeper도 클라이언트 옵션이지만 종단간 암호화가 활성화되어야 합니다.

상태: 플러그인을 통해 지원됨(@vector-im/matrix-bot-sdk). DM, 룸, 스레드, 미디어, 반응, 투표(전송 + poll-start를 텍스트로 변환), 위치 및 종단간 암호화(암호화 지원 필요) 지원.

## 플러그인 필요

Matrix는 플러그인으로 제공되며 코어 설치에 포함되지 않습니다.

CLI를 통해 설치(npm 레지스트리):

```bash
openclaw plugins install @openclaw/matrix
```

로컬 체크아웃(git 저장소에서 실행 시):

```bash
openclaw plugins install ./extensions/matrix
```

구성/온보딩 중 Matrix를 선택하고 git 체크아웃이 감지되면 OpenClaw가 자동으로 로컬 설치 경로를 제공합니다.

자세한 내용: [플러그인](/plugin)

## 설정

1. Matrix 플러그인 설치:
   - npm에서: `openclaw plugins install @openclaw/matrix`
   - 로컬 체크아웃에서: `openclaw plugins install ./extensions/matrix`
2. 홈서버에 Matrix 계정 생성:
   - [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)에서 호스팅 옵션 탐색
   - 또는 셀프 호스팅.
3. 봇 계정의 접근 토큰 얻기:
   - 홈서버에서 Matrix 로그인 API를 `curl`로 사용:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   - `matrix.example.org`를 홈서버 URL로 대체.
   - 또는 `channels.matrix.userId` + `channels.matrix.password` 설정: OpenClaw가 동일한 로그인 엔드포인트를 호출하고, 접근 토큰을 `~/.openclaw/credentials/matrix/credentials.json`에 저장하며, 다음 시작 시 재사용합니다.

4. 자격 증명 구성:
   - 환경 변수: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`(또는 `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   - 또는 구성: `channels.matrix.*`
   - 둘 다 설정되면 구성이 우선.
   - 접근 토큰 사용 시: 사용자 ID는 `/whoami`를 통해 자동 획득.
   - 설정 시 `channels.matrix.userId`는 전체 Matrix ID여야 합니다(예: `@bot:example.org`).
5. Gateway 재시작(또는 온보딩 완료).
6. 모든 Matrix 클라이언트(Element, Beeper 등; https://matrix.org/ecosystem/clients/ 참조)에서 봇과 DM을 시작하거나 룸에 초대. Beeper는 종단간 암호화가 필요하므로 `channels.matrix.encryption: true`를 설정하고 기기를 인증하세요.

최소 구성(접근 토큰, 사용자 ID 자동 획득):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

종단간 암호화 구성(E2EE 활성화):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## 암호화(종단간 암호화)

종단간 암호화는 Rust 암호화 SDK를 통해 **지원**됩니다.

`channels.matrix.encryption: true`로 활성화:

- 암호화 모듈이 성공적으로 로드되면 암호화된 룸이 자동으로 복호화됩니다.
- 암호화된 룸으로 전송 시 아웃바운드 미디어가 암호화됩니다.
- 첫 연결 시 OpenClaw가 다른 세션에서 기기 인증을 요청합니다.
- 키 공유를 활성화하려면 다른 Matrix 클라이언트(Element 등)에서 기기를 인증하세요.
- 암호화 모듈을 로드할 수 없으면 E2EE가 비활성화되고 암호화된 룸을 복호화할 수 없으며, OpenClaw가 경고를 기록합니다.
- 암호화 모듈 누락 오류(예: `@matrix-org/matrix-sdk-crypto-nodejs-*`)가 보이면, `@matrix-org/matrix-sdk-crypto-nodejs`의 빌드 스크립트를 허용하고 `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs`를 실행하거나 `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`로 바이너리를 가져오세요.

암호화 상태는 계정 + 접근 토큰별로 `~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`(SQLite 데이터베이스)에 저장됩니다. 동기화 상태는 같은 디렉토리의 `bot-storage.json`에 저장됩니다. 접근 토큰(기기)이 변경되면 새 저장소가 생성되고, 봇은 암호화된 룸에서 사용하려면 다시 인증해야 합니다.

**기기 인증:**
E2EE를 활성화하면 봇이 시작 시 다른 세션에서 인증을 요청합니다. Element(또는 다른 클라이언트)를 열고 인증 요청을 승인하여 신뢰를 설정하세요. 인증이 완료되면 봇은 암호화된 룸에서 메시지를 복호화할 수 있습니다.

## 라우팅 모델

- 응답은 항상 Matrix로 돌아갑니다.
- DM은 에이전트의 기본 세션을 공유하며, 룸은 그룹 세션에 매핑됩니다.

## 접근 제어(DM)

- 기본값: `channels.matrix.dm.policy = "pairing"`. 알 수 없는 발신자는 페어링 코드를 받습니다.
- 승인 방법:
  - `openclaw pairing list matrix`
  - `openclaw pairing approve matrix <CODE>`
- 공개 DM: `channels.matrix.dm.policy="open"` + `channels.matrix.dm.allowFrom=["*"]`.
- `channels.matrix.dm.allowFrom`은 사용자 ID 또는 표시 이름을 허용합니다. 마법사는 디렉토리 검색이 가능할 때 표시 이름을 사용자 ID로 해석합니다.

## 룸(그룹)

- 기본값: `channels.matrix.groupPolicy = "allowlist"`(멘션 게이트). `channels.defaults.groupPolicy`를 사용하여 설정되지 않은 경우 기본값 재정의.
- `channels.matrix.groups`를 사용하여 허용 목록의 룸(룸 ID, 별칭 또는 이름):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

- `requireMention: false`는 해당 룸의 자동 응답을 활성화합니다.
- `groups."*"`로 룸 전체의 멘션 게이트 기본값 설정 가능.
- `groupAllowFrom`은 룸에서 봇을 트리거할 수 있는 발신자를 제한합니다(선택).
- 룸별 `users` 허용 목록으로 특정 룸 내 발신자를 더 제한할 수 있습니다.
- 구성 마법사는 룸 허용 목록(룸 ID, 별칭 또는 이름)을 입력하라고 안내하고 가능한 경우 이름을 해석합니다.
- 시작 시 OpenClaw는 허용 목록의 룸/사용자 이름을 ID로 해석하고 매핑을 기록합니다. 해석되지 않은 항목은 그대로 유지됩니다.
- 초대는 기본적으로 자동 참가됩니다. `channels.matrix.autoJoin`과 `channels.matrix.autoJoinAllowlist`로 제어.
- 룸을 **허용하지 않으려면** `channels.matrix.groupPolicy: "disabled"`를 설정(또는 빈 허용 목록 유지).
- 레거시 키: `channels.matrix.rooms`(`groups`와 동일한 구조).

## 스레드

- 스레드 답장 지원.
- `channels.matrix.threadReplies`는 응답이 스레드에 유지되는지 제어:
  - `off`, `inbound`(기본값), `always`
- `channels.matrix.replyToMode`는 스레드에 응답하지 않을 때 reply-to 메타데이터 제어:
  - `off`(기본값), `first`, `all`

## 기능

| 기능 | 상태 |
| --- | --- |
| DM | ✅ 지원 |
| 룸 | ✅ 지원 |
| 스레드 | ✅ 지원 |
| 미디어 | ✅ 지원 |
| 종단간 암호화 | ✅ 지원(암호화 모듈 필요) |
| 반응 | ✅ 지원(도구를 통해 전송/읽기) |
| 투표 | ✅ 전송 지원; 인바운드 poll start는 텍스트로 변환(응답/종료 무시) |
| 위치 | ✅ 지원(geo URI; 고도 무시) |
| 네이티브 명령 | ✅ 지원 |

## 구성 참조(Matrix)

전체 구성: [구성](/gateway/configuration)

제공자 옵션:

- `channels.matrix.enabled`: 채널 시작 활성화/비활성화.
- `channels.matrix.homeserver`: 홈서버 URL.
- `channels.matrix.accessToken`: 접근 토큰.
- `channels.matrix.userId`: 사용자 ID(선택, 접근 토큰과 함께 자동 획득).
- `channels.matrix.password`: 비밀번호(접근 토큰 대신 사용 가능).
- `channels.matrix.encryption`: 종단간 암호화 활성화.
- `channels.matrix.dm.policy`: DM 정책.
- `channels.matrix.dm.allowFrom`: DM 허용 목록.
- `channels.matrix.groupPolicy`: 그룹 정책.
- `channels.matrix.groups`: 룸 허용 목록 및 설정.
