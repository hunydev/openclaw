---
read_when: '샌드박스'를 만났거나 도구/권한 상승 거부 메시지를 볼 때 정확히 어떤 설정 키를 수정해야 하는지 이해하기 위해 읽으세요.
status: active
summary: 도구가 차단되는 이유: 샌드박스 런타임, 도구 허용/거부 정책, 권한 상승 실행 게이팅
title: 샌드박스 vs 도구 정책 vs 권한 상승 모드
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 863ea5e6d137dfb61f12bd686b9557d6df1fd0c13ba5f15861bf72248bc975f1
  source_path: gateway/sandbox-vs-tool-policy-vs-elevated.md
  workflow: 14
---

# 샌드박스 vs 도구 정책 vs 권한 상승 모드

OpenClaw에는 관련되어 있지만 서로 다른 세 가지 제어 메커니즘이 있습니다:

1. **샌드박스** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`)는 **도구가 어디서 실행되는지** 결정합니다(Docker 또는 호스트).
2. **도구 정책** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`)은 **어떤 도구가 사용 가능/허용되는지** 결정합니다.
3. **권한 상승 모드** (`tools.elevated.*`, `agents.list[].tools.elevated.*`)는 샌드박스 환경에서 호스트에서 실행하기 위한 **exec 전용 탈출 해치**입니다.

## 빠른 디버깅

인스펙터를 사용하여 OpenClaw가 *실제로* 무엇을 하고 있는지 확인하세요:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

출력 내용:

- 적용되는 샌드박스 모드/범위/작업 공간 접근 권한
- 현재 세션이 샌드박스에 있는지 여부(메인 세션 vs 비메인 세션)
- 적용되는 샌드박스 도구 허용/거부 정책(및 해당 정책이 에이전트 레벨/전역 레벨/기본 레벨에서 오는지)
- 권한 상승 게이팅 및 수정 경로

## 샌드박스: 도구가 실행되는 위치

샌드박스는 `agents.defaults.sandbox.mode`로 제어됩니다:

- `"off"`: 모든 것이 호스트에서 실행됩니다.
- `"non-main"`: 비메인 세션만 샌드박스됩니다(그룹/채널에서 종종 예상과 다름).
- `"all"`: 모든 것이 샌드박스됩니다.

전체 설정 매트릭스(범위, 작업 공간 마운트, 이미지)는 [샌드박싱](/gateway/sandboxing)을 참조하세요.

### 바인드 마운트(보안 빠른 확인)

- `docker.binds`는 샌드박스 파일 시스템을 *관통*합니다: 마운트하는 모든 것이 설정한 모드(`:ro` 또는 `:rw`)로 컨테이너 내에서 볼 수 있습니다.
- 모드를 생략하면 기본값은 읽기-쓰기입니다. 소스 코드/시크릿에는 `:ro`를 권장합니다.
- `scope: "shared"`는 에이전트별 바인드를 무시합니다(전역 바인드만 적용).
- `/var/run/docker.sock`을 바인드하면 사실상 샌드박스에 호스트 제어권을 부여합니다. 의도한 경우에만 사용하세요.
- 작업 공간 접근(`workspaceAccess: "ro"`/`"rw"`)은 바인드 모드와 독립적입니다.

## 도구 정책: 어떤 도구가 존재하고 호출 가능한지

두 가지 레이어를 알아두세요:

- **도구 프로필**: `tools.profile` 및 `agents.list[].tools.profile`(기본 허용 목록)
- **제공자 도구 프로필**: `tools.byProvider[provider].profile` 및 `agents.list[].tools.byProvider[provider].profile`
- **전역/에이전트별 도구 정책**: `tools.allow`/`tools.deny` 및 `agents.list[].tools.allow`/`agents.list[].tools.deny`
- **제공자 도구 정책**: `tools.byProvider[provider].allow/deny` 및 `agents.list[].tools.byProvider[provider].allow/deny`
- **샌드박스 도구 정책**(샌드박스일 때만 적용): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` 및 `agents.list[].tools.sandbox.tools.*`

경험 법칙:

- `deny`가 항상 우선합니다.
- `allow`가 비어 있지 않으면 다른 모든 도구는 차단된 것으로 간주됩니다.
- 도구 정책은 하드 게이트입니다: `/exec`는 거부된 `exec` 도구를 재정의할 수 없습니다.
- `/exec`는 권한 있는 발신자의 세션 기본값만 변경합니다. 도구 접근 권한을 부여하지 않습니다.
  제공자 도구 키는 `provider`(예: `google-antigravity`) 또는 `provider/model`(예: `openai/gpt-5.2`) 형식을 허용합니다.

### 도구 그룹(약식)

도구 정책(전역, 에이전트, 샌드박스)은 여러 도구로 확장되는 `group:*` 항목을 지원합니다:

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

사용 가능한 그룹:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: 모든 내장 OpenClaw 도구(제공자 플러그인 제외)

## 권한 상승 모드: exec 전용 "호스트에서 실행"

권한 상승 모드는 추가 도구를 부여하지 **않습니다**. `exec`에만 영향을 줍니다.

- 샌드박스 내에 있을 때 `/elevated on`(또는 `elevated: true`로 `exec`)은 호스트에서 실행됩니다(승인이 여전히 적용될 수 있음).
- `/elevated full`을 사용하면 현재 세션의 exec 승인을 건너뜁니다.
- 이미 직접 실행 모드라면 권한 상승은 사실상 무작동입니다(여전히 게이트됨).
- 권한 상승 모드는 Skills 범위가 **아니며** 도구 허용/거부 정책을 재정의하지 **않습니다**.
- `/exec`는 권한 상승 모드와 별개입니다. 권한 있는 발신자의 세션당 exec 기본값만 조정합니다.

게이팅:

- 활성화: `tools.elevated.enabled`(및 선택적으로 `agents.list[].tools.elevated.enabled`)
- 발신자 허용 목록: `tools.elevated.allowFrom.<provider>`(및 선택적으로 `agents.list[].tools.elevated.allowFrom.<provider>`)

[권한 상승 모드](/tools/elevated)를 참조하세요.

## 일반적인 "샌드박스" 수정 사항

### "도구 X가 샌드박스 도구 정책에 의해 차단됨"

수정 키(하나 선택):

- 샌드박스 비활성화: `agents.defaults.sandbox.mode=off`(또는 에이전트별 `agents.list[].sandbox.mode=off`)
- 샌드박스 내에서 도구 허용:
  - `tools.sandbox.tools.deny`(또는 에이전트별 `agents.list[].tools.sandbox.tools.deny`)에서 제거
  - 또는 `tools.sandbox.tools.allow`(또는 에이전트별 허용 목록)에 추가

### "이게 메인 세션인 줄 알았는데, 왜 샌드박스인가요?"

`"non-main"` 모드에서 그룹/채널 키는 메인 세션이 *아닙니다*. 메인 세션 키를 사용하거나(`sandbox explain`으로 확인) 모드를 `"off"`로 전환하세요.
