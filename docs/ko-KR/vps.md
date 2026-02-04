---
read_when:
  - 클라우드에서 Gateway를 실행하려고 할 때
  - VPS/호스팅 가이드의 빠른 색인이 필요할 때
summary: OpenClaw용 VPS 호스팅 허브(Oracle/Fly/Hetzner/GCP/exe.dev)
title: VPS 호스팅
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# VPS 호스팅

이 허브 페이지는 지원되는 VPS/호스팅 가이드로 연결하고 클라우드 배포 작동 방식을 높은 수준에서 설명합니다.

## 제공자 선택

- **Railway**(원클릭 + 브라우저 설정): [Railway](/railway)
- **Northflank**(원클릭 + 브라우저 설정): [Northflank](/northflank)
- **Oracle Cloud(영구 무료)**: [Oracle](/platforms/oracle) — $0/월(영구 무료, ARM 아키텍처; 용량/가입이 불안정할 수 있음)
- **Fly.io**: [Fly.io](/platforms/fly)
- **Hetzner (Docker)**: [Hetzner](/platforms/hetzner)
- **GCP (Compute Engine)**: [GCP](/platforms/gcp)
- **exe.dev**(VM + HTTPS 프록시): [exe.dev](/platforms/exe-dev)
- **AWS (EC2/Lightsail/프리 티어)**: 마찬가지로 잘 작동합니다. 비디오 가이드:
  https://x.com/techfrenAJ/status/2014934471095812547

## 클라우드 설정 작동 방식

- **Gateway가 VPS에서 실행**되며 상태와 워크스페이스를 소유합니다.
- 노트북/휴대폰에서 **제어 UI** 또는 **Tailscale/SSH**를 통해 연결합니다.
- VPS를 데이터 소스로 취급하고 상태와 워크스페이스를 **백업**하세요.
- 보안 기본값: Gateway를 로컬 루프백에 바인딩하고 SSH 터널 또는 Tailscale Serve를 통해 액세스합니다.
  `lan`/`tailnet`에 바인딩하는 경우 `gateway.auth.token` 또는 `gateway.auth.password`를 설정하세요.

원격 액세스: [Gateway 원격 액세스](/gateway/remote)
플랫폼 허브: [플랫폼](/platforms)

## VPS에서 노드 사용

Gateway를 클라우드에 유지하고 로컬 장치(Mac/iOS/Android/헤드리스 장치)에서 **노드**를 페어링할 수 있습니다. 노드는 로컬 화면/카메라/캔버스 및 `system.run` 기능을 제공하고 Gateway는 클라우드에서 계속 실행됩니다.

문서: [노드](/nodes), [노드 CLI](/cli/nodes)
