---
read_when:
  - 어떤 환경 변수가 로드되는지와 그 순서를 알아야 하는 경우
  - Gateway에서 누락된 API 키를 디버깅하는 경우
  - 제공자 인증 또는 배포 환경 문서를 작성하는 경우
summary: OpenClaw가 환경 변수를 어디서 로드하고 우선순위 순서
title: 환경 변수
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 환경 변수

OpenClaw는 여러 소스에서 환경 변수를 가져옵니다. 규칙은 **이미 존재하는 값을 절대 덮어쓰지 않는 것**입니다.

## 우선순위(높은 순서에서 낮은 순서)

1. **프로세스 환경**(Gateway 프로세스가 부모 shell/데몬에서 상속받은 변수).
2. **현재 작업 디렉토리의 `.env`**(dotenv 기본 동작; 기존 값을 덮어쓰지 않음).
3. **전역 `.env`**, `~/.openclaw/.env`에 위치(즉 `$OPENCLAW_STATE_DIR/.env`; 기존 값을 덮어쓰지 않음).
4. **구성 파일의 `env` 블록**, `~/.openclaw/openclaw.json`에 위치(변수가 누락된 경우에만 적용).
5. **선택적 로그인 shell 가져오기**(`env.shellEnv.enabled` 또는 `OPENCLAW_LOAD_SHELL_ENV=1`), 누락된 예상 키에 대해서만 작동.

구성 파일이 전혀 존재하지 않으면 4단계를 건너뜁니다. 활성화된 경우 shell 가져오기는 여전히 실행됩니다.

## 구성 파일 `env` 블록

인라인 환경 변수를 설정하는 두 가지 동등한 방법(둘 다 기존 값을 덮어쓰지 않음):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Shell 환경 가져오기

`env.shellEnv`는 로그인 shell을 실행하고 **누락된** 예상 키만 가져옵니다:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

동등한 환경 변수:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## 구성에서 환경 변수 치환

`${VAR_NAME}` 구문을 사용하여 구성 문자열 값에서 직접 환경 변수를 참조할 수 있습니다:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

자세한 내용은 [구성: 환경 변수 치환](/gateway/configuration#env-var-substitution-in-config)을 참조하세요.

## 관련 내용

- [Gateway 구성](/gateway/configuration)
- [FAQ: 환경 변수 및 .env 로딩](/help/faq#env-vars-and-env-loading)
- [모델 개요](/concepts/models)
