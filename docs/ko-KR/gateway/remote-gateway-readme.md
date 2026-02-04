---
read_when: SSH를 통해 macOS 앱을 원격 Gateway에 연결할 때
summary: OpenClaw.app 원격 Gateway 연결을 위한 SSH 터널 설정
title: 원격 Gateway 설정
x-i18n:
  generated_at: "2026-02-03T12:00:00Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: b1ae266a7cb4911b82ae3ec6cb98b1b57aca592aeb1dc8b74bbce9b0ea9dd1d1
  source_path: gateway/remote-gateway-readme.md
  workflow: 14
---

# 원격 Gateway로 OpenClaw.app 실행하기

OpenClaw.app은 SSH 터널을 사용하여 원격 Gateway에 연결합니다. 이 가이드는 설정 방법을 안내합니다.

## 개요

```
┌─────────────────────────────────────────────────────────────┐
│                        클라이언트 머신                        │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789(로컬 포트)             │
│                     │                                        │
│                     ▼                                        │
│  SSH 터널 ──────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         원격 머신                              │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 빠른 설정

### 1단계: SSH 설정 추가

`~/.ssh/config`를 편집하고 추가하세요:

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # 예: 172.27.187.184
    User <REMOTE_USER>            # 예: jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

`<REMOTE_IP>`와 `<REMOTE_USER>`를 실제 값으로 교체하세요.

### 2단계: SSH 키 복사

공개 키를 원격 머신에 복사합니다(비밀번호 한 번 입력 필요):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### 3단계: Gateway 토큰 설정

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### 4단계: SSH 터널 시작

```bash
ssh -N remote-gateway &
```

### 5단계: OpenClaw.app 재시작

```bash
# OpenClaw.app 종료(⌘Q) 후 다시 열기:
open /path/to/OpenClaw.app
```

앱이 이제 SSH 터널을 통해 원격 Gateway에 연결됩니다.

---

## 로그인 시 자동 터널 시작

로그인 시 자동으로 SSH 터널을 시작하려면 Launch Agent를 생성하세요.

### PLIST 파일 생성

다음 내용을 `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`로 저장하세요:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Launch Agent 로드

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

이제 터널이:

- 로그인 시 자동으로 시작
- 충돌 시 자동 재시작
- 백그라운드에서 계속 실행

레거시 참고: 기존의 `com.openclaw.ssh-tunnel` LaunchAgent가 있다면 제거하세요.

---

## 문제 해결

**터널 실행 여부 확인:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**터널 재시작:**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**터널 중지:**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

---

## 작동 원리

| 구성 요소                            | 기능 설명                                   |
| ------------------------------------ | ------------------------------------------- |
| `LocalForward 18789 127.0.0.1:18789` | 로컬 포트 18789를 원격 포트 18789로 포워딩  |
| `ssh -N`                             | 포트 포워딩만 하는 SSH 연결(원격 명령 없음) |
| `KeepAlive`                          | 터널 충돌 시 자동 재시작                    |
| `RunAtLoad`                          | 에이전트 로드 시 터널 시작                  |

OpenClaw.app은 클라이언트 머신의 `ws://127.0.0.1:18789`에 연결합니다. SSH 터널이 해당 연결을 Gateway가 실행 중인 원격 머신의 18789 포트로 포워딩합니다.
