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
   * 4.4 [Restart Redis Service](#44-restart-redis-service)
   * 4.5 [Verify Service Status](#45-verify-service-status)
5. [Functional Validation](#5-functional-validation)
6. [Service Management Commands](#6-service-management-commands)
7. [Troubleshooting](#7-troubleshooting)
8. [Security Recommendations (Production)](#8-security-recommendations-production)
9. [Final Outcome](#9-final-outcome)
10. [Conclusion](#10-conclusion)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

## 1. Overview

This document provides a step-by-step procedure to install, configure, and validate **Redis Server (v6.x)** on **Ubuntu 22.04 (Jammy)** running on an EC2 instance.

This guide is intended for DevOps engineers and system administrators requiring a clean and production-ready Redis setup.

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

This ensures the latest package metadata is available before installation.

---

### 4.2 Install Redis Server

Install Redis using the APT package manager:

```bash
sudo apt install redis-server -y
```

#### Verify Installation

```bash
redis-server --version
```

Expected output should display Redis version information.

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

#### Important Notes

* Avoid writing inline comments on the same line as configuration values.
* Use `0.0.0.0` only when external access is required.
* Ensure firewall and security group rules are properly configured before exposing Redis externally.

Save and exit the file after making changes.

---

### 4.4 Restart Redis Service

Apply configuration changes by restarting Redis:

```bash
sudo systemctl restart redis
```

---

### 4.5 Verify Service Status

Check if Redis is running:

```bash
sudo systemctl status redis
```

Expected status:

```
Active: active (running)
Status: "Ready to accept connections"
```

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

### Common Issue

Error:

```
Could not create server TCP listening socket
```

**Cause:** Inline comments in configuration file.

**Resolution:**

Ensure comments are placed on separate lines only.

---

## 8. Security Recommendations (Production)

For production deployments, consider:

* Binding to specific private IP instead of `0.0.0.0`
* Enabling authentication (`requirepass`)
* Configuring firewall rules
* Using Redis over a private VPC network
* Enabling TLS (if applicable)

---

## 9. Final Outcome

Redis server successfully installed and configured.

* Listening on: `0.0.0.0:6379`
* Service Status: Active and Running
* Ready to accept client connections

---

## 10. Conclusion

Redis has been successfully deployed and validated on Ubuntu 22.04. The service is operational, configured as per requirements, and ready for integration with application workloads.

---

## 11. Contact Information

| Name            | Team    | Contact Type | Details                                                                                 |
| --------------- | ------- | ------------ | --------------------------------------------------------------------------------------- |
| Shreyas Awasthi | Saarthi | Email        | [shreyas.awasthi.snaatak@mygurukulam.CO](mailto:shreyas.awasthi.snaatak@mygurukulam.CO) |

---

## 12. References

| Descriptions                 | Links                                                                                        |
| ---------------------------- | -------------------------------------------------------------------------------------------- |
| Redis Official Documentation | [Redis Documentation](https://redis.io/docs/)                                                |
| Redis Configuration Guide    | [Redis Configuration](https://redis.io/docs/latest/operate/oss_and_stack/management/config/) |
| Redis Security Guide         | [Redis Security](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)    |
| Redis CLI Documentation      | [Redis CLI](https://redis.io/docs/latest/develop/tools/cli/)                                 |

---



