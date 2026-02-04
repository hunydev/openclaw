---
read_when:
  - API 프로바이더 장애 시 신뢰할 수 있는 폴백이 필요할 때
  - Claude Code CLI 또는 다른 로컬 AI CLI를 실행하고 재사용하고 싶을 때
  - 세션과 이미지를 지원하면서 순수 텍스트, 도구 없는 경로가 필요할 때
summary: CLI 백엔드: 로컬 AI CLI를 통한 순수 텍스트 폴백
title: CLI 백엔드
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 56a96e83b16a4f6443cbf4a9da7a660c41a5b178af5e13f35352c9d72e1b08dd
  source_path: gateway/cli-backends.md
  workflow: 14
---

# CLI 백엔드 (폴백 런타임)

OpenClaw는 **로컬 AI CLI**를 **순수 텍스트 폴백**으로 실행할 수 있습니다. API 프로바이더가 다운되거나, 제한되거나, 일시적으로 오동작할 때 유용합니다. 이 방식은 의도적으로 보수적으로 설계되었습니다:

- **도구 비활성화** (도구 호출 없음).
- **텍스트 입력 → 텍스트 출력** (신뢰할 수 있음).
- **세션 지원** (후속 대화가 일관성 유지).
- **이미지 전달 가능** (CLI가 이미지 경로를 수락하는 경우).

이것은 주요 경로가 아닌 **안전한 대비책**으로 설계되었습니다. 외부 API에 의존하지 않고 "항상 사용 가능한" 텍스트 응답이 필요할 때 사용하세요.

## 초보자 빠른 시작

**구성 없이** Claude Code CLI를 바로 사용할 수 있습니다 (OpenClaw가 기본 구성을 내장):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI도 바로 사용 가능:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Gateway가 launchd/systemd에서 최소 PATH로 실행 중이면 명령 경로만 추가하면 됩니다:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

그게 전부입니다. CLI 자체 외에 키도, 추가 인증 구성도 필요 없습니다.

## 폴백으로 사용

CLI 백엔드를 폴백 목록에 추가하면 주 모델이 실패할 때만 실행됩니다:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

참고:

- `agents.defaults.models` (허용 목록)를 사용하면 `claude-cli/...`를 포함해야 합니다.
- 주 프로바이더가 실패하면 (인증, 제한, 타임아웃) OpenClaw가 CLI 백엔드를 시도합니다.

## 구성 개요

모든 CLI 백엔드는 다음 위치에 있습니다:

```
agents.defaults.cliBackends
```

각 항목은 **프로바이더 ID**를 키로 합니다 (예: `claude-cli`, `my-cli`).
프로바이더 ID는 모델 참조의 왼쪽 절반이 됩니다:

```
<provider>/<model>
```

### 구성 예제

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## 작동 방식

1. **프로바이더 접두사로 백엔드 선택** (`claude-cli/...`).
2. **시스템 프롬프트 구성**, 동일한 OpenClaw 프롬프트 + 작업 공간 컨텍스트 사용.
3. **CLI 실행**, 세션 ID 첨부 (지원 시) 히스토리 일관성 유지.
4. **출력 파싱** (JSON 또는 순수 텍스트), 최종 텍스트 반환.
5. **세션 ID 지속** (백엔드별로 별도 저장), 후속 대화가 동일한 CLI 세션 재사용.

## 세션

- CLI가 세션을 지원하면 `sessionArg` 설정 (예: `--session-id`), 또는 `sessionArgs` 설정 (플레이스홀더 `{sessionId}`)으로 여러 플래그에 ID 삽입.
- CLI가 다른 플래그와 **재개 서브커맨드**를 사용하면 `resumeArgs` 설정 (재개 시 `args` 교체), 선택적으로 `resumeOutput` (비-JSON 재개용).
- `sessionMode`:
  - `always`: 항상 세션 ID 전송 (저장된 것이 없으면 새 UUID 생성).
  - `existing`: 이전에 세션 ID가 저장된 경우에만 전송.
  - `none`: 세션 ID 전송 안 함.

## 이미지 (패스스루)

CLI가 이미지 경로를 수락하면 `imageArg` 설정:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw는 base64 이미지를 임시 파일에 씁니다. `imageArg`가 설정되면 해당 경로가 CLI 인수로 전달됩니다. `imageArg`가 설정되지 않으면 OpenClaw가 파일 경로를 프롬프트에 추가합니다 (경로 주입). 이것은 순수 텍스트 경로에서 로컬 파일을 자동으로 로드하는 CLI에 충분합니다 (Claude Code CLI가 이렇게 동작합니다).

## 입력 / 출력

- `output: "json"` (기본값)은 JSON 파싱을 시도하고 텍스트 + 세션 ID를 추출합니다.
- `output: "jsonl"`은 JSONL 스트림을 파싱 (Codex CLI `--json`)하고 마지막 에이전트 메시지와 `thread_id` (있는 경우)를 추출합니다.
- `output: "text"`는 표준 출력을 최종 응답으로 사용합니다.

입력 모드:

- `input: "arg"` (기본값)은 프롬프트를 마지막 CLI 인수로 전달합니다.
- `input: "stdin"`은 표준 입력으로 프롬프트를 전송합니다.
- 프롬프트가 길고 `maxPromptArgChars`가 설정되면 표준 입력이 사용됩니다.

## 기본값 (내장)

OpenClaw는 `claude-cli`의 기본 구성을 내장합니다:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw는 `codex-cli`의 기본 구성도 내장합니다:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

필요할 때만 오버라이드하세요 (일반적인 경우: 절대 `command` 경로).

## 제한 사항

- **OpenClaw 도구 없음** (CLI 백엔드는 도구 호출을 받지 않음). 일부 CLI는 자체 에이전트 도구를 실행할 수 있습니다.
- **스트리밍 없음** (CLI 출력이 수집된 후 한 번에 반환).
- **구조화된 출력**은 CLI의 JSON 형식에 따라 다릅니다.
- **Codex CLI 세션**은 텍스트 출력으로 재개됩니다 (JSONL 아님). 초기 `--json` 실행보다 구조화 정도가 낮습니다. OpenClaw 세션은 여전히 정상 작동합니다.

## 문제 해결

- **CLI를 찾을 수 없음**: `command`를 전체 경로로 설정.
- **모델 이름 오류**: `modelAliases`를 사용하여 `provider/model`을 CLI 모델 이름으로 매핑.
- **세션 연속성 없음**: `sessionArg`가 설정되고 `sessionMode`가 `none`이 아닌지 확인 (Codex CLI는 현재 JSON 출력으로 세션 재개 불가).
- **이미지가 무시됨**: `imageArg` 설정 (CLI가 파일 경로를 지원하는지 확인).
