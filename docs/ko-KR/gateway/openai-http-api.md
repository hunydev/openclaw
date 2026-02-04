---
read_when:
  - OpenAI Chat Completions가 필요한 도구 통합 시
summary: Gateway에서 OpenAI 호환 /v1/chat/completions HTTP 엔드포인트 노출
title: OpenAI Chat Completions
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 6f935777f489bff925a3bf18b1e4b7493f83ae7b1e581890092e5779af59b732
  source_path: gateway/openai-http-api.md
  workflow: 14
---

# OpenAI Chat Completions (HTTP)

OpenClaw Gateway는 작은 OpenAI 호환 Chat Completions 엔드포인트를 제공할 수 있습니다.

이 엔드포인트는 **기본적으로 비활성화**되어 있습니다. 먼저 설정에서 활성화하세요.

- `POST /v1/chat/completions`
- Gateway와 동일한 포트 사용(WS + HTTP 다중화): `http://<gateway-host>:<port>/v1/chat/completions`

내부적으로 요청은 일반 Gateway 에이전트 실행으로 처리됩니다(`openclaw agent`와 동일한 코드 경로). 따라서 라우팅/권한/설정이 Gateway와 일치합니다.

## 인증

Gateway의 인증 설정을 사용합니다. Bearer 토큰을 전송하세요:

- `Authorization: Bearer <token>`

참고:

- `gateway.auth.mode="token"`일 때 `gateway.auth.token`(또는 `OPENCLAW_GATEWAY_TOKEN`)을 사용합니다.
- `gateway.auth.mode="password"`일 때 `gateway.auth.password`(또는 `OPENCLAW_GATEWAY_PASSWORD`)를 사용합니다.

## 에이전트 선택

커스텀 헤더 없이: OpenAI의 `model` 필드에 에이전트 ID를 인코딩합니다:

- `model: "openclaw:<agentId>"`(예: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"`(별칭)

또는 헤더를 통해 특정 OpenClaw 에이전트를 지정합니다:

- `x-openclaw-agent-id: <agentId>`(기본값: `main`)

고급 사용법:

- `x-openclaw-session-key: <sessionKey>`로 세션 라우팅을 완전히 제어합니다.

## 엔드포인트 활성화

`gateway.http.endpoints.chatCompletions.enabled`를 `true`로 설정합니다:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## 엔드포인트 비활성화

`gateway.http.endpoints.chatCompletions.enabled`를 `false`로 설정합니다:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## 세션 동작

기본적으로 엔드포인트는 **요청당 무상태**입니다(매번 새로운 세션 키 생성).

요청에 OpenAI의 `user` 문자열이 포함된 경우, Gateway는 이를 기반으로 안정적인 세션 키를 파생하여 반복 호출이 동일한 에이전트 세션을 공유할 수 있습니다.

## 스트리밍 (SSE)

서버 전송 이벤트(SSE)를 받으려면 `stream: true`를 설정합니다:

- `Content-Type: text/event-stream`
- 각 이벤트 라인은 `data: <json>` 형식
- 스트림은 `data: [DONE]`으로 종료

## 예제

비스트리밍:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

스트리밍:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
