---
read_when:
  - OpenClaw가 Nostr를 통해 DM을 받도록 설정할 때
  - 탈중앙화 메시징 설정 시
summary: NIP-04 암호화 메시지를 통한 Nostr DM 채널
title: Nostr
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 6b9fe4c74bf5e7c0f59bbaa129ec5270fd29a248551a8a9a7dde6cff8fb46111
  source_path: channels/nostr.md
  workflow: 14
---

# Nostr

**상태:** 선택적 플러그인(기본 비활성화).

Nostr는 탈중앙화 소셜 네트워크 프로토콜입니다. 이 채널은 OpenClaw가 NIP-04를 통해 암호화된 DM(다이렉트 메시지)을 받고 응답할 수 있게 합니다.

## 설치(필요 시)

### 온보딩(권장)

- 온보딩 마법사(`openclaw onboard`)와 `openclaw channels add`는 선택적 채널 플러그인을 나열합니다.
- Nostr를 선택하면 필요 시 플러그인 설치를 요청합니다.

설치 기본 동작:

- **개발 채널 + git 체크아웃 사용 가능:** 로컬 플러그인 경로 사용.
- **안정/베타 릴리스:** npm에서 다운로드.

프롬프트에서 항상 이 선택을 재정의할 수 있습니다.

### 수동 설치

```bash
openclaw plugins install @openclaw/nostr
```

로컬 체크아웃 사용(개발 워크플로):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

플러그인 설치 또는 활성화 후 Gateway를 재시작하세요.

## 빠른 설정

1. Nostr 키 쌍 생성(필요 시):

```bash
# nak 사용
nak key generate
```

2. 설정에 추가:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. 키 내보내기:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Gateway 재시작.

## 설정 참조

| 키           | 타입     | 기본값                                      | 설명                          |
| ------------ | -------- | ------------------------------------------- | ----------------------------- |
| `privateKey` | string   | 필수                                        | `nsec` 또는 hex 형식의 개인키 |
| `relays`     | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | 릴레이 URL(WebSocket)         |
| `dmPolicy`   | string   | `pairing`                                   | DM 접근 정책                  |
| `allowFrom`  | string[] | `[]`                                        | 허용된 발신자 공개키          |
| `enabled`    | boolean  | `true`                                      | 채널 활성화/비활성화          |
| `name`       | string   | -                                           | 표시 이름                     |
| `profile`    | object   | -                                           | NIP-01 프로필 메타데이터      |

## 프로필 메타데이터

프로필 데이터는 NIP-01 `kind:0` 이벤트로 게시됩니다. 제어 UI(Channels -> Nostr -> Profile)에서 관리하거나 설정에서 직접 설정할 수 있습니다.

예시:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Personal assistant DM bot",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

참고:

- 프로필 URL은 `https://`를 사용해야 합니다.
- 릴레이에서 가져오기는 필드를 병합하고 로컬 재정의를 유지합니다.

## 접근 제어

### DM 정책

- **pairing**(기본값): 알 수 없는 발신자는 페어링 코드를 받습니다.
- **allowlist**: `allowFrom`의 공개키만 DM 가능.
- **open**: 인바운드 DM 공개(`allowFrom: ["*"]` 필요).
- **disabled**: 인바운드 DM 무시.

### 허용 목록 예시

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## 키 형식

허용되는 형식:

- **개인키:** `nsec...` 또는 64자 hex
- **공개키(`allowFrom`):** `npub...` 또는 hex
