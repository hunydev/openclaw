---
read_when:
  - 런타임 문제 또는 장애 해결 시
summary: 일반적인 OpenClaw 장애에 대한 빠른 문제 해결 가이드
title: 문제 해결
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: a07bb06f0b5ef56872578aaff6ac83adb740e2f1d23e3eed86934b51f62a877e
  source_path: gateway/troubleshooting.md
  workflow: 14
---

# 문제 해결 🔧

OpenClaw에 이상이 생겼을 때 해결 방법입니다.

빠른 진단만 원한다면 먼저 일반적인 문제의 [처음 60초](/help/faq#first-60-seconds-if-somethings-broken)를 확인하세요. 이 페이지는 런타임 장애와 진단 방법을 자세히 다룹니다.

프로바이더 관련 바로가기: [/channels/troubleshooting](/channels/troubleshooting)

## 상태 및 진단

빠른 진단 명령 (순서대로 실행):

| 명령                               | 알려주는 것                                                                           | 사용 시기                         |
| ---------------------------------- | ------------------------------------------------------------------------------------- | --------------------------------- |
| `openclaw status`                  | 로컬 요약: OS + 업데이트, Gateway 도달 가능성/모드, 서비스, 에이전트/세션, 프로바이더 구성 상태 | 첫 검사, 빠른 개요                |
| `openclaw status --all`            | 전체 로컬 진단 (읽기 전용, 붙여넣기 가능, 기본 안전) 로그 끝 포함                      | 디버그 보고서 공유 필요 시        |
| `openclaw status --deep`           | Gateway 상태 검사 실행 (프로바이더 프로브 포함; Gateway 도달 가능 필요)                 | "구성됨"이 "작동함"이 아닐 때     |
| `openclaw gateway probe`           | Gateway 발견 + 도달 가능성 (로컬 + 원격 대상)                                          | 잘못된 Gateway를 프로브하는 것 같을 때 |
| `openclaw channels status --probe` | 실행 중인 Gateway에 채널 상태 쿼리 (선택적 프로브)                                     | Gateway 도달 가능하지만 채널 이상 시 |
| `openclaw gateway status`          | 관리자 상태 (launchd/systemd/schtasks), 런타임 PID/종료 코드, 마지막 Gateway 오류      | 서비스가 "로드된 것처럼 보이지만" 실제로 실행 안 될 때 |
| `openclaw logs --follow`           | 라이브 로그 (런타임 문제의 최고의 신호 소스)                                           | 실제 실패 원인 확인 필요 시       |

**출력 공유:** `openclaw status --all` 선호 (토큰이 마스킹됨). `openclaw status` 출력을 붙여넣는다면 먼저 `OPENCLAW_SHOW_SECRETS=0` 설정 권장 (토큰 미리보기).

참조: [상태 검사](/gateway/health) 및 [로깅](/logging).

## 일반적인 문제

### No API key found for provider "anthropic"

**에이전트의 인증 저장소가 비어 있거나** Anthropic 자격 증명이 누락되었음을 의미합니다.
인증은 **에이전트별로 격리**되므로 새 에이전트는 메인 에이전트의 키를 상속받지 않습니다.

수정 옵션:

- 해당 에이전트에 대해 **Anthropic**을 선택하여 온보딩을 다시 실행.
- 또는 **Gateway 호스트**에서 setup-token 붙여넣기:
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
- 또는 메인 에이전트 디렉토리의 `auth-profiles.json`을 새 에이전트 디렉토리로 복사.

확인:

```bash
openclaw models status
```

### OAuth token refresh failed (Anthropic Claude subscription)

저장된 Anthropic OAuth 토큰이 만료되었고 새로 고침이 실패했음을 의미합니다.
Claude 구독 (API 키 없음)을 사용하는 경우 가장 신뢰할 수 있는 수정은
**Claude Code setup-token**으로 전환하여 **Gateway 호스트**에서 붙여넣는 것입니다.

**권장 방법 (setup-token):**

```bash
# Gateway 호스트에서 실행 (setup-token 붙여넣기)
openclaw models auth setup-token --provider anthropic
openclaw models status
```

다른 곳에서 토큰을 생성한 경우:

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

자세한 내용: [Anthropic](/providers/anthropic) 및 [OAuth](/concepts/oauth).

### 제어 UI가 HTTP에서 실패 ("device identity required" / "connect failed")

대시보드를 일반 HTTP로 열면 (예: `http://<lan-ip>:18789/` 또는
`http://<tailscale-ip>:18789/`) 브라우저가 **비보안 컨텍스트**에서 실행되어
WebCrypto가 차단되고 장치 ID 생성이 불가능합니다.

**수정:**

- [Tailscale Serve](/gateway/tailscale)를 통한 HTTPS 선호.
- 또는 Gateway 호스트에서 로컬로 열기: `http://127.0.0.1:18789/`.
- HTTP를 꼭 사용해야 한다면 `gateway.controlUi.allowInsecureAuth: true` 활성화하고
  Gateway 토큰 사용 (토큰만; 장치 ID/페어링 없음). [제어 UI](/web/control-ui#insecure-http) 참조.

### CI 비밀 스캔 실패

`detect-secrets`가 아직 베이스라인에 포함되지 않은 새 후보를 발견했음을 의미합니다.
[비밀 스캔](/gateway/security#secret-scanning-detect-secrets) 참조.

### 서비스 설치됨 but 실행 안 됨

Gateway 서비스가 설치되었지만 프로세스가 즉시 종료되면 서비스는
"로드됨"으로 표시되지만 실제로 실행 중인 프로세스가 없습니다.

**확인:**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/service는 런타임 상태 (PID/마지막 종료 코드)와 로그 힌트를 표시합니다.

**로그:**

- 권장: `openclaw logs --follow`
- 파일 로그 (항상 사용 가능): `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (또는 구성된 `logging.file`)
- macOS LaunchAgent (설치된 경우): `$OPENCLAW_STATE_DIR/logs/gateway.log` 및 `gateway.err.log`
- Linux systemd (설치된 경우): `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**더 자세한 로그 활성화:**

- 파일 로그 상세도 증가 (영구 JSONL):
  ```json
  { "logging": { "level": "debug" } }
  ```
- 콘솔 로그 상세도 증가 (TTY 출력만):
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
- 참고: `--verbose`는 **콘솔** 출력에만 영향. 파일 로그는 여전히 `logging.level`로 제어.

형식, 구성 및 액세스의 전체 개요는 [/logging](/logging) 참조.

### "Gateway start blocked: set gateway.mode=local"

구성 파일이 존재하지만 `gateway.mode`가 설정되지 않았거나 `local`이 아니어서
Gateway가 시작을 거부합니다.

**수정 (권장):**

- 마법사를 실행하고 Gateway 실행 모드를 **Local**로 설정:
  ```bash
  openclaw configure
  ```
- 또는 직접 설정:
  ```bash
  openclaw config set gateway.mode local
  ```

**원격 Gateway를 실행하려는 경우:**

- 원격 URL을 설정하고 `gateway.mode=remote` 유지:
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**임시/개발 전용:** `--allow-unconfigured`를 전달하여
`gateway.mode=local` 설정 없이 Gateway 시작.

**아직 구성 파일이 없나요?** `openclaw setup`을 실행하여 초기 구성을 만든 다음
Gateway를 다시 실행하세요.

### 서비스 환경 (PATH + 런타임)

Gateway 서비스는 셸/관리자 간섭을 피하기 위해 **최소 PATH**로 실행:

- macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
- Linux: `/usr/local/bin`, `/usr/bin`, `/bin`

이는 의도적으로 버전 관리자 (nvm/fnm/volta/asdf)와 패키지
관리자 (pnpm/npm)를 제외합니다 (서비스가 셸 초기화 스크립트를 로드하지 않기 때문).
`DISPLAY`와 같은 런타임 변수는 `~/.openclaw/.env`에 넣어야 합니다 (Gateway가 시작 초기에 로드).
`host=gateway`에서의 Exec 실행은 로그인 셸 `PATH`를 실행 환경에 병합하므로
도구 누락은 일반적으로 셸 초기화 스크립트가 내보내지 않았거나
`tools.exec.pathPrepend`를 설정해야 함을 의미합니다. [/tools/exec](/tools/exec) 참조.

WhatsApp + Telegram 채널은 **Node** 필요; Bun 미지원. 서비스가
Bun 또는 버전 관리자 관리 Node 경로로 설치되었다면 `openclaw doctor`를 실행하여
시스템 수준 Node 설치로 마이그레이션하세요.

### 샌드박스에서 Skills API 키 누락

**증상:** Skills가 호스트에서는 작동하지만 샌드박스에서 API 키 누락으로 실패.

**원인:** 샌드박스 격리 exec는 Docker에서 실행되며 호스트의 `process.env`를 **상속받지 않음**.

**수정:**

- `agents.defaults.sandbox.docker.env` 설정 (또는 에이전트별 `agents.list[].sandbox.docker.env`)
- 또는 커스텀 샌드박스 이미지에 키를 내장
- 그런 다음 `openclaw sandbox recreate --agent <id>` (또는 `--all`) 실행

### 서비스 실행 중이지만 포트 리스닝 안 함

서비스가 **실행 중**으로 보고되지만 Gateway 포트에 리스너가 없으면
Gateway가 바인딩을 거부했을 가능성이 높습니다.

**여기서 "실행 중"의 의미**

- `Runtime: running`은 관리자 (launchd/systemd/schtasks)가 프로세스가 활성화되어 있다고 생각함을 의미.
- `RPC probe`는 CLI가 실제로 Gateway WebSocket에 연결하여 `status`를 호출할 수 있었음을 의미.
- 항상 `Probe target:` + `Config (service):`를 "실제로 무엇을 시도했나?"의 근거로 사용.

**확인:**

- `gateway.mode`가 `openclaw gateway`와 서비스 모두에 대해 `local`이어야 함.
- `gateway.mode=remote`로 설정했다면 **CLI 기본값**이 원격 URL을 사용합니다. 서비스는 여전히 로컬에서 실행 중일 수 있지만 CLI가 잘못된 위치를 프로브할 수 있음. `openclaw gateway status`를 사용하여 서비스가 해석한 포트 + 프로브 대상 확인 (또는 `--url` 전달).
- `openclaw gateway status`와 `openclaw doctor`는 서비스가 실행 중인 것처럼 보이지만 포트가 열리지 않은 경우 **마지막 Gateway 오류** 로그를 표시.
- 비-local loopback 바인딩 (`lan`/`tailnet`/`custom`, 또는 local loopback 불가용 시 `auto`)은 인증 필요:
  `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`).
- `gateway.remote.token`은 원격 CLI 호출에만 사용; 로컬 인증을 활성화하지 **않음**.
- `gateway.token`은 무시됨; `gateway.auth.token` 사용.

**`openclaw gateway status`가 구성 불일치 표시 시**

- `Config (cli): ...`와 `Config (service): ...`는 일반적으로 일치해야 함.
- 일치하지 않으면 하나의 구성을 편집하면서 서비스가 다른 구성을 실행 중일 가능성이 높음.
- 수정: 서비스가 사용하길 원하는 동일한 `--profile` / `OPENCLAW_STATE_DIR`에서 `openclaw gateway install --force` 재실행.

**`openclaw gateway status`가 서비스 구성 문제 보고 시**

- 관리자 구성 (launchd/systemd/schtasks)에 현재 기본값 누락.
- 수정: `openclaw doctor`를 실행하여 구성 업데이트 (또는 `openclaw gateway install --force`로 완전 다시 쓰기).

**`Last gateway error:`가 "refusing to bind … without auth" 언급 시**

- `gateway.bind`를 비-local loopback 모드 (`lan`/`tailnet`/`custom`, 또는 local loopback 불가용 시 `auto`)로 설정했지만 인증을 구성하지 않음.
- 수정: `gateway.auth.mode` + `gateway.auth.token` 설정 (또는 `OPENCLAW_GATEWAY_TOKEN` 내보내기) 후 서비스 재시작.

**`openclaw gateway status`가 `bind=tailnet`이지만 tailnet 인터페이스 없음 표시 시**

- Gateway가 Tailscale IP (100.64.0.0/10)에 바인딩하려 했지만 호스트에서 감지되지 않음.
- 수정: 해당 컴퓨터에서 Tailscale 시작 (또는 `gateway.bind`를 `loopback`/`lan`으로 변경).

**`Probe note:`가 local loopback으로 프로브 표시 시**

- `bind=lan`의 경우 예상됨: Gateway가 `0.0.0.0` (모든 인터페이스)에서 리스닝, local loopback은 여전히 로컬에서 연결 가능.
- 원격 클라이언트의 경우 실제 LAN IP (`0.0.0.0` 아님)와 포트를 사용하고 인증이 구성되었는지 확인.

### 주소 이미 사용 중 (포트 18789)

프로세스가 이미 Gateway 포트에서 리스닝 중임을 의미합니다.

**확인:**

```bash
openclaw gateway status
```

리스너와 가능한 원인 (Gateway 이미 실행 중, SSH 터널)을 표시합니다.
필요하면 서비스를 중지하거나 다른 포트를 선택하세요.

### 추가 작업 공간 폴더 감지

이전 버전에서 업그레이드한 경우 디스크에 `~/openclaw`가 여전히 있을 수 있습니다.
여러 작업 공간 디렉토리는 하나의 작업 공간만 활성화되므로
인증 또는 상태 드리프트에 대한 혼란을 야기할 수 있습니다.

**수정:** 하나의 활성 작업 공간을 유지하고 나머지는 아카이브/삭제. 
[에이전트 작업 공간](/concepts/agent-workspace#extra-workspace-folders) 참조.

### 메인 채팅이 샌드박스 작업 공간에서 실행됨

증상: `pwd` 또는 파일 도구가 `~/.openclaw/sandboxes/...`를 표시하지만
호스트 작업 공간을 기대함.

**원인:** `agents.defaults.sandbox.mode: "non-main"`은 `session.mainKey` (기본값 `"main"`)를 기준으로 함.
그룹/채널 세션은 자체 키를 사용하므로 비-메인으로 취급되어
샌드박스 작업 공간을 얻습니다.

**수정 옵션:**

- 특정 에이전트가 호스트 작업 공간을 사용하길 원하면: `agents.list[].sandbox.mode: "off"` 설정.
- 샌드박스 내에서 호스트 작업 공간에 액세스하고 싶으면: 해당 에이전트에 `workspaceAccess: "rw"` 설정.

### "Agent was aborted"

에이전트가 응답 중에 중단되었습니다.

**원인:**

- 사용자가 `stop`, `abort`, `esc`, `wait` 또는 `exit` 전송
- 타임아웃
- 프로세스 크래시

**수정:** 다른 메시지를 보내세요. 세션은 계속됩니다.

### "Agent failed before reply: Unknown model: anthropic/claude-haiku-3-5"

OpenClaw는 의도적으로 **레거시/안전하지 않은 모델** (특히 프롬프트 인젝션에
더 취약한 모델)을 거부합니다. 이 오류가 표시되면 해당 모델 이름이
더 이상 지원되지 않습니다.

**수정:**

- 해당 프로바이더의 **최신** 모델을 선택하고 구성 또는 모델 별칭을 업데이트.
- 사용 가능한 모델이 무엇인지 확실하지 않으면 `openclaw models list` 또는
  `openclaw models scan`을 실행하고 지원되는 모델을 선택.
- 자세한 실패 원인은 Gateway 로그 확인.

참조: [모델 CLI](/cli/models) 및 [모델 프로바이더](/concepts/model-providers).

### 메시지가 트리거되지 않음

**검사 1:** 발신자가 허용 목록에 있나요?

```bash
openclaw status
```

출력에서 `AllowFrom: ...`을 찾으세요.

**검사 2:** 그룹 채팅의 경우 멘션이 필요한가요?

```bash
# 메시지가 mentionPatterns와 일치하거나 명시적으로 멘션해야 함; 기본값은 채널 groups/guilds에 있음.
# 멀티 에이전트: `agents.list[].groupChat.mentionPatterns`가 전역 패턴을 재정의.
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**검사 3:** 로그 확인

```bash
openclaw logs --follow
# 또는 빠른 필터:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

### 페어링 코드가 전달되지 않음

`dmPolicy`가 `pairing`이면 알 수 없는 발신자는 확인 코드를 받아야 하며
승인 전까지 메시지가 무시됩니다.

**검사 1:** 이미 대기 중인 요청이 있나요?

```bash
openclaw pairing list <channel>
```

각 채널은 기본적으로 최대 **3개**의 대기 중인 DM 페어링 요청을 허용합니다. 목록이 가득 차면 새 요청은 하나가 승인되거나 만료될 때까지 코드를 생성하지 않습니다.

**검사 2:** 요청이 생성되었지만 회신이 전송되지 않았나요?

```bash
openclaw logs --follow | grep "pairing request"
```

**검사 3:** 해당 채널의 `dmPolicy`가 `open`/`allowlist`가 아닌지 확인.

### 이미지 + 멘션이 작동하지 않음

알려진 문제: 이미지를 보내고 **멘션만** 포함 (다른 텍스트 없음)하면
WhatsApp이 때때로 멘션 메타데이터를 포함하지 않습니다.

**해결 방법:** 멘션과 함께 텍스트 추가:

- ❌ `@openclaw` + 이미지
- ✅ `@openclaw check this` + 이미지

### 세션이 복원되지 않음

**검사 1:** 세션 파일이 존재하나요?

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**검사 2:** 리셋 창이 너무 짧나요?

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080 // 7일
    }
  }
}
```

**검사 3:** 누군가 `/new`, `/reset` 또는 리셋 트리거를 보냈나요?

### 에이전트 타임아웃

기본 타임아웃은 30분입니다. 장시간 작업의 경우:

```json
{
  "reply": {
    "timeoutSeconds": 3600 // 1시간
  }
}
```
