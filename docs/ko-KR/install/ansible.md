---
read_when:
  - 보안 강화와 함께 자동화된 서버 배포가 필요할 때
  - VPN 액세스가 있는 방화벽 격리 설정이 필요할 때
  - 원격 Debian/Ubuntu 서버에 배포할 때
summary: Ansible, Tailscale VPN 및 방화벽 격리를 통한 자동화된 보안 강화 OpenClaw 설치
title: Ansible
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 896807f344d923f09039f377c13b4bf4a824aa94eec35159fc169596a3493b29
  source_path: install/ansible.md
  workflow: 14
---

# Ansible 설치

프로덕션 서버에 OpenClaw를 배포하는 권장 방법은 **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — 보안 우선 아키텍처의 자동화 설치 도구입니다.

## 빠른 시작

한 줄 명령으로 설치:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 전체 가이드: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> openclaw-ansible 저장소가 Ansible 배포의 권위 있는 출처입니다. 이 페이지는 간략한 개요일 뿐입니다.

## 얻게 되는 것

- 🔒 **방화벽 우선 보안 정책**: UFW + Docker 격리 (SSH + Tailscale만 접근 가능)
- 🔐 **Tailscale VPN**: 서비스를 공개적으로 노출하지 않는 안전한 원격 액세스
- 🐳 **Docker**: localhost에만 바인딩된 격리된 샌드박스 컨테이너
- 🛡️ **심층 방어**: 4계층 보안 아키텍처
- 🚀 **한 줄 명령 배포**: 몇 분 안에 완전 배포
- 🔧 **Systemd 통합**: 보안 강화와 함께 부팅 시 자동 시작

## 사전 요구 사항

- **OS**: Debian 11+ 또는 Ubuntu 20.04+
- **액세스**: Root 또는 sudo 권한
- **네트워크**: 패키지 설치를 위한 인터넷 연결
- **Ansible**: 2.14+ (빠른 시작 스크립트가 자동 설치)

## 설치 내용

Ansible playbook은 다음을 설치하고 구성합니다:

1. **Tailscale** (안전한 원격 액세스를 위한 mesh VPN)
2. **UFW 방화벽** (SSH + Tailscale 포트만 개방)
3. **Docker CE + Compose V2** (에이전트 샌드박스용)
4. **Node.js 22.x + pnpm** (런타임 의존성)
5. **OpenClaw** (호스트 기반 설치, 컨테이너화되지 않음)
6. **Systemd 서비스** (보안 강화와 함께 자동 시작)

참고: 게이트웨이는 **호스트에서 직접 실행**됩니다 (Docker 내부가 아님). 하지만 에이전트 샌드박스는 격리를 위해 Docker를 사용합니다. [샌드박스](/gateway/sandboxing) 참조.

## 설치 후 설정

설치 완료 후 openclaw 사용자로 전환:

```bash
sudo -i -u openclaw
```

설치 후 스크립트가 다음을 안내합니다:

1. **온보딩 마법사**: OpenClaw 설정 구성
2. **프로바이더 로그인**: WhatsApp/Telegram/Discord/Signal 연결
3. **게이트웨이 테스트**: 설치 확인
4. **Tailscale 설정**: VPN mesh 네트워크에 연결

### 일반적인 명령

```bash
# 서비스 상태 확인
sudo systemctl status openclaw

# 실시간 로그 보기
sudo journalctl -u openclaw -f

# 게이트웨이 재시작
sudo systemctl restart openclaw

# 프로바이더 로그인 (openclaw 사용자로 실행)
sudo -i -u openclaw
openclaw channels login
```

## 보안 아키텍처

### 4계층 방어

1. **방화벽 (UFW)**: SSH (22) + Tailscale (41641/udp)만 공개 노출
2. **VPN (Tailscale)**: 게이트웨이는 VPN mesh 네트워크를 통해서만 접근 가능
3. **Docker 격리**: DOCKER-USER iptables 체인이 외부 포트 노출 차단
4. **Systemd 강화**: NoNewPrivileges, PrivateTmp, 비특권 사용자

### 검증

외부 공격 면 테스트:

```bash
nmap -p- YOUR_SERVER_IP
```

**포트 22** (SSH)만 열려 있어야 합니다. 모든 다른 서비스 (게이트웨이, Docker)는 잠겨 있습니다.

### Docker 가용성

Docker는 **에이전트 샌드박스** (격리된 도구 실행)에 사용되며, 게이트웨이 자체를 실행하는 데 사용되지 않습니다. 게이트웨이는 localhost에만 바인딩되고 Tailscale VPN을 통해 액세스됩니다.

샌드박스 구성은 [멀티 에이전트 샌드박스 및 도구](/multi-agent-sandbox-tools)를 참조하세요.

## 수동 설치

자동화 대신 수동 제어를 원하는 경우:

```bash
# 1. 사전 요구 사항 설치
sudo apt update && sudo apt install -y ansible git

# 2. 저장소 클론
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Ansible 컬렉션 설치
ansible-galaxy collection install -r requirements.yml

# 4. playbook 실행
./run-playbook.sh

# 또는 직접 실행 (이후 /tmp/openclaw-setup.sh 수동 실행)
# ansible-playbook playbook.yml --ask-become-pass
```

## OpenClaw 업데이트

Ansible 설치 프로그램은 OpenClaw를 수동 업데이트로 설정합니다. 표준 업데이트 플로우는 [업데이트](/install/updating)를 참조하세요.

Ansible playbook 재실행 (예: 구성 변경 시):

```bash
cd openclaw-ansible
./run-playbook.sh
```

참고: 멱등성이 있으므로 여러 번 안전하게 실행할 수 있습니다.

## 문제 해결

### 방화벽이 연결을 차단

잠긴 경우:

- 먼저 Tailscale VPN을 통해 액세스 확인
- SSH 액세스 (포트 22)는 항상 허용됨
- 게이트웨이는 **Tailscale을 통해서만** 접근 가능 — 설계 의도

### 서비스가 시작되지 않음

```bash
# 로그 확인
sudo journalctl -u openclaw -n 100

# 권한 확인
sudo ls -la /opt/openclaw

# 수동 시작 테스트
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker 샌드박스 문제

```bash
# Docker 실행 확인
sudo systemctl status docker

# 샌드박스 이미지 확인
sudo docker images | grep openclaw-sandbox

# 샌드박스 이미지 누락 시 빌드
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### 프로바이더 로그인 실패

`openclaw` 사용자로 실행 중인지 확인:

```bash
sudo -i -u openclaw
openclaw channels login
```

## 고급 구성

자세한 보안 아키텍처 및 문제 해결:

- [보안 아키텍처](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [기술 세부 사항](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [문제 해결 가이드](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## 관련 항목

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 전체 배포 가이드
- [Docker](/install/docker) — 컨테이너화된 게이트웨이 설정
- [샌드박스](/gateway/sandboxing) — 에이전트 샌드박스 구성
- [멀티 에이전트 샌드박스 및 도구](/multi-agent-sandbox-tools) — 에이전트별 격리
