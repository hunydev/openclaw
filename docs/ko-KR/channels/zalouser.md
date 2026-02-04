---
read_when:
  - OpenClaw Zalo Personal 설정 시
  - Zalo Personal 로그인 또는 메시지 흐름 디버깅 시
summary: zca-cli(QR 코드 로그인)를 통한 Zalo 개인 계정 지원, 기능 및 설정
title: Zalo Personal
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 2a249728d556e5cc52274627bdaf390fa10e815afa04f4497feb57a2a0cb9261
  source_path: channels/zalouser.md
  workflow: 14
---

# Zalo Personal(비공식)

상태: 실험적. 이 통합은 `zca-cli`를 통해 **개인 Zalo 계정**을 자동화합니다.

> **경고:** 이것은 비공식 통합이며 계정 정지 또는 차단을 초래할 수 있습니다. 사용에 따른 위험은 본인이 부담합니다.

## 플러그인 필요

Zalo Personal은 플러그인으로 제공되며 코어 설치에 포함되어 있지 않습니다.

- CLI를 통해 설치: `openclaw plugins install @openclaw/zalouser`
- 또는 소스 체크아웃에서: `openclaw plugins install ./extensions/zalouser`
- 자세한 내용: [플러그인](/plugin)

## 사전 요구사항: zca-cli

Gateway 머신의 `PATH`에 `zca` 실행 파일이 있어야 합니다.

- 확인: `zca --version`
- 없으면 zca-cli를 설치하세요(`extensions/zalouser/README.md` 또는 업스트림 zca-cli 문서 참조).

## 빠른 설정(초보자용)

1. 플러그인 설치(위 참조).
2. 로그인(QR 코드, Gateway 머신에서):
   - `openclaw channels login --channel zalouser`
   - 터미널에 표시된 QR 코드를 Zalo 모바일 앱으로 스캔.
3. 채널 활성화:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4. Gateway 재시작(또는 온보딩 완료).
5. DM 접근은 기본적으로 페어링 모드; 첫 연락 시 페어링 코드 승인.

## 기능

- `zca listen`을 사용하여 인바운드 메시지 수신.
- `zca msg ...`를 사용하여 응답 전송(텍스트/미디어/링크).
- Zalo Bot API를 사용할 수 없을 때 "개인 계정" 사용 사례를 위해 설계됨.

## 명명 참고

채널 ID는 `zalouser`로, **개인 Zalo 사용자 계정**(비공식)의 자동화임을 명확히 합니다. `zalo`는 향후 공식 Zalo API 통합을 위해 예약합니다.

## ID 찾기(디렉토리)

디렉토리 CLI를 사용하여 연락처/그룹과 ID 검색:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## 제한

- 아웃바운드 텍스트는 약 2000자로 청크 분할됨(Zalo 클라이언트 제한).
- 스트리밍은 기본적으로 비활성화됨.

## 접근 제어(DM)

`channels.zalouser.dmPolicy`는 `pairing | allowlist | open | disabled`를 지원합니다(기본값: `pairing`).
`channels.zalouser.allowFrom`은 사용자 ID 또는 이름을 허용합니다. 마법사는 가능한 경우 `zca friend find`를 통해 이름을 ID로 해석합니다.

승인 방법:

- `openclaw pairing list zalouser`
- `openclaw pairing approve zalouser <code>`

## 그룹 접근(선택사항)

- 기본값: `channels.zalouser.groupPolicy = "open"`(그룹 허용). 설정되지 않은 경우 `channels.defaults.groupPolicy`로 기본값 재정의.
- 허용 목록으로 제한:
  - `channels.zalouser.groupPolicy = "allowlist"`
  - `channels.zalouser.groups`(키는 그룹 ID 또는 이름)
- 모든 그룹 허용 안 함: `channels.zalouser.groupPolicy = "disabled"`.
- 설정 마법사가 그룹 허용 목록 설정을 요청할 수 있습니다.
- 시작 시 OpenClaw는 허용 목록의 그룹/사용자 이름을 ID로 해석하고 매핑을 로깅합니다; 해석되지 않은 항목은 그대로 유지.

예시:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

## 다중 계정

계정은 zca 프로필에 매핑됩니다. 예시:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## 문제 해결

**`zca`를 찾을 수 없음:**

- zca-cli를 설치하고 Gateway 프로세스의 `PATH`에 명령이 있는지 확인하세요.

**로그인 상태가 유지되지 않음:**

- `openclaw channels status --probe`
- 다시 로그인: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`
