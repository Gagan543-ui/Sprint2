# Salary API

---

---

| Author          | Created    | Version | Last updated by | Last Edited On | L0 Reviewer  | L1 Reviewer | L2 Reviewer |
| --------------- | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- |
| Shreyas Awasthi | 2026-02-04 | 1.0     | Shreyas Awasthi | 2026-02-12     | Divya Mishra |  Pritam           |             |

---
# Table of Contents

1. [Purpose](#1-purpose)  
   1.1 [Key Objectives](#11-key-objectives)  

2. [Pre-requisites](#2-pre-requisites)  

3. [System Requirements](#3-system-requirements)  

4. [Dependencies](#4-dependencies)  
   4.1 [Build Time Dependency](#41-build-time-dependency)  
   4.2 [Run Time Dependency](#42-run-time-dependency)  
   4.3 [Other Dependency](#43-other-dependency)  

5. [Application Configuration (application.yml)](#5-application-configuration-applicationyml)  

6. [Important Ports](#6-important-ports)  
   6.1 [Inbound](#61-inbound)  

7. [Step-by-Step Installation](#7-step-by-step-installation)  
   7.1 [Step 1: Install Java](#71-step-1-install-java)  
   7.2 [Step 2: Install Maven](#72-step-2-install-maven)  
   7.3 [Step 3: Clone Repository](#73-step-3-clone-repository)  
   7.4 [Step 4: Build Artifact](#74-step-4-build-artifact)  
   7.5 [Step 5: Run Cassandra Migration](#75-step-5-run-cassandra-migration)  
   7.6 [Step 6: Run Application](#76-step-6-run-application)  

8. [Deploy as Systemd Service](#8-deploy-as-systemd-service)  

9. [API Endpoints](#9-api-endpoints)  
   9.1 [Create Salary Record](#91-create-salary-record)  
   9.2 [Swagger UI](#92-swagger-ui)  
   9.3 [API Docs](#93-api-docs)  

10. [Monitoring](#10-monitoring)  
    10.1 [Actuator Endpoints](#101-actuator-endpoints)  

11. [Metrics](#11-metrics)  

12. [Health Checks](#12-health-checks)  

13. [Logging](#13-logging)  

14. [Docker Support (Optional)](#14-docker-support-optional)  
    14.1 [Build Image](#141-build-image)  
    14.2 [Push Image](#142-push-image)  

15. [Makefile](#15-makefile)  

16. [Disaster Recovery](#16-disaster-recovery)  

17. [High Availability](#17-high-availability)  

18. [Troubleshooting](#18-troubleshooting)  

19. [FAQs](#19-faqs)  

20. [Contact Information](#20-contact-information)  

21. [References](#21-references)  


## 1. Purpose

**Salary API** is a Spring Boot–based microservice designed to manage employee salary records.

### 1.1 Key Objectives

- Manage employee salary data efficiently  
- Provide REST APIs for CRUD operations  
- Integrate with ScyllaDB (Cassandra-compatible)  
- Use Redis for caching  
- Expose Actuator endpoints for monitoring  
- Provide Swagger-based API documentation  

This service ensures scalable, production-ready payroll data management with observability support.

---

## 2. Pre-requisites

Before deployment, ensure infrastructure and dependencies are in place.

---

## 3. System Requirements

| Hardware Specifications | Minimum Recommendation |
|--------------------------|------------------------|
| Processor | Dual-core |
| RAM | 4 GB |
| Disk | 20 GB |
| OS | Ubuntu 22.04 |

---

## 4. Dependencies

### 4.1 Build Time Dependency

| Name | Version | Description |
|------|----------|-------------|
| Java | 17+ | Required to run Spring Boot application |
| Maven | 3.8+ | Build & package management tool |

---

### 4.2 Run Time Dependency

| Name | Version | Description |
|------|----------|-------------|
| ScyllaDB / Cassandra | 4.x | Primary database |
| Redis | 6.x+ | Caching layer |

---

### 4.3 Other Dependency

| Name | Version | Description |
|------|----------|-------------|
| Docker | Latest | Containerization (Optional) |
| Systemd | Default | Service management |

---

## 5. Application Configuration (`application.yml`)

```yaml
server:
  port: 8082

spring:
  cassandra:
    keyspace-name: employee
    contact-points: 10.0.1.25
    port: 9042
    username: scylla
    password: 12345
    local-datacenter: datacenter1

  data:
    redis:
      host: 10.0.1.25
      port: 6379
      password: 12345

management:
  endpoints:
    web:
      base-path: /actuator
      exposure:
        include: [ "health","prometheus", "metrics" ]

  health:
    cassandra:
      enabled: true

  endpoint:
    health:
      show-details: always
    metrics:
      enabled: true
    prometheus:
      enabled: true

logging:
  level:
    org.springframework.web: DEBUG

springdoc:
  swagger-ui:
    path: /salary-documentation
    tryItOutEnabled: true
    filter: true
  api-docs:
    path: /salary-api-docs
  show-actuator: true

```
---

## 6. Important Ports

### 6.1 Inbound

| Port | Description |
|------|-------------|
| 8082 | Salary API |
| 9042 | ScyllaDB |
| 6379 | Redis |

---

## 7. Step-by-Step Installation

### 7.1 Step 1: Install Java

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

---

### 7.2 Step 2: Install Maven

```bash
sudo apt install maven -y
mvn -version
```

---

### 7.3 Step 3: Clone Repository

```bash
git clone <repository-url>
cd salary-api
```

---

### 7.4 Step 4: Build Artifact

```bash
mvn clean install -DskipTests
```

Generated JAR:

```
target/salary-0.1.0-RELEASE.jar
```

Test run:

```bash
java -jar target/salary-0.1.0-RELEASE.jar --help
```

---

### 7.5 Step 5: Run Cassandra Migration

```bash
migrate -path migration -database "cassandra://10.0.1.25:9042/employee?username=scylla&password=12345" up
```

Optional:

```sql
DROP TABLE schema_migrations;
describe tables;
```

---

### 7.6 Step 6: Run Application

```bash
java -jar target/salary-0.1.0-RELEASE.jar --server.port=8082
```

Health check:

```bash
curl http://10.0.2.75:8082/actuator/health
```

---

## 8. Deploy as Systemd Service

Create service file:

```bash
sudo nano /etc/systemd/system/salary-api.service
```

Service file:

```
[Unit]
Description=Salary API Spring Boot Service
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/salary/salary-api
ExecStart=/usr/bin/java -jar target/salary-0.1.0-RELEASE.jar --server.port=8082
SuccessExitStatus=143
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Reload & Start:

```bash
sudo systemctl daemon-reload
sudo systemctl start salary-api
sudo systemctl enable salary-api
sudo systemctl status salary-api
```

Verify port:

```bash
ss -tulpn | grep 8082
```

---

## 9. API Endpoints

### 9.1 Create Salary Record

```bash
curl http://10.0.2.75:8082/api/v1/salary/create/record
```

### 9.2 Swagger UI

```
http://10.0.2.75:8082/salary-documentation
```

### 9.3 API Docs

```
http://10.0.2.75:8082/salary-api-docs
```

---

## 10. Monitoring

### 10.1 Actuator Endpoints

- `/actuator/health`
- `/actuator/metrics`
- `/actuator/prometheus`

---

## 11. Metrics

| Parameter | Priority | Threshold |
|------------|----------|------------|
| Disk Utilization | High | > 90% |
| Availability | High | >= 99.9% |
| Memory Utilization | Medium | > 80% |
| CPU Utilization | Medium | > 70% |
| Latency | High | < 300ms |
| Errors | High | > 5 per minute |
| Throughput | High | > 1000 requests/min |

---

## 12. Health Checks

| Name | Type | InitialDelaySeconds | PeriodSeconds | TimeoutSeconds | SuccessThreshold | FailureThreshold |
|------|------|--------------------|--------------|----------------|-----------------|-----------------|
| Salary API | ReadinessProbe | 10 | 10 | 5 | 1 | 3 |
| Salary API | LivenessProbe | 10 | 10 | - | 5 | 1 |

---

## 13. Logging

```yaml
logging:
  level:
    org.springframework.web: DEBUG
```

Log Categories:

- Event Logs  
- Access Logs  
- Server Logs  
- Threat Logs  

---

## 14. Docker Support (Optional)

### 14.1 Build Image

```bash
make docker-build
```

### 14.2 Push Image

```bash
make docker-push
```

---

## 15. Makefile

```makefile
APP_VERSION ?= v0.1.0
IMAGE_REGISTRY ?= quay.io/opstree
IMAGE_NAME ?= salary-api

build:
	mvn clean package

fmt:
	mvn checkstyle:checkstyle

test:
	mvn test

docker-build:
	docker build -t ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION} -f Dockerfile .

docker-push:
	docker push ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION}

run-migrations:
	migrate -path migration -database "cassandra://10.0.1.25:9042/employee?username=scylla&password=12345" up
```

---

## 16. Disaster Recovery

- Regular ScyllaDB backups  
- Redis persistence enabled  
- Artifact backup in registry  
- Infrastructure as Code recommended  

---

## 17. High Availability

- Deploy multiple instances  
- Use Load Balancer  
- Enable Redis clustering  
- Use ScyllaDB cluster mode  

> DR focuses on recovery.  
> HA focuses on preventing downtime.

---

## 18. Troubleshooting

| Issue | Cause | Resolution |
|--------|--------|------------|
| Port 8082 not accessible | Firewall issue | Open port in security group |
| Cassandra connection failure | Wrong IP/credentials | Verify `application.yml` |
| Redis not connecting | Service down | Restart Redis |
| Service not starting | Incorrect Java path | Verify `/usr/bin/java` |

---

## 19. FAQs

| Question | Answer |
|----------|--------|
| Is this application free? | Yes, it is open-source. |
| Can it be deployed on any cloud platform? | Yes, it is cloud-agnostic. |
| Is an enterprise version available? | No, it is a community-driven implementation. |

---

## 20. Contact Information

| Name            | Team    | Contact Type | Details |
| --------------- | ------- | ------------ | ------- |
| Shreyas Awasthi | Saarthi | Email        | shreyas.awasthi.snaatak@mygurukulam.CO |

---

## 21. References

| Descriptions | Links |
|-------------|------|
| Spring Boot documentation | [Spring Boot Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/) |
| Git documentation | [Git Docs](https://git-scm.com/docs) |
| Documentation format reference | [AWS Documentation](https://docs.aws.amazon.com/) |

---
