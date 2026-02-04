---
read_when:
  - web_search에 Perplexity Sonar를 사용하려는 경우
  - PERPLEXITY_API_KEY 또는 OpenRouter 설정이 필요한 경우
summary: Perplexity Sonar의 web_search 설정
title: Perplexity Sonar
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Perplexity Sonar

OpenClaw는 Perplexity Sonar를 `web_search` 도구로 사용할 수 있습니다. Perplexity의 직접 API 또는 OpenRouter를 통해 연결할 수 있습니다.

## API 옵션

### Perplexity(직접)

- Base URL: https://api.perplexity.ai
- 환경 변수: `PERPLEXITY_API_KEY`

### OpenRouter(대안)

- Base URL: https://openrouter.ai/api/v1
- 환경 변수: `OPENROUTER_API_KEY`
- 선불/암호화폐 크레딧 지원.

## 구성 예시

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Brave에서 전환

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

`PERPLEXITY_API_KEY`와 `OPENROUTER_API_KEY`가 모두 설정된 경우 `tools.web.search.perplexity.baseUrl`(또는 `tools.web.search.perplexity.apiKey`)을 설정하여 명확히 하세요.

base URL이 설정되지 않으면 OpenClaw는 API 키 출처에 따라 기본값을 선택합니다:

- `PERPLEXITY_API_KEY` 또는 `pplx-...` → 직접 Perplexity(`https://api.perplexity.ai`)
- `OPENROUTER_API_KEY` 또는 `sk-or-...` → OpenRouter(`https://openrouter.ai/api/v1`)
- 알 수 없는 키 형식 → OpenRouter(안전한 폴백)

## 모델

- `perplexity/sonar` — 웹 검색을 포함한 빠른 Q&A
- `perplexity/sonar-pro`(기본값) — 다단계 추론 + 웹 검색
- `perplexity/sonar-reasoning-pro` — 깊은 리서치

전체 web_search 구성은 [웹 도구](/tools/web)를 참조하세요.
