---
read_when:
  - 자체 GPU 장치에 모델을 배포하고 싶을 때
  - LM Studio 또는 OpenAI 호환 프록시를 구성할 때
  - 가장 안전한 로컬 모델 가이드가 필요할 때
summary: 로컬 LLM에서 OpenClaw 실행 (LM Studio, vLLM, LiteLLM, 커스텀 OpenAI 엔드포인트)
title: 로컬 모델
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: f72b424c3d8986319868dc4c552596bcd599cc79fab5a57c14bf4f0695c39690
  source_path: gateway/local-models.md
  workflow: 14
---

# 로컬 모델

로컬 배포는 가능하지만 OpenClaw는 대용량 컨텍스트 창과 강력한 프롬프트 인젝션 방어 능력이 필요합니다. 적은 VRAM은 컨텍스트를 잘라내고 보안을 낮춥니다. 높은 사양 권장: **≥2대의 풀 스펙 Mac Studio 또는 동급 GPU 장치 (약 $30k+)**. 단일 **24 GB** 카드는 가벼운 프롬프트에만 적합하며 지연이 높습니다. **실행 가능한 가장 크고 완전한 버전의 모델**을 사용하세요; 공격적인 양자화 또는 "소형" 체크포인트는 프롬프트 인젝션 위험을 높입니다 ([보안](/gateway/security) 참조).

## 권장 방법: LM Studio + MiniMax M2.1 (Responses API, 전체 버전)

현재 최고의 로컬 기술 스택입니다. LM Studio에서 MiniMax M2.1을 로드하고, 로컬 서버를 활성화하고 (기본값 `http://127.0.0.1:1234`), Responses API를 사용하여 추론 과정과 최종 텍스트를 분리합니다.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**구성 체크리스트**

- LM Studio 설치: https://lmstudio.ai
- LM Studio에서 **사용 가능한 가장 큰 MiniMax M2.1 버전** 다운로드 ("소형"/과도한 양자화 버전 피함), 서버 시작, `http://127.0.0.1:1234/v1/models`에 모델이 나열되었는지 확인.
- 모델을 로드된 상태로 유지; 콜드 로드는 시작 지연을 추가합니다.
- LM Studio 버전이 다르면 `contextWindow`/`maxTokens` 조정.
- WhatsApp의 경우 Responses API를 사용하여 최종 텍스트만 전송되도록 합니다.

로컬 모델을 실행해도 호스팅 모델 구성을 유지하세요; `models.mode: "merge"`를 사용하여 폴백을 사용 가능하게 유지.

### 하이브리드 구성: 호스팅이 주, 로컬이 폴백

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### 로컬 우선, 호스팅 폴백

주 모델과 폴백 순서를 바꿈; 동일한 providers 구성 블록과 `models.mode: "merge"`를 유지하여 로컬 장치가 다운되면 Sonnet 또는 Opus로 폴백할 수 있습니다.

### 리전 호스팅/데이터 라우팅

- MiniMax/Kimi/GLM의 호스팅 버전도 OpenRouter에서 사용 가능하며 리전 잠금 엔드포인트 (예: 미국 호스팅)를 제공합니다. 해당 리전 버전을 선택하여 트래픽을 선택한 관할권에 유지하면서 `models.mode: "merge"`를 통해 Anthropic/OpenAI 폴백을 여전히 사용할 수 있습니다.
- 순수 로컬 배포가 가장 강력한 프라이버시 보호; 호스팅 리전 라우팅은 프로바이더 기능이 필요하지만 데이터 흐름을 제어하고 싶을 때의 타협안입니다.

## 기타 OpenAI 호환 로컬 프록시

vLLM, LiteLLM, OAI-proxy 또는 커스텀 게이트웨이 모두 OpenAI 스타일 `/v1` 엔드포인트를 노출하면 작동합니다. 위의 프로바이더 구성 블록을 자신의 엔드포인트와 모델 ID로 교체하세요:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

호스팅 모델을 폴백으로 사용 가능하게 유지하려면 `models.mode: "merge"`를 유지하세요.
