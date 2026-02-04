---
read_when:
  - Gateway 프로세스 실행 또는 디버깅 시
summary: Gateway 서비스 운영 매뉴얼, 라이프사이클 및 운영 가이드
title: Gateway 운영 매뉴얼
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 497d58090faaa6bdae62780ce887b40a1ad81e2e99ff186ea2a5c2249c35d9ba
  source_path: gateway/index.md
  workflow: 14
---

# Gateway 서비스 운영 매뉴얼

최종 업데이트: 2025-12-09

## 소개

- 항상 실행되는 프로세스로, 고유한 Baileys/Telegram 연결과 제어/이벤트 플레인을 보유합니다.
- 레거시 `gateway` 명령을 대체합니다. CLI 진입점: `openclaw gateway`.
- 중지될 때까지 지속 실행; 치명적 오류 시 0이 아닌 상태로 종료되어 supervisor가 재시작합니다.

## 실행 방법 (로컬)

```bash
openclaw gateway --port 18789
# stdout에서 전체 debug/trace 로그 얻기:
openclaw gateway --port 18789 --verbose
# 포트가 사용 중이면 리스닝 프로세스를 종료 후 시작:
openclaw gateway --force
# 개발 루프 (TS 파일 변경 시 자동 리로드):
pnpm gateway:watch
```

- 구성 핫 리로드는 `~/.openclaw/openclaw.json` (또는 `OPENCLAW_CONFIG_PATH`)을 감시합니다.
  - 기본 모드: `gateway.reload.mode="hybrid"` (안전한 변경은 핫 적용, 중요 변경은 재시작).
  - 핫 리로드는 필요시 **SIGUSR1**로 프로세스 내 재시작을 수행합니다.
  - `gateway.reload.mode="off"`로 비활성화.
- WebSocket 제어 플레인은 `127.0.0.1:<port>` (기본값 18789)에 바인딩됩니다.
- 동일 포트에서 HTTP도 제공 (제어 UI, 훅, A2UI). 단일 포트 다중화.
  - OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/gateway/openai-http-api).
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Tools Invoke (HTTP): [`/tools/invoke`](/gateway/tools-invoke-http-api).
- 기본적으로 `canvasHost.port` (기본값 `18793`)에서 Canvas 파일 서버 시작, `~/.openclaw/workspace/canvas`에서 `http://<gateway-host>:18793/__openclaw__/canvas/` 제공. `canvasHost.enabled=false` 또는 `OPENCLAW_SKIP_CANVAS_HOST=1`로 비활성화.
- 로그는 stdout으로 출력; launchd/systemd로 프로세스 유지 및 로그 로테이션.
- 문제 해결 시 `--verbose`를 전달하여 디버그 로그 (핸드셰이크, 요청/응답, 이벤트)를 로그 파일에서 stdout으로 미러링.
- `--force`는 `lsof`로 선택된 포트의 리스닝 프로세스를 찾아 SIGTERM을 보내고, 종료된 프로세스를 기록한 다음 Gateway를 시작 (`lsof`가 없으면 빠르게 실패).
- supervisor (launchd/systemd/mac app 자식 프로세스 모드) 하에서 실행 시, 중지/재시작은 일반적으로 **SIGTERM**을 보냄; 이전 버전에서는 `pnpm` `ELIFECYCLE` 종료 코드 **143** (SIGTERM)으로 표시될 수 있으며, 이는 정상 종료이지 크래시가 아닙니다.
- **SIGUSR1**은 인가된 경우 프로세스 내 재시작을 트리거 (Gateway 도구/구성 적용/업데이트, 또는 수동 재시작을 위해 `commands.restart` 활성화 시).
- 기본적으로 Gateway 인증 필요: `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`) 또는 `gateway.auth.password` 설정. 클라이언트는 Tailscale Serve ID를 사용하지 않는 한 `connect.params.auth.token/password`를 보내야 함.
- 마법사는 이제 로컬 loopback에서도 기본적으로 토큰을 생성합니다.
- 포트 우선순위: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > 기본값 `18789`.

## 원격 액세스

- Tailscale/VPN 권장; 그렇지 않으면 SSH 터널 사용:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
- 클라이언트는 터널을 통해 `ws://127.0.0.1:18789`에 연결.
- 토큰이 구성된 경우 클라이언트는 터널을 통해서도 `connect.params.auth.token`에 토큰을 포함해야 함.

## 여러 Gateway (동일 호스트)

일반적으로 불필요: 하나의 Gateway가 여러 메시징 채널과 에이전트를 서비스할 수 있습니다. 중복성이나 엄격한 격리 (예: 구조 봇)가 필요한 경우에만 여러 Gateway를 사용하세요.

상태 + 구성을 격리하고 고유 포트를 사용하면 다중 인스턴스가 지원됩니다. 전체 가이드: [다중 Gateway](/gateway/multiple-gateways).

서비스 이름은 프로필을 인식합니다:

- macOS: `bot.molt.<profile>` (레거시 `com.openclaw.*`가 여전히 존재할 수 있음)
- Linux: `openclaw-gateway-<profile>.service`
- Windows: `OpenClaw Gateway (<profile>)`

설치 메타데이터는 서비스 구성에 포함됩니다:

- `OPENCLAW_SERVICE_MARKER=openclaw`
- `OPENCLAW_SERVICE_KIND=gateway`
- `OPENCLAW_SERVICE_VERSION=<version>`

구조 봇 모드: 두 번째 Gateway를 격리된 상태로 유지하고 별도의 프로필, 상태 디렉토리, 작업 공간 및 기본 포트 간격을 사용합니다. 전체 가이드: [구조 봇 가이드](/gateway/multiple-gateways#rescue-bot-guide).

### 개발 프로필 (`--dev`)

빠른 경로: 주요 설정에 영향을 주지 않고 완전히 격리된 개발 인스턴스 (구성/상태/작업 공간) 실행.

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# 그런 다음 개발 인스턴스를 가리킴:
openclaw --dev status
openclaw --dev health
```

기본값 (환경 변수/플래그/구성으로 재정의 가능):

- `OPENCLAW_STATE_DIR=~/.openclaw-dev`
- `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
- `OPENCLAW_GATEWAY_PORT=19001` (Gateway WS + HTTP)
- 브라우저 제어 서비스 포트 = `19003` (파생: `gateway.port+2`, 로컬 loopback 전용)
- `canvasHost.port=19005` (파생: `gateway.port+4`)
- `--dev`에서 `setup`/`onboard` 실행 시 `agents.defaults.workspace`가 기본적으로 `~/.openclaw/workspace-dev`가 됨.

파생 포트 (경험 법칙):

- 기본 포트 = `gateway.port` (또는 `OPENCLAW_GATEWAY_PORT` / `--port`)
- 브라우저 제어 서비스 포트 = 기본 포트 + 2 (로컬 loopback 전용)
- `canvasHost.port = 기본 포트 + 4` (또는 `OPENCLAW_CANVAS_HOST_PORT` / 구성 재정의)
- 브라우저 프로필 CDP 포트는 `browser.controlPort + 9 .. + 108`에서 자동 할당 (프로필별 영구화).

인스턴스별 체크리스트:

- 고유한 `gateway.port`
- 고유한 `OPENCLAW_CONFIG_PATH`
- 고유한 `OPENCLAW_STATE_DIR`
- 고유한 `agents.defaults.workspace`
- 별도의 WhatsApp 번호 (WA 사용 시)

프로필별 서비스 설치:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

예시:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

## 프로토콜 (운영 관점)

- 전체 문서: [Gateway 프로토콜](/gateway/protocol) 및 [Bridge 프로토콜 (레거시)](/gateway/bridge-protocol).
- 클라이언트가 보내야 하는 첫 프레임: `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`.
- Gateway 응답 `res {type:"res", id, ok:true, payload:hello-ok }` (또는 `ok:false`와 오류 후 연결 종료).
- 핸드셰이크 후:
  - 요청: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  - 이벤트: `{type:"event", event, payload, seq?, stateVersion?}`
- 구조화된 프레즌스 항목: `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }` (WS 클라이언트의 경우 `instanceId`는 `connect.client.instanceId`에서 가져옴).
- `agent` 응답은 두 단계: 먼저 `res` 확인 `{runId,status:"accepted"}`, 그런 다음 실행 완료 후 최종 `res` `{runId,status:"ok"|"error",summary}`; 스트리밍 출력은 `event:"agent"`로 도착.

## 메서드 (초기 세트)

- `health` — 전체 상태 스냅샷 (`openclaw health --json`과 동일한 형태).
- `status` — 간략한 요약.
- `system-presence` — 현재 프레즌스 목록.
- `system-event` — 프레즌스/시스템 알림 발행 (구조화됨).
- `send` — 활성 채널을 통해 메시지 전송.
- `agent` — 에이전트 턴 실행 (동일 연결에서 이벤트 스트리밍).
- `node.list` — 페어링된 + 현재 연결된 노드 나열 (`caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected` 및 광고된 `commands` 포함).
- `node.describe` — 노드 설명 (능력 + 지원되는 `node.invoke` 명령; 페어링된 노드와 현재 연결된 미페어링 노드에 적용).
- `node.invoke` — 노드에서 명령 호출 (예: `canvas.*`, `camera.*`).
- `node.pair.*` — 페어링 라이프사이클 (`request`, `list`, `approve`, `reject`, `verify`).

참조: [프레즌스](/concepts/presence)에서 프레즌스 생성/중복 제거 방식과 안정적인 `client.instanceId`가 중요한 이유를 확인하세요.

## 이벤트

- `agent` — 에이전트 실행에서 스트리밍되는 도구/출력 이벤트 (seq 태그 포함).
- `presence` — 모든 연결된 클라이언트에 푸시되는 프레즌스 업데이트 (stateVersion 포함 델타).
- `tick` — 주기적 하트비트/noop, 활성 상태 확인.
- `shutdown` — Gateway가 종료 중; 페이로드에 `reason`과 선택적 `restartExpectedMs` 포함. 클라이언트는 재연결해야 함.

## WebChat 통합

- WebChat은 네이티브 SwiftUI UI로, Gateway WebSocket을 통해 직접 히스토리, 전송, 중단 및 이벤트 상호작용.
- 원격 사용은 동일한 SSH/Tailscale 터널을 통해; Gateway 토큰이 구성된 경우 클라이언트는 `connect` 시 이를 포함.
- macOS 앱은 단일 WS (공유 연결)로 연결; 초기 스냅샷에서 프레즌스를 얻고 `presence` 이벤트를 수신하여 UI 업데이트.

## 타입 및 검증

- 서버는 프로토콜 정의에서 생성된 JSON Schema에 대해 AJV로 각 인바운드 프레임 검증.
- 클라이언트 (TS/Swift)는 생성된 타입 사용 (TS 직접; Swift는 저장소 생성기 통해).
- 프로토콜 정의가 유일한 진실의 원천; 다음으로 schema/모델 재생성:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`

## 연결 스냅샷

- `hello-ok`는 `presence`, `health`, `stateVersion`, `uptimeMs`가 포함된 `snapshot`과 `policy {maxPayload,maxBufferedBytes,tickIntervalMs}`를 포함하여 클라이언트가 추가 요청 없이 즉시 렌더링할 수 있음.
- `health`/`system-presence`는 수동 새로고침에 여전히 사용 가능하지만 연결 시 필수 아님.

## 오류 코드 (res.error 구조)

- 오류는 `{ code, message, details?, retryable?, retryAfterMs? }` 형식 사용.
- 표준 오류 코드:
  - `NOT_LINKED` — WhatsApp 인증되지 않음.
  - `AGENT_TIMEOUT` — 에이전트가 구성된 기한 내에 응답하지 않음.
  - `INVALID_REQUEST` — schema/매개변수 검증 실패.
  - `UNAVAILABLE` — Gateway가 종료 중이거나 의존성 불가용.

## 하트비트 동작

- `tick` 이벤트 (또는 WS ping/pong)가 주기적으로 발생하여 트래픽이 없어도 클라이언트가 Gateway가 활성 상태임을 알 수 있음.
- 전송/에이전트 확인은 여전히 독립 응답; tick을 전송에 사용하지 마세요.

## 리플레이/갭

- 이벤트는 리플레이되지 않음. 클라이언트가 seq 갭을 감지하면 계속하기 전에 새로고침 (`health` + `system-presence`). WebChat과 macOS 클라이언트는 이제 갭 감지 시 자동 새로고침.

## 프로세스 감독 (macOS 예시)

- launchd로 서비스 유지:
  - Program: `openclaw` 경로
  - Arguments: `gateway`
  - KeepAlive: true
  - StandardOut/Err: 파일 경로 또는 `syslog`
- 실패 시 launchd가 재시작; 치명적 구성 오류는 지속적으로 종료되어 운영자가 인지할 수 있도록 함.
- LaunchAgents는 사용자별이며 로그인된 세션 필요; 헤드리스 설정에는 커스텀 LaunchDaemon 사용 (포함되지 않음).
  - `openclaw gateway install`은 `~/Library/LaunchAgents/bot.molt.gateway.plist` 기록
    (또는 `bot.molt.<profile>.plist`; 레거시 `com.openclaw.*`는 정리됨).
  - `openclaw doctor`는 LaunchAgent 구성을 감사하고 현재 권장 기본값으로 업데이트할 수 있음.

## Gateway 서비스 관리 (CLI)

Gateway CLI로 설치/시작/중지/재시작/상태 조회:

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

참고:

- `gateway status`는 기본적으로 서비스가 해석한 포트/구성으로 Gateway RPC 프로브 (`--url`로 재정의 가능).
- `gateway status --deep`은 시스템 수준 스캔 추가 (LaunchDaemons/시스템 유닛).
- `gateway status --no-probe`는 RPC 프로브 건너뛰기 (네트워크 불가용 시 유용).
- `gateway status --json` 출력은 스크립트에 적합한 안정적 형식.
- `gateway status`는 **supervisor 런타임** (launchd/systemd 실행 중)과 **RPC 도달 가능성** (WS 연결 + 상태 RPC)을 별도로 보고.
- `gateway status`는 구성 경로 + 프로브 대상을 출력하여 "localhost vs LAN 바인딩" 혼동과 프로필 불일치 방지.
- `gateway status`는 서비스가 실행 중인 것처럼 보이지만 포트가 다운된 경우 마지막 Gateway 오류 줄 포함.
- `logs`는 RPC를 통해 Gateway 파일 로그 추적 (수동 `tail`/`grep` 불필요).
- 다른 Gateway 유사 서비스가 감지되면 CLI가 경고, 단 OpenClaw 프로필 서비스는 제외.
  대부분의 시나리오에서는 여전히 **컴퓨터당 하나의 Gateway** 권장; 중복성이나 구조 봇에는 격리된 프로필/포트 사용. [다중 Gateway](/gateway/multiple-gateways) 참조.
  - 정리: `openclaw gateway uninstall` (현재 서비스) 및 `openclaw doctor` (레거시 마이그레이션).
- `gateway install`은 이미 설치된 경우 noop; 재설치 (프로필/환경/경로 변경)에는 `openclaw gateway install --force` 사용.

번들된 Mac 앱:

- OpenClaw.app은 Node 기반 Gateway 릴레이를 번들하고 사용자별 LaunchAgent 설치 가능, 레이블
  `bot.molt.gateway` (또는 `bot.molt.<profile>`; 레거시 `com.openclaw.*` 레이블은 여전히 정상 제거됨).
- 정상 중지하려면 `openclaw gateway stop` 사용 (또는 `launchctl bootout gui/$UID/bot.molt.gateway`).
- 재시작하려면 `openclaw gateway restart` 사용 (또는 `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
  - `launchctl`은 LaunchAgent가 설치된 경우에만 작동; 그렇지 않으면 먼저 `openclaw gateway install` 사용.
  - 명명된 프로필 실행 시 레이블을 `bot.molt.<profile>`로 교체.

## 프로세스 감독 (systemd 사용자 유닛)

OpenClaw는 Linux/WSL2에서 기본적으로 **systemd 사용자 서비스**를 설치합니다. 단일 사용자 컴퓨터에는 사용자 서비스 권장 (더 간단한 환경, 사용자별 구성). 다중 사용자 또는 항상 켜진 서버에는 **시스템 서비스** 사용 (lingering 불필요, 공유 감독).

`openclaw gateway install`은 사용자 유닛을 작성합니다. `openclaw doctor`는 해당 유닛을 감사하고 현재 권장 기본값으로 업데이트할 수 있습니다.

`~/.config/systemd/user/openclaw-gateway[-<profile>].service` 생성:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

lingering 활성화 (필수, 사용자 서비스가 로그아웃/유휴 후에도 계속 실행되도록):

```
sudo loginctl enable-linger youruser
```

온보딩은 Linux/WSL2에서 이 명령을 실행합니다 (sudo 비밀번호 입력 요청 가능; `/var/lib/systemd/linger`에 기록).
그런 다음 서비스 활성화:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**대안 (시스템 서비스)** - 항상 켜진 또는 다중 사용자 서버의 경우 사용자 유닛 대신 systemd **시스템** 유닛 설치 가능 (lingering 불필요). `/etc/systemd/system/openclaw-gateway[-<profile>].service` 생성 (위 유닛 복사, `WantedBy=multi-user.target`, `User=` + `WorkingDirectory=` 설정), 그런 다음:

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

## Windows (WSL2)

Windows 설치는 **WSL2**를 사용하고 위의 Linux systemd 섹션을 따라야 합니다.

## 운영 검사

- 활성 상태: WS를 열고 `req:connect` 전송 → `payload.type="hello-ok"` (스냅샷 포함)가 있는 `res` 기대.
- 준비 상태: `health` 호출 → `ok: true`와 `linkChannel`에 연결된 채널 (해당되는 경우) 기대.
- 디버그: `tick`과 `presence` 이벤트 구독; `status`가 연결/인증 시간 표시 확인; 프레즌스 항목이 Gateway 호스트와 연결된 클라이언트 표시.

## 보안 보장

- 기본적으로 호스트당 하나의 Gateway 가정; 여러 프로필 실행 시 포트/상태를 격리하고 올바른 인스턴스를 가리키세요.
- 직접 Baileys 연결로 대체하지 않음; Gateway가 불가용하면 전송이 빠르게 실패.
- connect가 아닌 첫 프레임 또는 잘못된 JSON은 거부되고 소켓이 닫힘.
- 정상 종료: 닫기 전 `shutdown` 이벤트 발생; 클라이언트는 닫힘 + 재연결을 처리해야 함.

## CLI 헬퍼

- `openclaw gateway health|status` — Gateway WS를 통해 상태/건강 정보 요청.
- `openclaw message send --target <num> --message "hi" [--media ...]` — Gateway를 통해 전송 (WhatsApp에 대해 멱등성).
- `openclaw agent --message "hi" --to <num>` — 에이전트 턴 실행 (기본적으로 최종 결과 대기).
- `openclaw gateway call <method> --params '{"k":"v"}'` — 디버깅용 원시 메서드 호출.
- `openclaw gateway stop|restart` — 감독된 Gateway 서비스 중지/재시작 (launchd/systemd).
- Gateway 헬퍼 하위 명령은 Gateway가 `--url`에서 실행 중이라고 가정; 더 이상 자동으로 Gateway를 시작하지 않음.

## 마이그레이션 가이드

- `openclaw gateway` 및 레거시 TCP 제어 포트 사용 중단.
- 필수 connect와 구조화된 프레즌스가 있는 WS 프로토콜을 사용하도록 클라이언트 업데이트.
