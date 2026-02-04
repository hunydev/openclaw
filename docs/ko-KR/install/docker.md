---
read_when:
  - 로컬 설치 대신 컨테이너화된 게이트웨이를 원할 때
  - Docker 플로우 검증 중
summary: 선택적 Docker 기반 OpenClaw 설정 및 온보딩
title: Docker
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 781dd01ca99a2101f622f03162eb1776079c3c444f54c5a054d6816ec203e2f2
  source_path: install/docker.md
  workflow: 14
---

# Docker (선택 사항)

Docker는 **선택 사항**입니다. 게이트웨이를 컨테이너화하거나 Docker 플로우를 검증해야 할 때만 사용하세요.

## Docker가 내게 맞을까?

- **맞음**: 격리된, 언제든 폐기 가능한 게이트웨이 환경이 필요하거나, 로컬 설치 없이 호스트에서 OpenClaw를 실행하고 싶을 때.
- **아님**: 자신의 컴퓨터에서 실행하며 가장 빠른 개발 사이클을 원할 때. 일반 설치 플로우를 사용하세요.
- **샌드박스 참고**: 에이전트 샌드박스도 Docker를 사용하지만, 전체 게이트웨이가 Docker에서 실행될 **필요는 없습니다**. 자세한 내용은 [샌드박스](/gateway/sandboxing)를 참조하세요.

이 가이드에서 다루는 내용:

- 컨테이너화된 게이트웨이 (전체 OpenClaw가 Docker에서 실행)
- 세션별 에이전트 샌드박스 (호스트 게이트웨이 + Docker 격리 에이전트 도구)

샌드박스 세부 정보: [샌드박스](/gateway/sandboxing)

## 사전 요구 사항

- Docker Desktop (또는 Docker Engine) + Docker Compose v2
- 이미지와 로그를 위한 충분한 디스크 공간

## 컨테이너화된 게이트웨이 (Docker Compose)

### 빠른 시작 (권장)

저장소 루트에서 실행:

```bash
./docker-setup.sh
```

이 스크립트는:

- 게이트웨이 이미지 빌드
- 온보딩 마법사 실행
- 선택적 프로바이더 설정 힌트 출력
- Docker Compose로 게이트웨이 시작
- 게이트웨이 토큰 생성 및 `.env`에 기록

선택적 환경 변수:

- `OPENCLAW_DOCKER_APT_PACKAGES` — 빌드 중 추가 apt 패키지 설치
- `OPENCLAW_EXTRA_MOUNTS` — 추가 호스트 바인드 마운트 추가
- `OPENCLAW_HOME_VOLUME` — `/home/node`를 명명된 볼륨에 영구화

완료 후:

- 브라우저에서 `http://127.0.0.1:18789/`를 여세요.
- 제어 UI에서 토큰을 붙여넣으세요 (설정 → 토큰).

구성과 작업 공간은 호스트에 기록됩니다:

- `~/.openclaw/`
- `~/.openclaw/workspace`

VPS에서 실행? [Hetzner (Docker VPS)](/platforms/hetzner) 참조.

### 수동 프로세스 (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

### 추가 마운트 (선택 사항)

추가 호스트 디렉토리를 컨테이너에 마운트하려면, `docker-setup.sh` 실행 전에 `OPENCLAW_EXTRA_MOUNTS`를 설정하세요. 이 변수는 쉼표로 구분된 Docker 바인드 마운트 목록을 받아 `docker-compose.extra.yml`을 생성하여 `openclaw-gateway`와 `openclaw-cli`에 적용합니다.

예시:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

참고:

- macOS/Windows에서는 경로가 Docker Desktop과 공유되어야 합니다.
- `OPENCLAW_EXTRA_MOUNTS`를 수정하면 `docker-setup.sh`를 다시 실행하여 추가 compose 파일을 재생성하세요.
- `docker-compose.extra.yml`은 자동 생성됩니다. 수동으로 편집하지 마세요.

### 전체 컨테이너 홈 디렉토리 영구화 (선택 사항)

컨테이너 재빌드 후에도 `/home/node`를 영구 보존하려면, `OPENCLAW_HOME_VOLUME`으로 명명된 볼륨을 설정하세요. Docker 볼륨을 생성하고 `/home/node`에 마운트하며, 표준 구성/작업 공간 바인드 마운트를 유지합니다. 여기에는 명명된 볼륨을 사용하세요 (바인드 경로가 아님); 바인드 마운트는 `OPENCLAW_EXTRA_MOUNTS`를 사용하세요.

예시:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

추가 마운트와 결합 가능:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

참고:

- `OPENCLAW_HOME_VOLUME`을 수정하면 `docker-setup.sh`를 다시 실행하여 추가 compose 파일을 재생성하세요.
- 명명된 볼륨은 `docker volume rm <name>`으로 삭제될 때까지 유지됩니다.

### 추가 apt 패키지 설치 (선택 사항)

이미지에 시스템 패키지 (예: 빌드 도구나 미디어 라이브러리)가 필요하면, `docker-setup.sh` 실행 전에 `OPENCLAW_DOCKER_APT_PACKAGES`를 설정하세요. 이미지 빌드 중 패키지를 설치하므로 컨테이너가 삭제되어도 유지됩니다.

예시:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

참고:

- 이 변수는 공백으로 구분된 apt 패키지 이름 목록을 받습니다.
- `OPENCLAW_DOCKER_APT_PACKAGES`를 수정하면 `docker-setup.sh`를 다시 실행하여 이미지를 재빌드하세요.

### 재빌드 속도 향상 (권장)

재빌드 속도를 높이려면 Dockerfile 순서를 조정하여 의존성 레이어 캐싱을 활용하세요. 이렇게 하면 lock 파일이 변경될 때만 `pnpm install`이 다시 실행됩니다:

```dockerfile
FROM node:22-bookworm

# Bun 설치 (빌드 스크립트에 필요)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# 패키지 메타데이터 변경 없으면 의존성 캐시
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### 채널 설정 (선택 사항)

CLI 컨테이너로 채널을 구성한 다음 필요에 따라 게이트웨이를 재시작하세요.

WhatsApp (QR 코드):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (봇 토큰):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (봇 토큰):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

문서: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord)

### 헬스 체크

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### 엔드투엔드 스모크 테스트 (Docker)

```bash
scripts/e2e/onboard-docker.sh
```

### QR 가져오기 스모크 테스트 (Docker)

```bash
pnpm test:docker:qr
```

### 참고 사항

- 게이트웨이 바인딩은 컨테이너 사용에 맞춰 기본적으로 `lan`으로 설정됩니다.
- 게이트웨이 컨테이너가 세션의 권위 있는 출처입니다 (`~/.openclaw/agents/<agentId>/sessions/`).

## 에이전트 샌드박스 (호스트 게이트웨이 + Docker 도구)

심층 설명: [샌드박스](/gateway/sandboxing)

### 작동 방식

`agents.defaults.sandbox`가 활성화되면, **비-메인 세션**은 Docker 컨테이너 내에서 도구를 실행합니다. 게이트웨이는 여전히 호스트에서 실행되지만 도구 실행은 격리됩니다:

- 범위: 기본값 `"agent"` (에이전트당 하나의 컨테이너 + 작업 공간)
- 범위: `"session"` 세션별 격리
- 범위별 작업 공간 폴더가 `/workspace`에 마운트
- 선택적 에이전트 작업 공간 액세스 (`agents.defaults.sandbox.workspaceAccess`)
- 허용/거부 도구 정책 (거부 우선)
- 인바운드 미디어는 활성 샌드박스 작업 공간에 복사 (`media/inbound/*`)하여 도구가 읽을 수 있음 (`workspaceAccess: "rw"` 사용 시 미디어가 에이전트 작업 공간으로 이동)

경고: `scope: "shared"`는 교차 세션 격리를 비활성화합니다. 모든 세션이 하나의 컨테이너와 하나의 작업 공간을 공유합니다.

### 에이전트별 샌드박스 구성 (멀티 에이전트)

멀티 에이전트 라우팅을 사용하면 각 에이전트가 샌드박스 및 도구 설정을 재정의할 수 있습니다: `agents.list[].sandbox` 및 `agents.list[].tools` (및 `agents.list[].tools.sandbox.tools`). 이를 통해 하나의 게이트웨이에서 혼합 액세스 수준을 실행할 수 있습니다:

- 전체 액세스 (개인 에이전트)
- 읽기 전용 도구 + 읽기 전용 작업 공간 (가족/직장 에이전트)
- 파일 시스템/셸 도구 없음 (공개 에이전트)

예시, 우선순위 및 문제 해결은 [멀티 에이전트 샌드박스 및 도구](/multi-agent-sandbox-tools)를 참조하세요.

### 기본 동작

- 이미지: `openclaw-sandbox:bookworm-slim`
- 에이전트당 하나의 컨테이너
- 에이전트 작업 공간 액세스: `workspaceAccess: "none"` (기본값)은 `~/.openclaw/sandboxes` 사용
  - `"ro"`는 샌드박스 작업 공간을 `/workspace`에 유지하고 에이전트 작업 공간을 읽기 전용으로 `/agent`에 마운트 (`write`/`edit`/`apply_patch` 비활성화)
  - `"rw"`는 에이전트 작업 공간을 읽기-쓰기로 `/workspace`에 마운트
- 자동 정리: 24시간 이상 유휴 또는 7일 이상 경과
- 네트워크: 기본값 `none` (아웃바운드 액세스 필요시 명시적 활성화)
- 기본 허용: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- 기본 거부: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### 샌드박스 활성화

`setupCommand`에서 패키지를 설치할 계획이라면 참고하세요:

- 기본 `docker.network`는 `"none"` (아웃바운드 액세스 없음).
- `readOnlyRoot: true`는 패키지 설치를 차단합니다.
- `apt-get`을 실행하려면 `user`가 root여야 합니다 (`user` 생략 또는 `user: "0:0"` 설정).
  `setupCommand` (또는 docker 구성)이 변경되면 OpenClaw는 컨테이너가 **최근 사용되지 않은 경우** (~5분 이내)에 자동으로 재빌드합니다. 활성 컨테이너는 정확한 `openclaw sandbox recreate ...` 명령이 포함된 경고를 기록합니다.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (기본값 agent)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 유휴 정리 비활성화
          maxAgeDays: 7, // 0 최대 기간 정리 비활성화
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

강화 옵션은 `agents.defaults.sandbox.docker` 아래:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

멀티 에이전트: `agents.list[].sandbox.{docker,browser,prune}.*`로 에이전트별로 `agents.defaults.sandbox.{docker,browser,prune}.*` 재정의
(`agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope`가 `"shared"`면 무시됨).

### 기본 샌드박스 이미지 빌드

```bash
scripts/sandbox-setup.sh
```

`Dockerfile.sandbox`로 `openclaw-sandbox:bookworm-slim`을 빌드합니다.

### 샌드박스 공통 이미지 (선택 사항)

일반적인 빌드 도구 (Node, Go, Rust 등)가 포함된 샌드박스 이미지가 필요하면 공통 이미지를 빌드하세요:

```bash
scripts/sandbox-common-setup.sh
```

`openclaw-sandbox-common:bookworm-slim`을 빌드합니다. 사용법:

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } },
    },
  },
}
```

### 샌드박스 브라우저 이미지

샌드박스 내에서 브라우저 도구를 실행하려면 브라우저 이미지를 빌드하세요:

```bash
scripts/sandbox-browser-setup.sh
```

`Dockerfile.sandbox-browser`로 `openclaw-sandbox-browser:bookworm-slim`을 빌드합니다. 컨테이너는 CDP가 활성화된 Chromium을 실행하고 선택적 noVNC 뷰어를 제공합니다 (Xvfb를 통한 헤드풀 모드).

참고:

- 헤드풀 모드 (Xvfb)는 헤드리스보다 봇 감지를 더 줄입니다.
- `agents.defaults.sandbox.browser.headless=true`로 여전히 헤드리스를 사용할 수 있습니다.
- 전체 데스크톱 환경 (GNOME)은 필요 없음; Xvfb가 디스플레이를 제공합니다.

구성 사용법:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true },
      },
    },
  },
}
```

커스텀 브라우저 이미지:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } },
    },
  },
}
```

활성화되면 에이전트는 다음을 받습니다:

- 샌드박스 브라우저 제어 URL (`browser` 도구용)
- noVNC URL (활성화되고 headless=false인 경우)

참고: 도구 허용 목록을 사용하는 경우 `browser`를 추가하고 (거부 목록에서 제거), 그렇지 않으면 도구가 여전히 차단됩니다.
정리 규칙 (`agents.defaults.sandbox.prune`)도 브라우저 컨테이너에 적용됩니다.

### 커스텀 샌드박스 이미지

자신의 이미지를 빌드하고 구성에서 가리키세요:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } },
    },
  },
}
```

### 도구 정책 (허용/거부)

- `deny`가 `allow`보다 우선합니다.
- `allow`가 비어 있으면: 모든 도구 (거부된 것 제외) 사용 가능.
- `allow`가 비어 있지 않으면: `allow`의 도구만 사용 가능 (거부된 것 제외).

### 정리 정책

두 가지 구성:

- `prune.idleHours`: X시간 이상 사용되지 않은 컨테이너 제거 (0 = 비활성화)
- `prune.maxAgeDays`: X일 이상 된 컨테이너 제거 (0 = 비활성화)

예시:

- 활성 세션 유지하되 수명 제한:
  `idleHours: 24`, `maxAgeDays: 7`
- 정리 안 함:
  `idleHours: 0`, `maxAgeDays: 0`

### 보안 참고

- 하드 격리는 **도구** (exec/read/write/edit/apply_patch)에만 적용됩니다.
- 호스트 전용 도구 (browser/camera/canvas 등)는 기본적으로 차단됩니다.
- 샌드박스에서 `browser`를 허용하면 **격리가 깨집니다** (브라우저가 호스트에서 실행).

## 문제 해결

- 이미지 누락: [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh)로 빌드하거나 `agents.defaults.sandbox.docker.image`를 설정하세요.
- 컨테이너 실행 안 됨: 각 세션에서 필요시 자동 생성됩니다.
- 샌드박스 권한 오류: `docker.user`를 마운트된 작업 공간 소유권과 일치하는 UID:GID로 설정 (또는 작업 공간 폴더에 chown).
- 커스텀 도구 찾을 수 없음: OpenClaw는 `sh -lc` (로그인 셸)로 명령을 실행하여 `/etc/profile`을 로드하고 PATH를 재설정할 수 있습니다. `docker.env.PATH`로 커스텀 도구 경로를 앞에 추가 (예: `/custom/bin:/usr/local/share/npm-global/bin`)하거나 Dockerfile의 `/etc/profile.d/`에 스크립트를 추가하세요.
