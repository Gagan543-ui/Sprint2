# Redis Server Installation & Configuration Guide

---

## Document Control

| Author          | Created    | Version | Last updated by | Last Edited On | L0 Reviewer  | L1 Reviewer | L2 Reviewer |
| --------------- | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- |
| Shreyas Awasthi | 2026-02-12 | 1.0     | Shreyas Awasthi | 2026-02-12     | Divya Mishra | Pritam      |             |

---

# Table of Contents

1. [Overview](#1-overview)
2. [Scope](#2-scope)
3. [Prerequisites](#3-prerequisites)
4. [Installation & Configuration Steps](#4-installation--configuration-steps)

   * 4.1 [Update Package Repository](#41-update-package-repository)
   * 4.2 [Install Redis Server](#42-install-redis-server)
   * 4.3 [Configure Redis](#43-configure-redis)
   * 4.4 [Verify Service Status](#44-verify-service-status)
5. [Functional Validation](#5-functional-validation)
6. [Service Management Commands](#6-service-management-commands)
7. [Troubleshooting](#7-troubleshooting)
8. [Final Outcome](#8-final-outcome)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---

## 1. Introduction

This document provides a step-by-step procedure to install, configure, and validate **Redis Server (v6.x)** on **Ubuntu 22.04 (Jammy)** running on an EC2 instance.

---

## 2. Scope

* Ubuntu 22.04 LTS
* AWS EC2 Instance
* Redis 6.x
* Default Port: 6379

---

## 3. Prerequisites

Before proceeding, ensure the following:

* SSH access to the Ubuntu server
* User with `sudo` privileges
* Internet connectivity enabled
* Security Group configured (Port 6379 open if remote access is required)

---

## 4. Installation & Configuration Steps

---

### 4.1 Update Package Repository

Update the system package index:

```bash
sudo apt update
```

<img width="600" height="800" alt="configuration" src="https://github.com/user-attachments/assets/7d5168c2-7ac9-4f61-9a6e-9bc0ebb56c8d" />


---

### 4.2 Install Redis Server

Install Redis using the APT package manager:

```bash
sudo apt install redis-server -y
```
<img width="600" height="800" alt="redis-installation" src="https://github.com/user-attachments/assets/6f23e686-85d8-41a1-bd76-5f2998405589" />


#### Verify Installation

```bash
redis-server --version
```
<img width="600" height="300" alt="Screenshot from 2026-02-12 12-17-34" src="https://github.com/user-attachments/assets/014c253d-4170-4826-8956-38e1c931fc76" />


---

### 4.3 Configure Redis

Open the Redis configuration file:

```bash
sudo nano /etc/redis/redis.conf
```

Modify the following parameters as required:

```conf
bind 0.0.0.0
port 6379
```
<img width="600" height="800" alt="configuration-file" src="https://github.com/user-attachments/assets/bc33edc2-98f9-4631-af3f-8c02d1413ee6" />


---
### 4.4 Verify Service Status
Apply configuration changes by restarting Redis:

```bash
sudo systemctl restart redis
```
Check if Redis is running:

```bash
sudo systemctl status redis
```

Expected status:

```
Active: active (running)
Status: "Ready to accept connections"
```
<img width="600" height="800" alt="restart" src="https://github.com/user-attachments/assets/70e69fe8-026b-486d-8989-e67d2f8ea3d0" />

---

## 5. Functional Validation

Test Redis using the CLI tool:

```bash
redis-cli ping
```

Expected response:

```
PONG
```
<img width="600" height="800" alt="Screenshot from 2026-02-12 12-34-14" src="https://github.com/user-attachments/assets/0f5ba2cf-ff7e-4046-8a9f-b5d171f87908" />

---

## 6. Service Management Commands

| Action          | Command                        |
| --------------- | ------------------------------ |
| Start Service   | `sudo systemctl start redis`   |
| Stop Service    | `sudo systemctl stop redis`    |
| Restart Service | `sudo systemctl restart redis` |
| Check Status    | `sudo systemctl status redis`  |
| Enable at Boot  | `sudo systemctl enable redis`  |

---

## 7. Troubleshooting

If Redis fails to start, review logs:

```bash
sudo journalctl -u redis-server -n 50 --no-pager
```
---

## 8. Final Outcome

Redis server successfully installed and configured.

* Listening on: `0.0.0.0:6379`
* Service Status: Active and Running
* Ready to accept client connections

---

## 9. Conclusion

Redis has been successfully deployed and validated on Ubuntu 22.04. The service is operational, configured as per requirements, and ready for integration with application workloads.

---

## 10. Contact Information

| Name            | Team    | Contact Type | Details                                                                                 |
| --------------- | ------- | ------------ | --------------------------------------------------------------------------------------- |
| Shreyas Awasthi | Saarthi | Email        | [shreyas.awasthi.snaatak@mygurukulam.CO](mailto:shreyas.awasthi.snaatak@mygurukulam.CO) |

---

## 11. References

| Descriptions                 | Links                                                                                        |
| ---------------------------- | -------------------------------------------------------------------------------------------- |
| Redis Official Documentation | [Redis Documentation](https://redis.io/docs/)                                                |
| Redis Configuration Guide    | [Redis Configuration](https://redis.io/docs/latest/operate/oss_and_stack/management/config/) |
| Redis Security Guide         | [Redis Security](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)    |
| Redis CLI Documentation      | [Redis CLI](https://redis.io/docs/latest/develop/tools/cli/)                                 |

---



