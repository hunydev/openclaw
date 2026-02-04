---
read_when:
  - Telegram 또는 grammY 관련 기능 개발 시
summary: grammY를 통한 Telegram Bot API 통합 및 설정 안내
title: grammY
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: ea7ef23e6d77801f4ef5fc56685ef4470f79f5aecab448d644a72cbab53521b7
  source_path: channels/grammy.md
  workflow: 14
---

# grammY 통합(Telegram Bot API)

# grammY를 선택한 이유

- TypeScript 우선 Bot API 클라이언트로 내장 롱 폴링 + webhook 헬퍼, 미들웨어, 오류 처리, 속도 제한기 포함.
- 수동 fetch + FormData보다 깔끔한 미디어 헬퍼; 모든 Bot API 메서드 지원.
- 확장 가능: 커스텀 fetch를 통한 프록시 지원, 세션 미들웨어(선택적), 타입 안전 컨텍스트.

# 제공된 기능

- **단일 클라이언트 경로:** fetch 기반 구현 제거됨; grammY가 이제 유일한 Telegram 클라이언트(전송 + Gateway), 기본적으로 grammY throttler 활성화.
- **Gateway:** `monitorTelegramProvider`는 grammY `Bot`을 구축하고 멘션/허용 목록 게이팅에 연결, `getFile`/`download`를 통한 미디어 다운로드, `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument`를 통한 응답 전달. `webhookCallback`을 통한 롱 폴링 또는 webhook 지원.
- **프록시:** 선택적 `channels.telegram.proxy`는 grammY의 `client.baseFetch`를 통해 `undici.ProxyAgent` 사용.
- **Webhook 지원:** `webhook-set.ts`는 `setWebhook/deleteWebhook`을 래핑; `webhook.ts`는 헬스체크 + 정상 종료를 지원하는 콜백 호스팅. `channels.telegram.webhookUrl` + `channels.telegram.webhookSecret`이 설정되면 Gateway가 webhook 모드 활성화(그렇지 않으면 롱 폴링).
- **세션:** DM은 에이전트 메인 세션(`agent:<agentId>:<mainKey>`)으로 병합; 그룹은 `agent:<agentId>:telegram:group:<chatId>` 사용; 응답은 동일 채널로 라우팅.
- **설정 옵션:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups`(허용 목록 + 멘션 기본값), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **초안 스트리밍:** 선택적 `channels.telegram.streamMode`는 개인 토픽 채팅에서 `sendMessageDraft`(Bot API 9.3+) 사용. 이는 채널 청크 스트리밍과 별개.
- **테스트:** grammY mock이 DM + 그룹 멘션 게이팅 및 아웃바운드 전송을 커버; 더 많은 미디어/webhook 테스트 케이스 환영.

논의할 문제

- Bot API 429 오류 발생 시 선택적 grammY 플러그인(throttler) 고려.
- 더 많은 구조화된 미디어 테스트 추가(스티커, 음성 메시지).
- webhook 수신 포트를 설정 가능하게(현재 Gateway를 통하지 않으면 8787로 고정됨).
