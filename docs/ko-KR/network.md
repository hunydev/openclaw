---
read_when:
  - 네트워크 아키텍처와 보안 개요를 알아야 하는 경우
  - 로컬 액세스 vs tailnet 액세스 또는 페어링 문제를 디버깅하는 경우
  - 네트워크 문서의 권위 있는 목록이 필요한 경우
summary: 네트워크 허브: Gateway 인터페이스, 페어링, 발견 및 보안
title: 네트워크
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  workflow: 14
---

# 네트워크 허브

이 허브는 OpenClaw가 localhost, LAN, tailnet 간에 연결, 페어링 및 기기 보안을 처리하는 방법에 대한 핵심 문서를 수집합니다.

## 핵심 모델

- [Gateway 아키텍처](/concepts/architecture)
- [Gateway 프로토콜](/gateway/protocol)
- [Gateway 운영 가이드](/gateway)
- [웹 인터페이스 + 바인드 모드](/web)

## 페어링 + 신원

- [페어링 개요(DM + 노드)](/start/pairing)
- [Gateway 소유 노드 페어링](/gateway/pairing)
- [기기 CLI(페어링 + 토큰 로테이션)](/cli/devices)
- [페어링 CLI(DM 승인)](/cli/pairing)

로컬 신뢰:

- 로컬 연결(local loopback 또는 Gateway 호스트 자체의 tailnet 주소)은 동일 호스트 사용자 경험을 원활하게 유지하기 위해 페어링을 자동 승인할 수 있습니다.
- 비로컬 tailnet/LAN 클라이언트는 여전히 명시적인 페어링 승인이 필요합니다.

## 발견 + 전송

- [발견 및 전송](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [원격 액세스(SSH)](/gateway/remote)
- [Tailscale](/gateway/tailscale)

## 노드 + 전송

- [노드 개요](/nodes)
- [Bridge 프로토콜(레거시 노드)](/gateway/bridge-protocol)
- [노드 운영 가이드: iOS](/platforms/ios)
- [노드 운영 가이드: Android](/platforms/android)

## 보안

- [보안 개요](/gateway/security)
- [Gateway 구성 참조](/gateway/configuration)
- [문제 해결](/gateway/troubleshooting)
- [Doctor](/gateway/doctor)
