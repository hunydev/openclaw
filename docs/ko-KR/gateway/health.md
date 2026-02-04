---
read_when:
  - WhatsApp 채널 상태 진단 시
summary: 채널 연결을 위한 상태 검사 단계
title: 상태 검사
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 74f242e98244c135e1322682ed6b67d70f3b404aca783b1bb5de96a27c2c1b01
  source_path: gateway/health.md
  workflow: 14
---

# 상태 검사 (CLI)

추측 없이 채널 연결 상태를 확인하는 간단한 가이드입니다.

## 빠른 확인

- `openclaw status` — 로컬 요약: Gateway 도달 가능성/모드, 업데이트 힌트, 연결된 채널 인증 기간, 세션 + 최근 활동.
- `openclaw status --all` — 전체 로컬 진단 (읽기 전용, 컬러, 디버그용으로 붙여넣기 가능).
- `openclaw status --deep` — 실행 중인 Gateway도 프로브 (지원 시 채널별 프로브).
- `openclaw health --json` — 실행 중인 Gateway에서 전체 상태 스냅샷 요청 (WS만; Baileys 소켓에 직접 액세스 안 함).
- WhatsApp/WebChat에서 `/status`를 독립 메시지로 보내면 에이전트를 깨우지 않고 상태 응답을 받음.
- 로그: `/tmp/openclaw/openclaw-*.log` 테일 후 `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound` 필터링.

## 심층 진단

- 디스크의 자격 증명: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (수정 시간이 최근이어야 함).
- 세션 저장소: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (구성에서 경로 오버라이드 가능). 개수와 최근 수신자는 `status`를 통해 확인.
- 재연결 흐름: 로그에 상태 코드 409–515 또는 `loggedOut`이 표시되면 `openclaw channels logout && openclaw channels login --verbose` 실행. (참고: QR 로그인 흐름은 페어링 후 상태 코드 515를 만나면 자동으로 한 번 재시도.)

## 문제 발생 시

- `logged out` 또는 상태 코드 409–515 → `openclaw channels logout` 후 `openclaw channels login`으로 재연결.
- Gateway 도달 불가 → 시작: `openclaw gateway --port 18789` (포트 점유 시 `--force` 사용).
- 인바운드 메시지 없음 → 연결된 전화가 온라인이고 발신자가 허용되었는지 확인 (`channels.whatsapp.allowFrom`); 그룹 채팅의 경우 허용 목록 + 멘션 규칙 일치 확인 (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## 전용 "health" 명령

`openclaw health --json`은 실행 중인 Gateway에서 상태 스냅샷을 요청합니다 (CLI는 채널 소켓에 직접 액세스하지 않음). 연결된 자격 증명/인증 기간 (가용 시), 채널별 프로브 요약, 세션 저장소 요약 및 프로브 소요 시간을 보고합니다. Gateway에 도달할 수 없거나 프로브가 실패/타임아웃되면 0이 아닌 상태 코드로 종료합니다. `--timeout <ms>`로 기본 10초 타임아웃을 오버라이드할 수 있습니다.
