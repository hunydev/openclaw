---
description: Real-world OpenClaw projects from the community
summary: OpenClaw로 구동되는 커뮤니티 빌드 프로젝트 및 통합 쇼케이스
title: 프로젝트 쇼케이스
x-i18n:
  generated_at: "2026-02-03T00:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: b3460f6a7b9948799a6082fee90fa8e5ac1d43e34872aea51ba431813dcead7a
  source_path: start/showcase.md
  workflow: 15
---

# 프로젝트 쇼케이스

커뮤니티의 실제 프로젝트입니다. 사람들이 OpenClaw로 무엇을 만들고 있는지 확인해보세요.

<Info>
**소개되고 싶으신가요?** [Discord의 #showcase 채널](https://discord.gg/clawd)에서 프로젝트를 공유하거나 [X에서 @openclaw를 태그](https://x.com/openclaw)해주세요.
</Info>

## 🎥 OpenClaw 실전 데모

VelvetShark의 전체 설정 튜토리얼 (28분).

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw: The self-hosted AI that Siri should have been (Full setup)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="OpenClaw showcase video"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="OpenClaw community showcase"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=5kkIJNUGFho)

## 🆕 Discord 최신 소식

<CardGroup cols={2}>

<Card title="PR 리뷰 → Telegram 피드백" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
  **@bangnokia** • `review` `github` `telegram`

OpenCode가 변경 완료 → PR 오픈 → OpenClaw가 diff를 리뷰하고 Telegram에서 "사소한 제안"과 함께 명확한 머지 의견 (우선 적용해야 할 중요 수정 포함) 답변.

  <img src="/assets/showcase/pr-review-telegram.jpg" alt="OpenClaw PR 리뷰 피드백을 Telegram으로 전달" />
</Card>

<Card title="몇 분 만에 와인 셀러 Skill 생성" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
  **@prades_maxime** • `skills` `local` `csv`

"Robby" (@openclaw)에게 로컬 와인 셀러 Skill 요청. CSV 샘플 내보내기 + 저장 위치를 요청한 후 빠르게 Skill을 빌드하고 테스트 (예시에서 962병).

  <img src="/assets/showcase/wine-cellar-skill.jpg" alt="CSV에서 로컬 와인 셀러 Skill을 빌드하는 OpenClaw" />
</Card>

<Card title="Tesco 쇼핑 자동화" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
  **@marchattonhere** • `automation` `browser` `shopping`

주간 식단 계획 → 정기 구매 품목 → 배송 시간대 예약 → 주문 확인. API 없이 브라우저 제어만으로.

  <img src="/assets/showcase/tesco-shop.jpg" alt="채팅을 통한 Tesco 쇼핑 자동화" />
</Card>

<Card title="SNAG 스크린샷-to-Markdown" icon="scissors" href="https://github.com/am-will/snag">
  **@am-will** • `devtools` `screenshots` `markdown`

단축키로 화면 영역 선택 → Gemini 비전 → 즉시 클립보드에 Markdown.

  <img src="/assets/showcase/snag.png" alt="SNAG 스크린샷-to-Markdown 도구" />
</Card>

<Card title="Agents UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
  **@kitze** • `ui` `skills` `sync`

Agents, Claude, Codex, OpenClaw 간의 Skills/명령어를 관리하는 데스크톱 앱.

  <img src="/assets/showcase/agents-ui.jpg" alt="Agents UI 앱" />
</Card>

<Card title="Telegram 음성 메시지 (papla.media)" icon="microphone" href="https://papla.media/docs">
  **커뮤니티** • `voice` `tts` `telegram`

papla.media TTS를 래핑하고 결과를 Telegram 음성 메시지로 전송 (성가신 자동 재생 없음).

  <img src="/assets/showcase/papla-tts.jpg" alt="TTS 출력의 Telegram 음성 메시지" />
</Card>

<Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
  **@odrobnik** • `devtools` `codex` `brew`

Homebrew로 설치하는 헬퍼로 로컬 OpenAI Codex 세션 (CLI + VS Code) 목록/검사/모니터링.

  <img src="/assets/showcase/codexmonitor.png" alt="ClawHub의 CodexMonitor" />
</Card>

<Card title="Bambu 3D 프린터 제어" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
  **@tobiasbischoff** • `hardware` `3d-printing` `skill`

BambuLab 프린터 제어 및 문제 해결: 상태, 작업, 카메라, AMS, 캘리브레이션 등.

  <img src="/assets/showcase/bambu-cli.png" alt="ClawHub의 Bambu CLI Skill" />
</Card>

<Card title="비엔나 대중교통 (Wiener Linien)" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
  **@hjanuschka** • `travel` `transport` `skill`

비엔나 대중교통의 실시간 출발 정보, 운행 중단, 엘리베이터 상태 및 경로 안내.

  <img src="/assets/showcase/wienerlinien.png" alt="ClawHub의 Wiener Linien Skill" />
</Card>

<Card title="ParentPay 학교 급식" icon="utensils" href="#">
  **@George5562** • `automation` `browser` `parenting`

ParentPay를 통한 영국 학교 급식 자동 예약. 신뢰할 수 있는 테이블 셀 클릭을 위해 마우스 좌표 사용.
</Card>

<Card title="R2 업로드 (내 파일 보내기)" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
  **@julianengel** • `files` `r2` `presigned-urls`

Cloudflare R2/S3에 업로드하고 보안 presigned 다운로드 링크 생성. 원격 OpenClaw 인스턴스에 최적.
</Card>

<Card title="Telegram으로 iOS 앱 개발" icon="mobile" href="#">
  **@coard** • `ios` `xcode` `testflight`

지도와 음성 녹음 기능이 있는 완전한 iOS 앱을 Telegram 채팅만으로 빌드하여 TestFlight에 배포.

  <img src="/assets/showcase/ios-testflight.jpg" alt="TestFlight의 iOS 앱" />
</Card>

<Card title="Oura 링 건강 어시스턴트" icon="heart-pulse" href="#">
  **@AS** • `health` `oura` `calendar`

Oura 링 데이터를 캘린더, 예약, 헬스장 일정과 통합하는 개인 AI 건강 어시스턴트.

  <img src="/assets/showcase/oura-health.png" alt="Oura 링 건강 어시스턴트" />
</Card>
<Card title="Kev의 드림팀 (14+ 에이전트)" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
  **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

하나의 Gateway 아래 14개 이상의 에이전트를 Opus 4.5 오케스트레이터가 Codex 워커에게 작업 위임. 드림팀 로스터, 모델 선택, 샌드박싱, Webhook, 하트비트, 위임 플로우를 다루는 상세한 [기술 문서](https://github.com/adam91holt/orchestrated-ai-articles). 에이전트 샌드박싱을 위한 [Clawdspace](https://github.com/adam91holt/clawdspace). [블로그 포스트](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/).
</Card>

<Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
  **@NessZerra** • `devtools` `linear` `cli` `issues`

에이전트 워크플로우 (Claude Code, OpenClaw)와 통합되는 Linear CLI. 터미널에서 이슈, 프로젝트, 워크플로우 관리. 첫 외부 PR 머지 완료!
</Card>

<Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
  **@jules** • `messaging` `beeper` `cli` `automation`

Beeper Desktop을 통해 메시지 읽기, 보내기, 보관. Beeper 로컬 MCP API를 사용하여 에이전트가 한 곳에서 모든 채팅 (iMessage, WhatsApp 등) 관리.
</Card>

</CardGroup>

## 🤖 자동화 & 워크플로우

<CardGroup cols={2}>

<Card title="Winix 공기청정기 제어" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

Claude Code가 공기청정기 제어 방법을 발견하고 확인한 후, OpenClaw가 인수하여 실내 공기질 관리.

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="OpenClaw를 통한 Winix 공기청정기 제어" />
</Card>

<Card title="아름다운 하늘 카메라 촬영" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

옥상 카메라에서 트리거: 하늘이 예쁘게 보일 때마다 OpenClaw에게 하늘 사진을 찍어달라고 요청 — Skill을 설계하고 촬영 완료.

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="OpenClaw가 캡처한 옥상 카메라 하늘 스냅샷" />
</Card>

<Card title="비주얼 모닝 브리핑 씬" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

예약된 프롬프트가 매일 아침 OpenClaw 페르소나를 통해 단일 "씬" 이미지 생성 (날씨, 할 일, 날짜, 좋아하는 게시물/인용문).
</Card>

<Card title="패들 코트 예약" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Playtomic 가용성 체커 + 예약 CLI. 다시는 빈 코트를 놓치지 마세요.
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cli 스크린샷" />
</Card>

<Card title="회계 서류 수집" icon="file-invoice-dollar">
  **커뮤니티** • `automation` `email` `pdf`
  
  이메일에서 PDF 수집, 세무사를 위한 문서 준비. 월간 회계 자동화.
</Card>

<Card title="카우치 포테이토 개발 모드" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

Netflix 보면서 Telegram으로 전체 개인 웹사이트 리빌드 — Notion → Astro, 18개 글 마이그레이션, DNS를 Cloudflare로. 노트북은 열지도 않음.
</Card>

<Card title="취업 검색 에이전트" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

구인 목록 검색, 이력서 키워드와 매칭, 관련 기회와 링크 반환. JSearch API를 사용하여 30분 만에 빌드.
</Card>

<Card title="Jira Skill 빌더" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

OpenClaw가 Jira에 연결한 후 즉석에서 새 Skill 생성 (ClawHub에 없을 때).
</Card>

<Card title="Telegram으로 Todoist Skill 생성" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

Todoist 작업 자동화하고 OpenClaw가 Telegram 채팅에서 직접 Skill 생성.
</Card>

<Card title="TradingView 분석" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

브라우저 자동화로 TradingView 로그인, 차트 스크린샷, 요청 시 기술 분석 수행. API 필요 없음 — 브라우저 제어만.
</Card>

<Card title="Slack 자동 지원" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

회사 Slack 채널 모니터링, 유용한 응답 제공, Telegram으로 알림 전달. 요청받지 않고도 배포된 앱의 프로덕션 버그를 자율적으로 수정.
</Card>

</CardGroup>

## 🧠 지식 & 메모리

<CardGroup cols={2}>

<Card title="xuezh 중국어 학습" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  OpenClaw를 통한 발음 피드백과 학습 플로우가 있는 중국어 학습 엔진.
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh 발음 피드백" />
</Card>

<Card title="WhatsApp 메모리 볼트" icon="vault">
  **커뮤니티** • `memory` `transcription` `indexing`
  
  전체 WhatsApp 내보내기 파일 가져오기, 1000개 이상의 음성 메모 트랜스크립션, git 로그와 크로스체크, 링크된 Markdown 리포트 출력.
</Card>

<Card title="Karakeep 시맨틱 검색" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Qdrant + OpenAI/Ollama 임베딩을 사용하여 Karakeep 북마크에 벡터 검색 추가.
</Card>

<Card title="인사이드 아웃 2 스타일 메모리" icon="brain">
  **커뮤니티** • `memory` `beliefs` `self-model`
  
  세션 파일을 기억 → 신념 → 진화하는 자아 모델로 변환하는 별도의 메모리 매니저.
</Card>

</CardGroup>

## 🎙️ 음성 & 전화

<CardGroup cols={2}>

<Card title="Clawdia 전화 브릿지" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Vapi 음성 어시스턴트 ↔ OpenClaw HTTP 브릿지. 에이전트와 거의 실시간 전화 통화.
</Card>

<Card title="OpenRouter 트랜스크립션" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

OpenRouter (Gemini 등)를 통한 다국어 오디오 트랜스크립션. ClawHub에서 이용 가능.
</Card>

</CardGroup>

## 🏗️ 인프라 & 배포

<CardGroup cols={2}>

<Card title="Home Assistant 애드온" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  SSH 터널 지원과 영구 상태를 갖춘 Home Assistant OS에서 실행되는 OpenClaw Gateway.
</Card>

<Card title="Home Assistant Skill" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  자연어로 Home Assistant 디바이스 제어 및 자동화.
</Card>

<Card title="Nix 패키징" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  재현 가능한 배포를 위한 배터리 포함 nixified OpenClaw 설정.
</Card>

<Card title="CalDAV 캘린더" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  khal/vdirsyncer를 사용하는 캘린더 Skill. 셀프 호스팅 캘린더 통합.
</Card>

</CardGroup>

## 🏠 홈 & 하드웨어

<CardGroup cols={2}>

<Card title="GoHome 자동화" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  OpenClaw를 인터페이스로 사용하는 Nix 네이티브 홈 자동화, 아름다운 Grafana 대시보드 포함.
  
  <img src="/assets/showcase/gohome-grafana.png" alt="GoHome Grafana 대시보드" />
</Card>

<Card title="Roborock 로봇청소기" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  자연스러운 대화로 Roborock 로봇청소기 제어.
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Roborock 상태" />
</Card>

</CardGroup>

## 🌟 커뮤니티 프로젝트

<CardGroup cols={2}>

<Card title="StarSwap 마켓플레이스" icon="star" href="https://star-swap.com/">
  **커뮤니티** • `marketplace` `astronomy` `webapp`
  
  완전한 천문 장비 마켓플레이스. OpenClaw 에코시스템을 기반으로 구축.
</Card>

</CardGroup>

---

## 프로젝트 제출하기

공유하고 싶은 것이 있으신가요? 소개해드리고 싶습니다!

<Steps>
  <Step title="공유하기">
    [Discord의 #showcase 채널](https://discord.gg/clawd)에 게시하거나 [X에서 @openclaw 태그](https://x.com/openclaw)
  </Step>
  <Step title="세부 정보 포함">
    무엇을 하는지 알려주시고, 레포/데모 링크, 가능하면 스크린샷 공유
  </Step>
  <Step title="소개되기">
    뛰어난 프로젝트를 이 페이지에 추가합니다
  </Step>
</Steps>
