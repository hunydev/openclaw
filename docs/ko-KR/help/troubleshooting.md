---
read_when:
  - 오류가 발생했고 수정 방법을 찾고 싶을 때
  - 설치 프로그램이 "성공"을 표시하지만 CLI가 작동하지 않을 때
summary: 문제 해결 센터: 증상 → 확인 → 수정
title: 문제 해결
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 문제 해결

## 처음 60초

다음 명령을 순서대로 실행하세요:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Gateway에 연결할 수 있으면 심층 탐색을 수행하세요:

```bash
openclaw status --deep
```

## 일반적인 "문제가 발생했습니다" 시나리오

### `openclaw: command not found`

거의 항상 Node/npm PATH 문제입니다. 여기서 시작하세요:

- [설치(Node/npm PATH 상태 확인)](/install#nodejs--npm-path-sanity)

### 설치 프로그램 실패(또는 전체 로그 필요)

자세한 모드로 설치 프로그램을 다시 실행하여 전체 추적 및 npm 출력을 확인하세요:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

베타 설치의 경우:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

플래그 대신 `OPENCLAW_VERBOSE=1`을 설정할 수도 있습니다.

### Gateway "unauthorized", 연결 불가 또는 계속 재연결

- [Gateway 문제 해결](/gateway/troubleshooting)
- [Gateway 인증](/gateway/authentication)

### 제어 UI가 HTTP에서 실패(장치 신원 필요)

- [Gateway 문제 해결](/gateway/troubleshooting)
- [제어 UI](/web/control-ui#insecure-http)

### `docs.openclaw.ai`에서 SSL 오류 표시(Comcast/Xfinity)

일부 Comcast/Xfinity 연결은 Xfinity Advanced Security를 통해 `docs.openclaw.ai`를 차단합니다.
Advanced Security를 비활성화하거나 `docs.openclaw.ai`를 허용 목록에 추가한 다음 다시 시도하세요.

- Xfinity Advanced Security 도움말: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
- 빠른 확인: 모바일 핫스팟이나 VPN을 시도하여 ISP 수준 필터링인지 확인

### 서비스가 실행 중으로 표시되지만 RPC 탐색 실패

- [Gateway 문제 해결](/gateway/troubleshooting)
- [백그라운드 프로세스/서비스](/gateway/background-process)

### 모델/인증 실패(속도 제한, 청구, "모든 모델 실패")

- [모델](/cli/models)
- [OAuth / 인증 개념](/concepts/oauth)

### `/model`에서 `model not allowed` 표시

이는 일반적으로 `agents.defaults.models`가 허용 목록으로 구성되어 있음을 의미합니다. 비어 있지 않으면 해당 제공자/모델 키만 선택할 수 있습니다.

- 허용 목록 확인: `openclaw config get agents.defaults.models`
- 필요한 모델을 추가(또는 허용 목록을 비움)한 다음 `/model` 다시 시도
- `/models`로 허용된 제공자/모델 탐색

### 이슈 제출 시

안전 보고서를 붙여넣으세요:

```bash
openclaw status --all
```

가능하면 `openclaw logs --follow`의 관련 로그 끝부분을 첨부하세요.
