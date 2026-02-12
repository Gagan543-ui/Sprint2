# Redis Server Installation & Configuration Guide

---

## Document Control

| Author          | Created    | Version | Last updated by | Last Edited On | L0 Reviewer  | L1 Reviewer | L2 Reviewer |
| --------------- | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- |
| Shreyas Awasthi | 2026-02-12 | 1.0     | Shreyas Awasthi | 2026-02-12     | Divya Mishra | Pritam      |             |

---

# Table of Contents

1. [Introduction](#1-introduction)  
2. [What is Redis?](#2-what-is-redis)  
3. [Why We Use Redis?](#3-why-we-use-redis)  
4. [Scope](#4-scope)  
5. [Prerequisites](#5-prerequisites)  
6. [Installation & Configuration Steps](#6-installation--configuration-steps)  
   - 6.1 [Update Package Repository](#61-update-package-repository)  
   - 6.2 [Install Redis Server](#62-install-redis-server)  
   - 6.3 [Configure Redis](#63-configure-redis)  
   - 6.4 [Verify Service Status](#64-verify-service-status)  
7. [Functional Validation](#7-functional-validation)  
8. [Service Management Commands](#8-service-management-commands)  
9. [Troubleshooting](#9-troubleshooting)  
10. [Final Outcome](#10-final-outcome)  
11. [Conclusion](#11-conclusion)  
12. [Contact Information](#12-contact-information)  
13. [References](#13-references)  

---

## 1. Introduction

This document provides a step-by-step procedure to install, configure, and validate **Redis Server (v6.x)** on **Ubuntu 22.04 (Jammy)** running on an EC2 instance.

---
## 2. What is Redis?
Redis (Remote Dictionary Server) is an open-source, in-memory data store used as a database, cache, and message broker.
Unlike traditional databases that store data on disk, Redis primarily stores data in memory. Because data is stored in RAM, Redis delivers extremely fast read and write operations.

---
## 3. Why we use Redis?
Redis is commonly used to improve application performance and scalability.

Key use cases include:

- Caching: Stores frequently accessed data to reduce load on the main database.
- Session Management: Manages user sessions in web applications.
- Real-Time Analytics: Supports counters and tracking systems.
- Message Queues: Handles background jobs and asynchronous tasks.
- Rate Limiting: Controls API request frequency.

---
## 4. Scope

This SOP covers the end-to-end installation, configuration, and validation of Redis Server on an Ubuntu 22.04 LTS environment hosted on an AWS EC2 instance. The document is specifically aligned to:

- Ubuntu 22.04 LTS (Jammy Jellyfish) operating system
- AWS EC2 virtual machine infrastructure
- Redis Server version 6.x (APT repository package)
- Default Redis service port: 6379

The scope includes package installation, configuration file modification, service management using systemd, and basic functional validation. It does not cover advanced clustering, replication, or high-availability configurations.

---

## 5. Prerequisites

Prior to initiating the installation and configuration process, ensure that the following technical and access requirements are fulfilled:
- An active AWS EC2 instance running Ubuntu 22.04 LTS.
- Secure Shell (SSH) access to the server with verified connectivity.
- A user account with sudo privileges to perform administrative operations.
- Stable internet connectivity to download packages from official Ubuntu repositories.
- System resources meeting minimum operational requirements (recommended: ≥1 GB RAM for basic workloads).
- Security Group and/or firewall rules reviewed and configured appropriately if Redis is intended to accept remote connections on port 6379.

Basic familiarity with Linux command-line operations and systemd service management.

---

## 6. Installation & Configuration Steps

### 6.1 Update Package Repository

Update the system package index:

```bash
sudo apt update
```

<img width="600" height="800" alt="configuration" src="https://github.com/user-attachments/assets/7d5168c2-7ac9-4f61-9a6e-9bc0ebb56c8d" />



### 6.2 Install Redis Server

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



### 6.3 Configure Redis

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


### 6.4 Verify Service Status
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

## 7. Functional Validation

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

## 8. Service Management Commands

| Action          | Command                        |
| --------------- | ------------------------------ |
| Start Service   | `sudo systemctl start redis`   |
| Stop Service    | `sudo systemctl stop redis`    |
| Restart Service | `sudo systemctl restart redis` |
| Check Status    | `sudo systemctl status redis`  |
| Enable at Boot  | `sudo systemctl enable redis`  |

---

## 9. Troubleshooting

If Redis fails to start, review logs:

```bash
sudo journalctl -u redis-server -n 50 --no-pager
```
---

## 10. Final Outcome

Redis server successfully installed and configured.

* Listening on: `0.0.0.0:6379`
* Service Status: Active and Running
* Ready to accept client connections

---

## 11. Conclusion

Redis has been successfully deployed and validated on Ubuntu 22.04. The service is operational, configured as per requirements, and ready for integration with application workloads.

---

## 12. Contact Information

| Name            | Team    | Contact Type | Details                                                                                 |
| --------------- | ------- | ------------ | --------------------------------------------------------------------------------------- |
| Shreyas Awasthi | Saarthi | Email        | [shreyas.awasthi.snaatak@mygurukulam.CO](mailto:shreyas.awasthi.snaatak@mygurukulam.CO) |

---

## 13. References

| Descriptions                 | Links                                                                                        |
| ---------------------------- | -------------------------------------------------------------------------------------------- |
| Redis Official Documentation | [Redis Documentation](https://redis.io/docs/)                                                |
| Redis Configuration Guide    | [Redis Configuration](https://redis.io/docs/latest/operate/oss_and_stack/management/config/) |
| Redis Security Guide         | [Redis Security](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)    |
| Redis CLI Documentation      | [Redis CLI](https://redis.io/docs/latest/develop/tools/cli/)                                 |

---



