---
title: JumpServer — Bastion Host và Privileged Access Management cho hạ tầng hiện đại
description: >-
  Phân tích JumpServer PAM open-source: kiến trúc bastion host, session
  recording, credential vault, command filtering. So sánh với Teleport và
  Guacamole. Hướng dẫn deployment Docker.
pubDate: '2026-06-19'
lang: vi
tags:
  - devops
  - security
  - infrastructure
concepts:
  - bastion-host
  - privileged-access-management
  - session-recording
  - credential-vault
  - rbac
tools:
  - jumpserver
  - docker
  - nginx
  - mysql
  - redis
topics:
  - DevOps
  - Security
readTime: 8
---
# JumpServer — Bastion Host và PAM cho hạ tầng hiện đại

Khi hạ tầng server tăng规模, quản lý truy cập SSH/RDP trở thành bài toán hóc búa. DevOps cần debug nhanh, Security cần audit log, Compliance cần byok/session recording. Chia sẻ SSH key qua Slack, dùng chung `root`, không có audit trail — đây là công thức cho breach.

JumpServer là một **Privileged Access Management (PAM)** open-source giải quyết bài toán này. Bài viết này phân tích kiến trúc, use cases, và cách triển khai thực tế.

## PAM là gì? Tại sao cần?

**Privileged Access Management** là tập hợp các kỹ thuật kiểm soát, giám sát, và audit các truy cập có đặc quyền (privileged access) vào hệ thống:

- **SSH đến production server**
- **RDP đến Windows server**
- **Database admin access**
- **Cloud console access**

Không giống VPN (chỉ tunnel mạng), PAM thêm lớp identity + authorization + recording lên trên:

| Khía cạnh | VPN truyền thống | PAM |
|---|---|---|
| Ai truy cập | IP-based | Identity-based (user + MFA) |
| Truy cập gì | Cả network | Chỉ server được authorize |
| Audit | Connection log | Full session recording |
| Credential | User tự giữ | Vault — user không bao giờ thấy password |
| Revoke | Block IP | Disable account, instant |

## JumpServer — Kiến trúc tổng thể

JumpServer implement mô hình **Bastion Host** (hay "jump box") — một server trung gian mà mọi truy cập đều phải đi qua:

```
User → [Browser/Terminal] → JumpServer → Target Server
         ↓
    Identity + MFA + Authorization + Recording
```

### Core Components

1. **Core Service** — Backend Python/Django, xử lý authentication, authorization, session management
2. **Coco/WS** — SSH/RDP proxy server, ghi lại session
3. **Luna** — Web Terminal, kết nối trực tiếp từ browser
4. **KoKo** — Next-gen SSH/Telnet server (thay thế Coco)
5. **Magnus** — RDP/VNC proxy (MySQL/Redis/MariaDB)
6. **Engine** — Record và replay session (Guacamole cho RDP, asciinema cho SSH)

### Asset Management

JumpServer quản lý **Assets** (server, database) theo **Node** tree:

```
Production
├── Web Servers
│   ├── web-01.example.com (SSH :22)
│   └── web-02.example.com (SSH :22)
├── Database
│   ├── db-master (SSH :22 + MySQL :3306)
│   └── db-slave (SSH :22)
└── Internal Tools
    └── jenkins.internal (SSH :22)
```

Mỗi asset có:
- **System User** — account trên target server (e.g., `deploy`, `root`, `admin`)
- **Admin User** — privileged account để JumpServer push key/ thay đổi password
- **Labels** — tagging cho filtering (`env:prod`, `team:backend`)
- **Platform** — Linux/Windows/MySQL/Redis/...

## Tính năng chính

### 1. Authentication & Authorization

JumpServer dùng **RBAC** (Role-Based Access Control):

```
User → Role → Permission → Asset (via Node)
```

- **User**: người dùng thực (SSO/LDAP/local)
- **User Group**: nhóm user (e.g., `devops-team`, `backend-squad`)
- **Asset Permission**: rule grant `User Group` → `Asset Node` với `System User` và `Actions`:
  - `connect` — mở terminal
  - `upload` — SCP upload
  - `download` — SCP download
  - `updownload` — cả hai

### 2. Session Recording & Replay

Mỗi SSH/RDP session đều được ghi lại:

- **SSH**: Record terminal output (asciinema format) — có thể replay như video
- **RDP**: Screen recording (Guacamole protocol)
- **Metadata**: Command history, file transfer, clipboard share
- **Real-time monitor**: Admin có thể `join` session đang diễn ra (shadow session)

> **Compliance**: Session recording là yêu cầu bắt buộc cho PCI-DSS, SOC 2, ISO 27001. JumpServer đáp ứng tất cả.

### 3. Credential Vault

System User có thể cấu hình ở hai chế độ:

- **Manual**: Admin set password/key cố định — user không thấy
- **Auto**: JumpServer tự rotate password định kỳ (password rotation)

User **không bao giờ** nhìn thấy password thực. Khi kết nối:
1. User chọn asset → click "Connect"
2. JumpServer vault lấy credential → proxy SSH với credential đó
3. User chỉ thấy terminal, không thấy password

### 4. Command Filter & ACL

Admin định nghĩa **Command Filter**:
- **Allow list**: Chỉ cho phép `git`, `docker`, `kubectl` — block mọi thứ khác
- **Block list**: Block `rm -rf /`, `shutdown`, `dd`

Command filter chạy **real-time** — nếu user gõ command bị block, session log flag ngay.

### 5. Database Proxy

JumpServer không chỉ proxy SSH/RDP — nó còn proxy database:

- **MySQL/MariaDB**: User connect qua JumpServer, không cần direct access
- **Redis**: Tương tự
- **PostgreSQL**: Supported
- **Oracle/SQL Server**: Supported (Enterprise)

Query cũng được audit log.

## Deployment thực tế

### Docker Compose (Quick Start)

```yaml
version: '3.8'
services:
  jumpserver:
    image: jumpserver/jms_all:latest
    container_name: jumpserver
    ports:
      - "80:80"     # Web UI
      - "2222:2222" # SSH Proxy
    environment:
      SECRET_KEY: <random-32-bytes>
      BOOTSTRAP_TOKEN: <random-token>
      DB_HOST: mysql
      DB_PASSWORD: <mysql-password>
      REDIS_HOST: redis
    volumes:
      - jumpserver_data:/opt/data
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: <mysql-password>
      MYSQL_DATABASE: jumpserver
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7-alpine

volumes:
  jumpserver_data:
  mysql_data:
```

### Production Checklist

| Item | Why |
|---|---|
| **HTTPS only** (reverse proxy nginx) | Session recording contains sensitive data |
| **MFA bắt buộc** | Password alone không đủ |
| **Backup MySQL + Redis** | Mất DB = mất toàn bộ audit log |
| **Network isolation** | JumpServer chỉ accessible từ internal network |
| **Disk space cho recording** | 1 SSH session ~1-10MB, RDP có thể 100MB+ |
| **Regular credential rotation** | Auto-rotate system user passwords |
| **Command filtering** | Block destructive commands trên production |
| **SSO/LDAP integration** | Không tạo account thủ công, revoke nhanh |

## JumpServer vs Alternatives

| | JumpServer | Teleport | Apache Guacamole | Teleport |
|---|---|---|---|---|
| License | AGPLv3 (free) | Apache 2.0 | Apache 2.0 | Apache 2.0 |
| SSH Proxy | ✅ | ✅ | ✅ | ✅ |
| RDP Proxy | ✅ | ❌ | ✅ | ❌ |
| Web UI | ✅ Full | ✅ | ✅ | ✅ |
| Session Record | ✅ | ✅ | ✅ | ✅ |
| DB Proxy | ✅ MySQL/Redis | ✅ MySQL/PG/Redis | ❌ | ✅ |
| LDAP/SSO | ✅ | ✅ OIDC/SAML | ❌ | ✅ |
| Auto Password Rotation | ✅ | ❌ | ❌ | ❌ |
| Language | Python/Django | Go | Java | Go |
| **Điểm mạnh** | PAM đầy đủ, UI đẹp, RDP support | Short-lived certs, Kubernetes-native | RDP/VNC only | Mạnh nhất cho cloud-native |

> **Khi nào dùng JumpServer**: Hạ tầng mixed (Linux + Windows + DB), cần compliance audit, team quen thuộc PAM truyền thống.

## Kết luận

JumpServer biến quản lý truy cập server từ "shared SSH key trong 1Password" thành một hệ thống PAM đầy đủ: identity, authorization, recording, credential vault, command filtering. Trong môi trường production với compliance requirements, đây là must-have.

So với giải pháp thương mại (CyberArk, BeyondTrust), JumpServer open-source với AGPLv3 cho phép tự host, không vendor lock-in, community active. Trade-off là phải tự vận hành — nhưng với Docker deployment, nó khá dễ quản lý.
