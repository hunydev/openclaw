---
read_when: 샌드박스 메커니즘을 깊이 이해하거나 agents.defaults.sandbox 구성을 조정해야 할 때
status: active
summary: OpenClaw 샌드박스 작동 방식 - 모드, 범위, 작업 공간 액세스 및 이미지
title: 샌드박스
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 184fc53001fc6b2847bbb1963cc9c54475d62f74555a581a262a448a0333a209
  source_path: gateway/sandboxing.md
  workflow: 14
---

# 샌드박스

OpenClaw는 **Docker 컨테이너 내에서 도구를 실행**하여 영향 범위를 줄일 수 있습니다.
이 기능은 **선택 사항**이며 구성으로 제어됩니다 (`agents.defaults.sandbox` 또는
`agents.list[].sandbox`). 샌드박스가 활성화되지 않으면 도구는 호스트에서 실행됩니다.
Gateway는 항상 호스트에서 실행; 샌드박스가 활성화되면 도구 실행이 격리된 샌드박스에서 진행됩니다.

이것은 완벽한 보안 경계가 아니지만 모델이 잘못된 작업을 수행할 때 파일 시스템 및 프로세스 액세스를 효과적으로 제한합니다.

## 샌드박스되는 것

- 도구 실행 (`exec`, `read`, `write`, `edit`, `apply_patch`, `process` 등).
- 선택적 샌드박스 브라우저 (`agents.defaults.sandbox.browser`).
  - 기본적으로 샌드박스 브라우저는 브라우저 도구가 필요할 때 자동 시작됩니다 (CDP 접근 가능 보장).
    `agents.defaults.sandbox.browser.autoStart` 및 `agents.defaults.sandbox.browser.autoStartTimeoutMs`로 구성.
  - `agents.defaults.sandbox.browser.allowHostControl`은 샌드박스 세션이 명시적으로 호스트 브라우저에 액세스하도록 허용.
  - 선택적 허용 목록이 `target: "custom"` 제어: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

샌드박스되지 않는 것:

- Gateway 프로세스 자체.
- 호스트에서 실행이 명시적으로 허용된 모든 도구 (예: `tools.elevated`).
  - **승격된 exec는 호스트에서 실행되어 샌드박스를 우회합니다.**
  - 샌드박스가 활성화되지 않으면 `tools.elevated`는 실행 방식을 변경하지 않음 (이미 호스트에 있음). [승격 모드](/tools/elevated) 참조.

## 모드

`agents.defaults.sandbox.mode`는 **언제** 샌드박스를 사용할지 제어:

- `"off"`: 샌드박스 사용 안 함.
- `"non-main"`: **비-메인** 세션에만 샌드박스 활성화 (일반 채팅이 호스트에서 실행되길 원하면 기본값).
- `"all"`: 모든 세션이 샌드박스에서 실행.
  참고: `"non-main"`은 에이전트 ID가 아닌 `session.mainKey` (기본값 `"main"`)를 기준으로 함.
  그룹/채널 세션은 각자의 키를 사용하므로 비-메인으로 취급되어 샌드박스됨.

## 범위

`agents.defaults.sandbox.scope`는 **얼마나 많은 컨테이너가 생성되는지** 제어:

- `"session"` (기본값): 세션당 하나의 컨테이너.
- `"agent"`: 에이전트당 하나의 컨테이너.
- `"shared"`: 모든 샌드박스 세션이 하나의 컨테이너 공유.

## 작업 공간 액세스

`agents.defaults.sandbox.workspaceAccess`는 **샌드박스가 무엇을 볼 수 있는지** 제어:

- `"none"` (기본값): 도구가 `~/.openclaw/sandboxes` 아래 샌드박스 작업 공간에서 실행.
- `"ro"`: 에이전트 작업 공간을 읽기 전용으로 `/agent`에 마운트 (`write`/`edit`/`apply_patch` 비활성화).
- `"rw"`: 에이전트 작업 공간을 읽기-쓰기로 `/workspace`에 마운트.

인바운드 미디어는 활성 샌드박스 작업 공간에 복사됩니다 (`media/inbound/*`).
Skills 참고: `read` 도구는 샌드박스를 루트로 사용. `workspaceAccess: "none"`일 때
OpenClaw는 적합한 Skills를 샌드박스 작업 공간 (`.../skills`)에 미러링하여 읽기 가능. `"rw"`일 때 작업 공간 Skills를
`/workspace/skills`에서 읽을 수 있음.

## 커스텀 바인드 마운트

`agents.defaults.sandbox.docker.binds`는 추가 호스트 디렉토리를 컨테이너에 마운트.
형식: `host:container:mode` (예: `"/home/user/source:/source:rw"`).

전역 및 에이전트별 바인드 마운트는 **병합**됨 (교체가 아님). `scope: "shared"`에서 에이전트별 바인드 마운트는 무시됨.

예시 (읽기 전용 소스 + Docker 소켓):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/run/docker.sock:/var/run/docker.sock"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

보안 참고:

- 바인드 마운트는 샌드박스 파일 시스템을 우회: 설정한 모드 (`:ro` 또는 `:rw`)로 호스트 경로를 노출.
- 민감한 마운트 (예: `docker.sock`, 시크릿, SSH 키)는 쓰기가 정말 필요한 게 아니면 `:ro` 사용.
- 작업 공간에 대해 읽기 액세스만 필요하면 `workspaceAccess: "ro"`와 함께 사용; 바인드 모드는 독립적으로 유지.
- 바인드 마운트가 도구 정책 및 승격된 exec와 상호작용하는 방식은 [샌드박스 vs 도구 정책 vs 승격](/gateway/sandbox-vs-tool-policy-vs-elevated) 참조.

## 이미지 + 설정

기본 이미지: `openclaw-sandbox:bookworm-slim`

한 번 빌드:

```bash
scripts/sandbox-setup.sh
```

참고: 기본 이미지에는 **Node가 포함되지 않음**. Skills에 Node (또는
다른 런타임)가 필요하면 커스텀 이미지를 빌드하거나
`sandbox.docker.setupCommand`를 통해 설치 (네트워크 이그레스 + 쓰기 가능한 루트 파일 시스템 +
root 사용자 필요).

샌드박스 브라우저 이미지:

```bash
scripts/sandbox-browser-setup.sh
```

기본적으로 샌드박스 컨테이너는 **네트워크 없이** 실행.
`agents.defaults.sandbox.docker.network`로 재정의 가능.

Docker 설치 및 컨테이너화된 Gateway 지침:
[Docker](/install/docker)

## setupCommand (일회성 컨테이너 설정)

`setupCommand`는 샌드박스 컨테이너 생성 후 **한 번만 실행** (매 실행 시 아님).
컨테이너 내에서 `sh -lc`로 실행됩니다.

경로:

- 전역: `agents.defaults.sandbox.docker.setupCommand`
- 에이전트별: `agents.list[].sandbox.docker.setupCommand`

일반적인 문제:

- 기본 `docker.network`가 `"none"` (이그레스 없음)이므로 패키지 설치 실패.
- `readOnlyRoot: true`는 패키지 설치를 차단.
- `apt-get` 실행에는 `user`가 root여야 함 (`user` 생략 또는 `user: "0:0"` 설정).

구성 또는 setupCommand가 변경되면 OpenClaw가 컨테이너를 자동 재빌드하며,
컨테이너가 **최근 사용되지 않은 경우**에만 (~5분 이내). 활성 컨테이너는 정확한
`openclaw sandbox recreate ...` 명령이 포함된 경고를 기록.

예시 (네트워크 + root 사용자 + setupCommand 포함):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          network: "bridge",
          readOnlyRoot: false,
          user: "0:0",
          setupCommand: "apt-get update && apt-get install -y git curl jq",
        },
      },
    },
  },
}
```

## 에이전트별 샌드박스 구성

`agents.list[].sandbox`를 사용하여 에이전트별로 샌드박스 설정 재정의.
에이전트 구성은 `agents.defaults.sandbox` 위에 **병합**됩니다 (교체가 아님).

예시:

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "non-main", scope: "session" },
    },
    list: [
      { id: "main" }, // 기본값 사용
      { id: "build", sandbox: { workspaceAccess: "rw" } }, // rw 재정의
    ],
  },
}
```

`scope: "shared"` 사용 시 에이전트별 바인드 마운트 및 setupCommand는 무시됩니다.

## 브라우저 샌드박스

`agents.defaults.sandbox.browser`는 샌드박스 브라우저 동작 제어:

- `enabled: true` — 브라우저 도구가 샌드박스 브라우저를 사용.
- `image` — 커스텀 브라우저 샌드박스 이미지 (기본값: `openclaw-sandbox-browser:bookworm-slim`).
- `headless` — `true`면 헤드리스 모드 (기본값: `false` — Xvfb로 유헤드).
- `autoStart` — 필요시 자동 시작 (기본값: `true`).
- `autoStartTimeoutMs` — 자동 시작 타임아웃 (기본값: `30000`).
- `allowHostControl` — 샌드박스 세션이 호스트 브라우저에 액세스 허용 (기본값: `false`).

브라우저 샌드박스 이미지 빌드:

```bash
scripts/sandbox-browser-setup.sh
```

## 정리 정책

`agents.defaults.sandbox.prune`은 유휴 컨테이너 자동 정리 제어:

- `idleHours` — X시간 이상 미사용 컨테이너 제거 (기본값: 24).
- `maxAgeDays` — X일 이상 된 컨테이너 제거 (기본값: 7).

정리 비활성화: 둘 다 `0`으로 설정.

## 관련 항목

- [Docker](/install/docker) — 컨테이너화된 Gateway 설정
- [멀티 에이전트 샌드박스 및 도구](/multi-agent-sandbox-tools) — 에이전트별 격리
- [샌드박스 vs 도구 정책 vs 승격](/gateway/sandbox-vs-tool-policy-vs-elevated) — 상호작용 이해
