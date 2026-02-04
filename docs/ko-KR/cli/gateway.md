---
read_when:
  - CLI에서 Gateway를 실행할 때(개발 또는 서버 환경)
  - Gateway 인증, 바인딩 모드 및 연결 문제를 디버깅할 때
  - Bonjour를 통해 Gateway를 검색할 때(LAN + tailnet)
summary: OpenClaw Gateway CLI(`openclaw gateway`) — Gateway 실행, 쿼리 및 검색
title: gateway
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# Gateway CLI

Gateway는 OpenClaw의 WebSocket 서버입니다(채널, 노드, 세션, 훅).

이 페이지의 하위 명령은 `openclaw gateway …` 아래에 있습니다.

관련 문서:

- [/gateway/bonjour](/gateway/bonjour)
- [/gateway/discovery](/gateway/discovery)
- [/gateway/configuration](/gateway/configuration)

## Gateway 실행

로컬 Gateway 프로세스 실행:

```bash
openclaw gateway
```

포그라운드 실행 별칭:

```bash
openclaw gateway run
```

참고 사항:

- 기본적으로 `~/.openclaw/openclaw.json`에 `gateway.mode=local`이 설정되지 않으면 Gateway가 시작을 거부합니다. 임시/개발 실행에는 `--allow-unconfigured`를 사용하세요.
- 인증 없이 로컬 루프백 외부에 바인딩하는 것은 차단됩니다(보안 보호 조치).
- 승인 후 `SIGUSR1`은 프로세스 내 재시작을 트리거합니다(`commands.restart` 활성화 또는 Gateway 도구/구성 적용/업데이트를 통해).
- `SIGINT`/`SIGTERM` 핸들러는 Gateway 프로세스를 중지하지만 사용자 정의 터미널 상태를 복원하지 않습니다. TUI 또는 원시 모드 입력으로 CLI를 래핑하는 경우 종료 전에 터미널을 복원하세요.

### 옵션

- `--port <port>`: WebSocket 포트(구성/환경 변수에서 기본값; 일반적으로 `18789`).
- `--bind <loopback|lan|tailnet|auto|custom>`: 리스너 바인딩 모드.
- `--auth <token|password>`: 인증 모드 재정의.
- `--token <token>`: 토큰 재정의(프로세스에 `OPENCLAW_GATEWAY_TOKEN`도 설정).
- `--password <password>`: 비밀번호 재정의(프로세스에 `OPENCLAW_GATEWAY_PASSWORD`도 설정).
- `--tailscale <off|serve|funnel>`: Tailscale을 통해 Gateway 노출.
- `--tailscale-reset-on-exit`: 종료 시 Tailscale serve/funnel 구성 재설정.
- `--allow-unconfigured`: 구성에 `gateway.mode=local`이 없어도 Gateway 시작 허용.
- `--dev`: 누락된 경우 개발 구성 및 작업 공간 생성(BOOTSTRAP.md 건너뛰기).
- `--reset`: 개발 구성 + 자격 증명 + 세션 + 작업 공간 재설정(`--dev` 필요).
- `--force`: 시작 전 선택한 포트의 기존 리스너 종료.
- `--verbose`: 상세 로그.
- `--claude-cli-logs`: 콘솔에 claude-cli 로그만 표시(및 stdout/stderr 활성화).
- `--ws-log <auto|full|compact>`: WebSocket 로그 스타일(기본값 `auto`).
- `--compact`: `--ws-log compact`의 별칭.
- `--raw-stream`: 원시 모델 스트림 이벤트를 jsonl로 기록.
- `--raw-stream-path <path>`: 원시 스트림 jsonl 경로.

## 실행 중인 Gateway 쿼리

모든 쿼리 명령은 WebSocket RPC를 사용합니다.

출력 모드:

- 기본값: 사람이 읽을 수 있음(TTY에서 색상 포함).
- `--json`: 기계 판독 가능한 JSON(스타일/로딩 애니메이션 없음).
- `--no-color`(또는 `NO_COLOR=1`): ANSI 비활성화하지만 사람이 읽을 수 있는 레이아웃 유지.

공유 옵션(지원되는 명령에서):

- `--url <url>`: Gateway WebSocket URL.
- `--token <token>`: Gateway 토큰.
- `--password <password>`: Gateway 비밀번호.
- `--timeout <ms>`: 타임아웃/예산(명령에 따라 다름).
- `--expect-final`: "최종" 응답 대기(에이전트 호출).

### `gateway health`

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### `gateway status`

`gateway status`는 Gateway 서비스(launchd/systemd/schtasks) 및 선택적 RPC 탐색을 표시합니다.

```bash
openclaw gateway status
openclaw gateway status --json
```

옵션:

- `--url <url>`: 탐색 URL 재정의.
- `--token <token>`: 탐색을 위한 토큰 인증.
- `--password <password>`: 탐색을 위한 비밀번호 인증.
- `--timeout <ms>`: 탐색 타임아웃(기본값 `10000`).
- `--no-probe`: RPC 탐색 건너뛰기(서비스 상태만 보기).
- `--deep`: 시스템 수준 서비스도 스캔.

### `gateway probe`

`gateway probe`는 "전체 디버그" 명령입니다. 항상 다음을 탐색합니다:

- 구성된 원격 Gateway(설정된 경우), 및
- localhost(로컬 루프백), **원격 Gateway가 구성되어 있어도**.

여러 Gateway에 도달할 수 있으면 모두 출력합니다. 격리된 프로필/포트(예: 구조 봇)를 사용할 때 여러 Gateway가 지원되지만, 대부분의 설치는 여전히 단일 Gateway를 실행합니다.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### SSH를 통한 원격 연결(Mac 앱 피어 모드)

macOS 앱의 "SSH를 통한 원격 연결" 모드는 로컬 포트 포워딩을 사용하여 원격 Gateway(로컬 루프백에만 바인딩되어 있을 수 있음)를 `ws://127.0.0.1:<port>`를 통해 액세스할 수 있게 합니다.

CLI 동등 명령:

```bash
openclaw gateway probe --ssh user@gateway-host
```

옵션:

- `--ssh <target>`: `user@host` 또는 `user@host:port`(포트 기본값 `22`).
- `--ssh-identity <path>`: 신원 파일.
- `--ssh-auto`: 검색된 첫 번째 Gateway 호스트를 SSH 대상으로 자동 선택(LAN/WAB 전용).

구성(선택 사항, 기본값으로 사용):

- `gateway.remote.sshTarget`
- `gateway.remote.sshIdentity`

### `gateway call <method>`

저수준 RPC 도우미.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gateway 서비스 관리

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

참고 사항:

- `gateway install`은 `--port`, `--runtime`, `--token`, `--force`, `--json`을 지원합니다.
- 수명 주기 명령은 스크립팅을 위해 `--json`을 허용합니다.

## Gateway 검색(Bonjour)

`gateway discover`는 Gateway 비콘(`_openclaw-gw._tcp`)을 스캔합니다.

- 멀티캐스트 DNS-SD: `local.`
- 유니캐스트 DNS-SD(광역 Bonjour): 도메인(예: `openclaw.internal.`)을 선택하고 분할 DNS + DNS 서버를 설정합니다; [/gateway/bonjour](/gateway/bonjour) 참조

Bonjour 검색이 활성화된(기본적으로 활성화) Gateway만 비콘을 브로드캐스트합니다.

광역 검색 레코드에는 다음이 포함됩니다(TXT):

- `role`(Gateway 역할 힌트)
- `transport`(전송 힌트, 예: `gateway`)
- `gatewayPort`(WebSocket 포트, 일반적으로 `18789`)
- `sshPort`(SSH 포트; 지정되지 않으면 기본값 `22`)
- `tailnetDns`(사용 가능한 경우 MagicDNS 호스트 이름)
- `gatewayTls` / `gatewayTlsSha256`(TLS 활성화 + 인증서 지문)
- `cliPath`(선택적 원격 설치 경로 힌트)

### `gateway discover`

```bash
openclaw gateway discover
```

옵션:

- `--timeout <ms>`: 명령당 타임아웃(브라우즈/해석); 기본값 `2000`.
- `--json`: 기계 판독 가능한 출력(스타일/로딩 애니메이션도 비활성화).

예시:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
