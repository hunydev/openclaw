---
read_when:
  - 전체 에이전트 턴 없이 직접 도구 호출 시
  - 도구 정책 적용이 필요한 자동화 워크플로우 구축 시
summary: Gateway HTTP 엔드포인트를 통해 개별 도구 직접 호출
title: 도구 호출 API
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 17ccfbe0b0d9bb61cc46fb21f5c09b106ba6e8e4e4c56a5c69984847685d518c
  source_path: gateway/tools-invoke-http-api.md
  workflow: 14
---

# 도구 호출 (HTTP)

OpenClaw Gateway는 개별 도구를 직접 호출할 수 있는 간단한 HTTP 엔드포인트를 노출합니다. 이 엔드포인트는 항상 활성화되어 있지만 Gateway 인증 및 도구 정책의 적용을 받습니다.

- `POST /tools/invoke`
- Gateway와 동일한 포트(WS + HTTP 다중화): `http://<gateway-host>:<port>/tools/invoke`

기본 최대 요청 본문 크기는 2MB입니다.

## 인증

Gateway 인증 설정을 사용합니다. Bearer 토큰을 전송하세요:

- `Authorization: Bearer <token>`

참고:

- `gateway.auth.mode="token"`일 때 `gateway.auth.token`(또는 `OPENCLAW_GATEWAY_TOKEN`)을 사용합니다.
- `gateway.auth.mode="password"`일 때 `gateway.auth.password`(또는 `OPENCLAW_GATEWAY_PASSWORD`)를 사용합니다.

## 요청 본문

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

필드:

- `tool`(문자열, 필수): 호출할 도구 이름.
- `action`(문자열, 선택): 도구 스키마가 `action`을 지원하고 args에 포함되지 않은 경우 args로 매핑됩니다.
- `args`(객체, 선택): 도구별 인수.
- `sessionKey`(문자열, 선택): 대상 세션 키. 생략하거나 `"main"`이면 Gateway는 설정된 메인 세션 키를 사용합니다(`session.mainKey` 및 기본 에이전트를 따르거나, 전역 범위에서는 `global` 사용).
- `dryRun`(불린, 선택): 향후 사용을 위해 예약됨. 현재는 무시됩니다.

## 정책 + 라우팅 동작

도구 가용성은 Gateway 에이전트와 동일한 정책 체인을 통해 필터링됩니다:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- 그룹 정책(세션 키가 그룹 또는 채널에 매핑되는 경우)
- 서브 에이전트 정책(서브 에이전트 세션 키로 호출 시)

정책에서 도구가 허용되지 않으면 엔드포인트는 **404**를 반환합니다.

그룹 정책이 컨텍스트를 해석하는 데 도움이 되도록 선택적으로 설정할 수 있습니다:

- `x-openclaw-message-channel: <channel>`(예: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>`(여러 계정이 있는 경우)

## 응답

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }`(잘못된 요청 또는 도구 오류)
- `401` → 인증되지 않음
- `404` → 도구를 사용할 수 없음(찾을 수 없거나 허용되지 않음)
- `405` → 메서드가 허용되지 않음

## 예제

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
