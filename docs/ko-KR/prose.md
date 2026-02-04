---
read_when:
  - .prose 워크플로를 실행하거나 작성하려고 할 때
  - OpenProse 플러그인을 활성화하려고 할 때
  - 상태 저장소에 대해 알아야 할 때
summary: OpenProse: OpenClaw의 .prose 워크플로, 슬래시 명령 및 상태 관리
title: OpenProse
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# OpenProse

OpenProse는 AI 세션을 오케스트레이션하기 위한 이식 가능한 Markdown 중심 워크플로 형식입니다. OpenClaw에서는 플러그인으로 제공되며, 설치 시 OpenProse Skills 번들과 `/prose` 슬래시 명령이 포함됩니다. 프로그램은 `.prose` 파일에 저장되며 명시적 제어 흐름으로 여러 하위 에이전트를 생성할 수 있습니다.

공식 웹사이트: https://www.prose.md

## 기능 개요

- 명시적 병렬 처리를 지원하는 다중 에이전트 연구 및 합성.
- 반복 가능한 승인 안전 워크플로(코드 리뷰, 이벤트 분류, 콘텐츠 파이프라인).
- 지원되는 에이전트 런타임에서 실행 가능한 재사용 가능한 `.prose` 프로그램.

## 설치 및 활성화

내장 플러그인은 기본적으로 비활성화되어 있습니다. OpenProse 활성화:

```bash
openclaw plugins enable open-prose
```

플러그인 활성화 후 Gateway를 재시작하세요.

개발/로컬 체크아웃: `openclaw plugins install ./extensions/open-prose`

관련 문서: [플러그인](/plugin), [플러그인 매니페스트](/plugins/manifest), [Skills](/tools/skills).

## 슬래시 명령

OpenProse는 `/prose`를 사용자가 호출할 수 있는 Skills 명령으로 등록합니다. OpenClaw 도구를 기반으로 하는 OpenProse VM 명령어로 라우팅됩니다.

일반적인 명령:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## 예시: 간단한 `.prose` 파일

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## 파일 위치

OpenProse는 워크스페이스의 `.prose/` 디렉토리에 상태를 저장합니다:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

사용자 수준의 영구 에이전트는 다음에 저장됩니다:

```
~/.openclaw/prose/agents/
```

## 상태 관리

각 실행은 독립된 `runs/` 서브디렉토리를 생성합니다. 상태는 `state.md`에 Markdown으로 저장되어 검사 및 디버깅이 쉽습니다.

바인딩(변수)은 `bindings/` 폴더에 저장됩니다. 에이전트 컨텍스트는 `agents/`에 저장됩니다.

## 고급 기능

- **병렬 실행**: `parallel:` 블록으로 여러 에이전트가 동시에 작업을 수행합니다.
- **컨텍스트 공유**: `context:` 키워드로 이전 결과를 다음 세션에 전달합니다.
- **조건부 실행**: `if:` 블록으로 조건부 로직을 구현합니다.
- **반복**: `loop:` 블록으로 반복 작업을 수행합니다.

## 관련 문서

- [플러그인](/plugin)
- [플러그인 매니페스트](/plugins/manifest)
- [Skills](/tools/skills)
