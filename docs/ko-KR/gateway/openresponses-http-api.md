---
read_when:
  - OpenResponses API를 사용하는 클라이언트 통합 시
  - item 기반 입력, 클라이언트 도구 호출 또는 SSE 이벤트가 필요할 때
summary: Gateway에서 OpenResponses 호환 /v1/responses HTTP 엔드포인트 노출
title: OpenResponses API
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 0597714837f8b210c38eeef53561894220c1473e54c56a5c69984847685d518c
  source_path: gateway/openresponses-http-api.md
  workflow: 14
---

# OpenResponses API (HTTP)

OpenClaw Gateway는 OpenResponses 호환 `POST /v1/responses` 엔드포인트를 제공할 수 있습니다.

이 엔드포인트는 **기본적으로 비활성화**되어 있습니다. 먼저 설정에서 활성화하세요.

- `POST /v1/responses`
- Gateway와 동일한 포트 사용(WS + HTTP 다중화): `http://<gateway-host>:<port>/v1/responses`

내부적으로 요청은 일반 Gateway 에이전트 실행으로 처리됩니다(`openclaw agent`와 동일한 코드 경로). 따라서 라우팅/권한/설정이 Gateway와 일치합니다.

## 인증

Gateway의 인증 설정을 사용합니다. bearer 토큰을 전송하세요:

- `Authorization: Bearer <token>`

참고:

- `gateway.auth.mode="token"`일 때 `gateway.auth.token`(또는 `OPENCLAW_GATEWAY_TOKEN`)을 사용합니다.
- `gateway.auth.mode="password"`일 때 `gateway.auth.password`(또는 `OPENCLAW_GATEWAY_PASSWORD`)를 사용합니다.

## 에이전트 선택

커스텀 헤더 없이: OpenResponses의 `model` 필드에 에이전트 ID를 인코딩합니다:

- `model: "openclaw:<agentId>"`(예: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"`(별칭)

또는 헤더를 통해 특정 OpenClaw 에이전트를 지정합니다:

- `x-openclaw-agent-id: <agentId>`(기본값: `main`)

고급 사용법:

- `x-openclaw-session-key: <sessionKey>`로 세션 라우팅을 완전히 제어합니다.

## 엔드포인트 활성화

`gateway.http.endpoints.responses.enabled`를 `true`로 설정합니다:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## 엔드포인트 비활성화

`gateway.http.endpoints.responses.enabled`를 `false`로 설정합니다:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## 세션 동작

기본적으로 엔드포인트는 **요청당 무상태**입니다(매번 새로운 세션 키 생성).

요청에 OpenResponses의 `user` 문자열이 포함된 경우, Gateway는 이를 기반으로 안정적인 세션 키를 파생하여 반복 호출이 동일한 에이전트 세션을 공유할 수 있습니다.

## 요청 구조(지원됨)

요청은 item 기반 입력을 사용하는 OpenResponses API를 따릅니다. 현재 지원:

- `input`: 문자열 또는 item 객체 배열.
- `instructions`: 시스템 프롬프트에 병합됨.
- `tools`: 클라이언트 도구 정의(함수 도구).
- `tool_choice`: 클라이언트 도구 필터 또는 요구.
- `stream`: SSE 스트리밍 활성화.
- `max_output_tokens`: 최선 노력 출력 제한(제공자에 따라 다름).
- `user`: 안정적인 세션 라우팅.

수락되지만 **현재 무시됨**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

## Items(입력)

### `message`

역할: `system`, `developer`, `user`, `assistant`.

- `system` 및 `developer`는 시스템 프롬프트에 추가됩니다.
- 가장 최근 `user` 또는 `function_call_output` item이 "현재 메시지"가 됩니다.
- 이전 user/assistant 메시지는 히스토리 컨텍스트로 포함됩니다.

### `function_call_output`(턴 기반 도구)

모델에 도구 결과를 다시 보냅니다:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### `reasoning` 및 `item_reference`

스키마 호환성을 위해 수락되지만 프롬프트 작성 시 무시됩니다.

## 도구(클라이언트 함수 도구)

`tools: [{ type: "function", function: { name, description?, parameters? } }]`를 통해 도구를 제공합니다.

에이전트가 도구 호출을 결정하면 응답에 `function_call` 출력 item이 반환됩니다.
그런 다음 `function_call_output`을 포함한 후속 요청을 보내 해당 턴을 계속합니다.

## 이미지(`input_image`)

base64 또는 URL 소스 지원:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

허용되는 MIME 타입(현재): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
최대 크기(현재): 10MB.

## 파일(`input_file`)

base64 또는 URL 소스 지원:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

허용되는 MIME 타입(현재): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

최대 크기(현재): 5MB.

현재 동작:

- 파일 콘텐츠가 디코딩되어 사용자 메시지가 아닌 **시스템 프롬프트**에 추가되므로 일시적입니다(세션 히스토리에 지속되지 않음).
- PDF는 텍스트 추출을 위해 파싱됩니다. 발견된 텍스트가 적으면 처음 몇 페이지가 이미지로 래스터화되어 모델에 전달됩니다.

PDF 파싱은 Node 친화적인 `pdfjs-dist` 레거시 빌드(워커 없음)를 사용합니다. 최신 PDF.js 빌드는 브라우저 워커/DOM 전역이 필요하므로 Gateway에서 사용되지 않습니다.

URL 가져오기 기본값:

- `files.allowUrl`: `true`
- `images.allowUrl`: `true`
- 요청은 보호됩니다(DNS 해석, 프라이빗 IP 차단, 리다이렉트 상한, 타임아웃).

## 파일 및 이미지 제한(설정)

기본값은 `gateway.http.endpoints.responses` 아래에서 조정할 수 있습니다:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

생략 시 기본값:

- `maxBodyBytes`: 20MB
- `files.maxBytes`: 5MB
- `files.maxChars`: 200k
- `files.maxRedirects`: 3
- `files.timeoutMs`: 10s
- `files.pdf.maxPages`: 4
- `files.pdf.maxPixels`: 4,000,000
- `files.pdf.minTextChars`: 200
- `images.maxBytes`: 10MB
- `images.maxRedirects`: 3
- `images.timeoutMs`: 10s

## 스트리밍(SSE)

서버 전송 이벤트(SSE)를 받으려면 `stream: true`를 설정합니다:

- `Content-Type: text/event-stream`
- 각 이벤트 라인은 `event: <type>` 및 `data: <json>` 형식
- 스트림은 `data: [DONE]`으로 종료

현재 발생하는 이벤트 타입:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed`(오류 시)

## 사용량

기본 제공자가 토큰 수를 보고하면 `usage`가 채워집니다.

## 오류

오류는 다음 JSON 객체를 사용합니다:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

일반적인 경우:

- `401` 인증 누락/잘못됨
- `400` 잘못된 요청 본문
- `405` 잘못된 메서드

## 예제

비스트리밍:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

스트리밍:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
