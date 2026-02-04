---
read_when:
  - Token 사용량, 비용 또는 컨텍스트 윈도우를 설명할 때
  - 컨텍스트 증가 또는 압축 동작을 디버깅할 때
summary: OpenClaw가 프롬프트 컨텍스트를 구성하고 토큰 사용량과 비용을 보고하는 방법
title: Token 사용량과 비용
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Token 사용량과 비용

OpenClaw는 문자가 아닌 **토큰**을 추적합니다. 토큰은 모델마다 다르지만 대부분의 OpenAI 스타일 모델은 영문 텍스트에 대해 평균적으로 약 4자당 1토큰입니다.

## 시스템 프롬프트 구성 방식

OpenClaw는 매 실행마다 자체 시스템 프롬프트를 조립합니다. 여기에는 다음이 포함됩니다:

- 도구 목록 + 간략한 설명
- Skills 목록(메타데이터만; 지침은 `read`를 통해 온디맨드로 로드)
- 자체 업데이트 지침
- 워크스페이스 + 부트스트랩 파일(`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`(새 세션에서만 로드)). 큰 파일은 `agents.defaults.bootstrapMaxChars`(기본값: 20000)로 잘립니다.
- 시간(UTC + 사용자 시간대)
- 응답 태그 + 하트비트 동작
- 런타임 메타데이터(호스트/OS/모델/사고)

전체 분석은 [시스템 프롬프트](/concepts/system-prompt)를 참조하세요.

## 컨텍스트 윈도우에 포함되는 것

모델이 받는 모든 것이 컨텍스트 제한에 포함됩니다:

- 시스템 프롬프트(위에 나열된 모든 부분)
- 대화 기록(사용자 + 어시스턴트 메시지)
- 도구 호출 및 도구 결과
- 첨부 파일/전사(이미지, 오디오, 파일)
- 압축 요약 및 정리 아티팩트
- 제공자 래퍼 또는 안전 헤더(보이지 않지만 여전히 포함)

실제 분석(주입된 파일, 도구, Skills 및 시스템 프롬프트 크기별)을 보려면 `/context list` 또는 `/context detail`을 사용하세요. [컨텍스트](/concepts/context)를 참조하세요.

## 현재 토큰 사용량 확인 방법

채팅에서 다음 명령을 사용하세요:

- `/status` → **풍부한 상태 카드**, 세션 모델, 컨텍스트 사용량, 마지막 응답의 입력/출력 토큰 수 및 **예상 비용**(API 키 모드에서만) 표시.
- `/usage off|tokens|full` → 각 응답 후 **응답별 사용량 푸터** 추가.
  - 세션별로 지속됨(`responseUsage`로 저장).
  - OAuth 인증에서는 **비용 숨김**(토큰 수만 표시).
- `/usage cost` → OpenClaw 세션 로그에서 로컬 비용 요약을 표시.

기타 인터페이스:

- **TUI/Web TUI:** `/status` + `/usage` 지원.
- **CLI:** `openclaw status --usage` 및 `openclaw channels list`는 제공자 할당량 창(응답별 비용이 아님)을 표시.

## 비용 추정(표시 시점)

비용은 모델 가격 구성에 따라 추정됩니다:

```
models.providers.<provider>.models[].cost
```

이는 `input`, `output`, `cacheRead`, `cacheWrite`에 대한 **백만 토큰당 달러 가격**입니다. 가격 정보가 없으면 OpenClaw는 토큰 수만 표시합니다. OAuth 토큰은 달러 비용을 표시하지 않습니다.

## 캐시 TTL 및 정리 영향

제공자의 프롬프트 캐시는 캐시 TTL 윈도우 내에서만 유효합니다. OpenClaw는 선택적으로 **캐시 TTL 정리**를 실행할 수 있습니다: 캐시 TTL이 만료된 후 세션을 정리하여 캐시 윈도우를 재설정하고, 후속 요청이 전체 기록을 다시 캐싱하는 대신 새로 캐싱된 컨텍스트를 재사용할 수 있게 합니다. 이렇게 하면 세션이 TTL보다 오래 유휴 상태일 때 캐시 쓰기 비용을 줄일 수 있습니다.

[Gateway 구성](/gateway/configuration)에서 구성하고 [세션 정리](/concepts/session-pruning)에서 동작 세부 정보를 확인하세요.

하트비트는 유휴 간격 동안 캐시를 **활성** 상태로 유지할 수 있습니다. 모델 캐시 TTL이 `1h`인 경우 하트비트 간격을 그보다 약간 짧게(예: `55m`) 설정하면 전체 프롬프트를 다시 캐싱하는 것을 피하여 캐시 쓰기 비용을 줄일 수 있습니다.

Anthropic API 가격에 대해, 캐시 읽기는 입력 토큰보다 훨씬 저렴하고 캐시 쓰기는 더 높은 배수로 청구됩니다. 최신 요율과 TTL 배수는 Anthropic의 프롬프트 캐싱 가격을 참조하세요:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

### 예시: 하트비트로 1시간 캐시 활성 유지

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

## 토큰 압력을 줄이는 팁

- `/compact`를 사용하여 긴 세션을 요약합니다.
- 워크플로에서 큰 도구 출력을 정리합니다.
- Skills 설명을 짧게 유지합니다(Skills 목록이 프롬프트에 주입됨).
- 장황한 탐색 작업에는 작은 모델을 선호합니다.

Skills 목록 오버헤드의 정확한 공식은 [Skills](/tools/skills)를 참조하세요.
