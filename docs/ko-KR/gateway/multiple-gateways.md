---
read_when:
  - 동일 머신에서 여러 Gateway 실행 시
  - 각 Gateway에 별도의 구성/상태/포트가 필요할 때
summary: 동일 호스트에서 여러 OpenClaw Gateway 실행 (격리, 포트 및 프로필)
title: 다중 Gateway
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 09b5035d4e5fb97c8d4596f7e23dea67224dad3b6d9e2c37ecb99840f28bd77d
  source_path: gateway/multiple-gateways.md
  workflow: 14
---

# 다중 Gateway (동일 호스트)

대부분의 시나리오에서는 하나의 Gateway로 충분합니다. 단일 Gateway가 여러 메시징 연결과 에이전트를 처리할 수 있기 때문입니다. 더 강력한 격리나 중복성 (예: 구조 봇)이 필요하면 별도의 프로필/포트로 여러 Gateway를 실행하세요.

## 격리 체크리스트 (필수)

- `OPENCLAW_CONFIG_PATH` — 인스턴스별 별도의 구성 파일
- `OPENCLAW_STATE_DIR` — 인스턴스별 별도의 세션, 자격 증명, 캐시
- `agents.defaults.workspace` — 인스턴스별 별도의 작업 공간 루트
- `gateway.port` (또는 `--port`) — 인스턴스별 고유
- 파생 포트 (브라우저/캔버스)가 겹치지 않아야 함

이것들이 공유되면 구성 경쟁과 포트 충돌이 발생합니다.

## 권장 방법: 프로필 (`--profile`)

프로필은 `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH`의 범위를 자동으로 지정하고 서비스 이름에 접미사를 추가합니다.

```bash
# 메인 인스턴스
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# 구조 인스턴스
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

프로필별 서비스 설치:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## 구조 봇 가이드

동일 호스트에서 두 번째 Gateway를 실행하며 각각 별도의:

- 프로필/구성
- 상태 디렉토리
- 작업 공간
- 기본 포트 (및 파생 포트)

이렇게 하면 구조 봇이 메인 봇과 격리되어 메인 봇이 다운되었을 때도 디버깅하거나 구성 변경을 적용할 수 있습니다.

포트 간격: 기본 포트 사이에 최소 20개 포트 간격을 두어 파생된 브라우저/캔버스/CDP 포트가 충돌하지 않도록 하세요.

### 설치 방법 (구조 봇)

```bash
# 메인 봇 (기존 또는 새로 설치, --profile 파라미터 없음)
# 포트 18789 + Chrome CDC/Canvas/... 포트에서 실행
openclaw onboard
openclaw gateway install

# 구조 봇 (별도 프로필 + 포트)
openclaw --profile rescue onboard
# 참고:
# - 작업 공간 이름에 기본적으로 -rescue 접미사가 붙음
# - 포트는 최소 18789 + 20 포트 이상이어야 함,
#   완전히 다른 기본 포트 (예: 19789) 권장
# - 나머지 온보딩 흐름은 일반 흐름과 동일

# 서비스 설치 (온보딩 중 자동 설치되지 않은 경우)
openclaw --profile rescue gateway install
```

## 포트 매핑 (파생)

기본 포트 = `gateway.port` (또는 `OPENCLAW_GATEWAY_PORT` / `--port`).

- 브라우저 제어 서비스 포트 = 기본 포트 + 2 (local loopback만)
- `canvasHost.port = 기본 포트 + 4`
- 브라우저 프로필 CDP 포트는 `browser.controlPort + 9 .. + 108`에서 자동 할당

구성이나 환경 변수에서 이것들을 오버라이드하면 각 인스턴스에서 값이 고유한지 확인해야 합니다.

## 브라우저/CDP 참고 사항 (일반적인 함정)

- 여러 인스턴스에서 `browser.cdpUrl`을 동일한 값으로 설정하지 **마세요**.
- 각 인스턴스에는 별도의 브라우저 제어 포트와 CDP 포트 범위가 필요합니다 (Gateway 포트에서 파생).
- CDP 포트를 명시적으로 지정하려면 각 인스턴스에 `browser.profiles.<name>.cdpPort`를 설정하세요.
- 원격 Chrome: `browser.profiles.<name>.cdpUrl` 사용 (프로필별, 인스턴스별 설정).

## 수동 환경 변수 예제

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## 빠른 확인

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
