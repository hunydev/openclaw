---
read_when:
  - web_search를 위해 Brave Search를 사용하려는 경우
  - BRAVE_API_KEY 또는 요금제 세부 정보가 필요한 경우
summary: web_search를 위한 Brave Search API 설정
title: Brave Search
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Brave Search API

OpenClaw는 `web_search`의 기본 제공자로 Brave Search를 사용합니다.

## API 키 얻기

1. https://brave.com/search/api/ 에서 Brave Search API 계정을 생성합니다.
2. 대시보드에서 **Data for Search** 요금제를 선택하고 API 키를 생성합니다.
3. 키를 구성에 저장(권장)하거나 Gateway 환경에서 `BRAVE_API_KEY`를 설정합니다.

## 구성 예시

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## 참고 사항

- Data for AI 요금제는 `web_search`와 **호환되지 않습니다**.
- Brave는 무료 요금제와 유료 요금제를 제공합니다. 현재 제한 사항은 Brave API 포털을 확인하세요.

전체 web_search 구성은 [Web 도구](/tools/web)를 참조하세요.
