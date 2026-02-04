---
read_when:
  - 로그 출력 또는 형식 변경 시
  - CLI 또는 Gateway 출력 디버깅 시
summary: 로그 인터페이스, 파일 로그, WebSocket 로그 스타일 및 콘솔 포맷팅
title: 로깅
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: efb8eda5e77e3809369a8ff569fac110323a86b3945797093f20e9bc98f39b2e
  source_path: gateway/logging.md
  workflow: 14
---

# 로깅

사용자 대면 개요 (CLI + 제어 인터페이스 + 구성)는 [/logging](/logging)을 참조하세요.

OpenClaw에는 두 가지 로그 "인터페이스"가 있습니다:

- **콘솔 출력** (터미널 / 디버그 인터페이스에서 보는 것).
- **파일 로그** (JSON 행), Gateway 로거가 작성.

## 파일 기반 로거

- 기본 롤링 로그 파일은 `/tmp/openclaw/` 아래에 있습니다 (일별): `openclaw-YYYY-MM-DD.log`
  - 날짜는 Gateway 호스트의 현지 시간대 사용.
- 로그 파일 경로 및 레벨은 `~/.openclaw/openclaw.json`에서 구성 가능:
  - `logging.file`
  - `logging.level`

파일 형식은 행당 하나의 JSON 객체입니다.

제어 인터페이스의 로그 탭은 Gateway (`logs.tail`)를 통해 이 파일을 라이브 테일합니다.
CLI도 동일하게 할 수 있습니다:

```bash
openclaw logs --follow
```

**자세한 모드 vs 로그 레벨**

- **파일 로그**는 `logging.level`에 의해서만 제어됩니다.
- `--verbose`는 **콘솔 상세도** (및 WebSocket 로그 스타일)에만 영향을 미칩니다; 파일 로그 레벨을 **올리지 않습니다**.
- 자세한 모드에서만 표시되는 세부 정보를 파일 로그에 캡처하려면 `logging.level`을 `debug` 또는 `trace`로 설정하세요.

## 콘솔 캡처

CLI는 `console.log/info/warn/error/debug/trace`를 캡처하여 파일 로그에 쓰면서 여전히 stdout/stderr로 출력합니다.

콘솔 상세도를 독립적으로 조정할 수 있습니다:

- `logging.consoleLevel` (기본값 `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## 도구 요약 수정

자세한 도구 요약 (예: `🛠️ Exec: ...`)은 민감한 토큰이 콘솔 스트림에 들어가기 전에 마스킹할 수 있습니다. 이것은 **도구에만 적용**되며 파일 로그는 변경되지 않습니다.

- `logging.redactSensitive`: `off` | `tools` (기본값: `tools`)
- `logging.redactPatterns`: 정규식 문자열 배열 (기본값 오버라이드)
  - 원시 정규식 문자열 사용 (자동 `gi`), 사용자 정의 플래그가 필요하면 `/pattern/flags` 사용.
  - 일치 항목은 처음 6자와 마지막 4자를 유지하여 마스킹 (길이 >= 18), 그렇지 않으면 `***` 표시.
  - 기본값은 일반적인 키 할당, CLI 플래그, JSON 필드, bearer 헤더, PEM 블록 및 일반적인 토큰 접두사를 포함.

## Gateway WebSocket 로그

Gateway는 두 가지 모드로 WebSocket 프로토콜 로그를 출력합니다:

- **일반 모드 (`--verbose` 없음)**: "의미 있는" RPC 결과만 출력:
  - 오류 (`ok=false`)
  - 느린 호출 (기본 임계값: `>= 50ms`)
  - 파싱 오류
- **자세한 모드 (`--verbose`)**: 모든 WebSocket 요청/응답 트래픽 출력.

### WebSocket 로그 스타일

`openclaw gateway`는 Gateway별 설정 스타일 스위치를 지원합니다:

- `--ws-log auto` (기본값): 일반 모드는 최적화; 자세한 모드는 컴팩트 출력 사용
- `--ws-log compact`: 자세한 모드에서 컴팩트 출력 사용 (페어링된 요청/응답)
- `--ws-log full`: 자세한 모드에서 전체 프레임별 출력
- `--compact`: `--ws-log compact`의 별칭

예제:

```bash
# 최적화 모드 (오류/느린 호출만)
openclaw gateway

# 모든 WS 트래픽 표시 (페어링)
openclaw gateway --verbose --ws-log compact

# 모든 WS 트래픽 표시 (전체 메타데이터)
openclaw gateway --verbose --ws-log full
```

## 콘솔 포맷팅 (서브시스템 로그)

콘솔 포매터는 **TTY 인식**이며 일관된 접두사가 붙은 행을 출력합니다.
서브시스템 로거는 출력을 그룹화하고 스캔하기 쉽게 유지합니다.

동작:

- 각 행에 **서브시스템 접두사** (예: `[gateway]`, `[canvas]`, `[tailscale]`)
- **서브시스템 색상** (각 서브시스템에 안정적으로 할당) + 레벨 색상
- **TTY이거나 환경이 풍부한 터미널로 보일 때 색상 활성화** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), `NO_COLOR` 준수
- **단축된 서브시스템 접두사**: 선행 `gateway/` + `channels/` 제거, 마지막 2개 세그먼트 유지 (예: `whatsapp/outbound`)
- **서브시스템별 서브 로거** (자동 접두사 + 구조화된 필드 `{ subsystem }`)
- **`logRaw()`**는 QR/UX 출력용 (접두사 없음, 포맷팅 없음)
- **콘솔 스타일** (예: `pretty | compact | json`)
- **콘솔 로그 레벨**은 파일 로그 레벨과 분리 (`logging.level`이 `debug`/`trace`로 설정되면 파일이 전체 세부 정보 유지)
- **WhatsApp 메시지 본문**은 `debug` 레벨로 로깅 (보려면 `--verbose` 사용)

이것은 기존 파일 로그를 안정적으로 유지하면서 대화형 출력을 더 스캔하기 쉽게 만듭니다.
