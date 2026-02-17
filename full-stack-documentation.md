# OT Microservices Documentation

---
# Table of Contents

1. [Overview](#1-overview)  
2. [VPC Configuration](#2-vpc-configuration)  
3. [Subnet Design](#3-subnet-design)  

<details>
<summary><strong>4. Gateway Configuration</strong></summary>

- [4.1 Internet Gateway](#41-internet-gateway)  
- [4.2 NAT Gateway](#42-nat-gateway)

</details>

<details>
<summary><strong>5. Route Tables</strong></summary>

- [5.1 Public Route Table](#51-public-route-table)  
- [5.2 Private Route Table](#52-private-route-table)

</details>

<details>
<summary><strong>6. EC2 Instances</strong></summary>

- [6.1 Bastion Host](#61-bastion-host)  
- [6.2 Backend Server](#62-backend-server)  
- [6.3 Database Server](#63-database-server)

</details>

<details>
<summary><strong>7. SSH Access</strong></summary>

- [7.1 Local → Bastion](#71-local--bastion)

</details>

<details>
<summary><strong>8. Database Setup</strong></summary>

- [8.1 PostgreSQL](#81-postgresql)  
- [8.2 Redis](#82-redis)  
- [8.3 ScyllaDB](#scylladb)

</details>

9. [Backend Services](#9-backend-services)  

<details>
<summary><strong>10. Frontend Deployment</strong></summary>

- [10.1 Build Process](#101-build-process)  
- [10.2 Deployment Steps](#102-deployment-steps)

</details>

11. [NGINX Reverse Proxy Configuration](#11-nginx-reverse-proxy-configuration)  
12. [Health Checks](#12-health-checks)  
13. [Security Measures](#13-security-measures)  
14. [Architecture Flow](#14-architecture-flow)  
15. [Conclusion](#15-conclusion)  
16. [Contact Information](#16-contact-information)  
17. [References](#17-references)  
 
  
---

##  1 .Overview

This project demonstrates a complete end-to-end deployment of a secure microservices-based architecture on AWS using:

- Employee API (Go + ScyllaDB)
- Attendance API (Python + PostgreSQL + Redis)
- Salary API (Spring Boot + ScyllaDB)
- React Frontend (NGINX Reverse Proxy)
- Bastion Host for secure SSH access

Architecture follows a **multi-tier VPC design**:


---

## 2 VPC Configuration

| Resource | Name | CIDR |
|----------|------|------|
| VPC | ot_micro_vpc | 10.0.0.0/16 |

---

## 3 Subnet Design

| Subnet | AZ | CIDR | Type |
|--------|----|-------|------|
| ot-private-sub-1 | us-east-1a | 10.0.1.0/24 | Private |
| ot-private-sub-2 | us-east-1b | 10.0.2.0/24 | Private |
| ot-private-sub-3 | us-east-1c | 10.0.3.0/24 | Private |
| ot-public-sub-1 | us-east-1d | 10.0.5.0/28 | Public |

---

## 4 Gateway Configuration

### 4.1 Internet Gateway
- Name: `ot-micro-gw`
- Attached to: `ot-public-rt`

### 4.2 NAT Gateway
- Name: `OT-nat`
- Attached to: `ot-private-rt`
- Purpose: Enables outbound internet for private subnets

---

## 5 Route Tables

### 5.1 Public Route Table
Associated with:
- ot_micro_subnet_entry
- ot_micro_subnet_frontend

Route: 0.0.0.0/0 → Internet Gateway


### 5.2 Private Route Table
Associated with:
- ot_micro_subnet_api
- ot_micro_subnet_db

Route: 0.0.0.0/0 → NAT Gateway


---

## 6. EC2 Instances

### 6.1 Bastion Host

| Parameter | Value |
|------------|--------|
| AMI | Ubuntu 22.04 |
| Type | t3.micro |
| Subnet | Public |
| Purpose | SSH Access |

---

### 6.2 Backend Server

| Parameter | Value |
|------------|--------|
| AMI | Ubuntu 22.04 |
| Type | c7i-flex.large |
| Subnet | Private |

Runs:
- Employee API (8080)
- Attendance API (8081)
- Salary API (8082)

---

### 6.3 Database Server

| Parameter | Value |
|------------|--------|
| AMI | Ubuntu 22.04 |
| Type | m7i-flex.large |
| Subnet | Private |

Runs:
- PostgreSQL (5432)
- ScyllaDB (9042)
- Redis (6379)

---

## 7. SSH Access

### 7.1 Local → Bastion

```bash
chmod 400 ot_micro_service.pem
ssh -i ot_micro_service.pem ubuntu@<Bastion_Public_IP>
```
#### Bastion → Private Instances
```bash
ssh -i ot_micro_service.pem ubuntu@<Private_IP>
```
---

## 8. Database Setup

### 8.1 PostgreSQL

#### Configuration:
```bash
listen_addresses = '*'
port = 5432
```
```bash
Create DB:

CREATE DATABASE attendance_db;
ALTER USER postgres WITH PASSWORD 'password';
```

### 8.2 Redis
```bash
bind 0.0.0.0
```

Restart:
```bash
sudo systemctl restart redis-server
```
### ScyllaDB

Enable authentication in:
``` bash
/etc/scylla/scylla.yaml
```

```bash
authenticator: PasswordAuthenticator
authorizer: CassandraAuthorizer
```

## 9. Backend Services

### Employee API

###  Overview

The **Employee API** is a Golang-based microservice designed to manage employee data through scalable REST endpoints.  
It follows a microservices architecture using **Redis (cache)** and **ScyllaDB (primary database)**.

---

### Architecture Workflow

| Component        | Layer                   | Description |
|------------------|-------------------------|-------------|
| Employee API     | Application Layer       | Handles client requests and coordinates between cache and database. Acts as the control layer. |
| Redis            | Cache Layer             | In-memory store used for fast data retrieval. Reduces load on the primary database. |
| ScyllaDB         | Primary Database Layer  | Distributed NoSQL database. Serves as the source of truth for persistent employee data. |
| Migrations       | Deployment / Schema Layer | Manages schema creation and structural updates in ScyllaDB during deployment. |

#### Architecture Diagram 
<img width="700" height="700" alt="employee" src="https://github.com/user-attachments/assets/12912c76-fba9-427e-9103-efcf333f03ff" />

---
### Core Components

| Component      | Description |
|---------------|------------|
| Gin           | Lightweight HTTP framework used to build REST APIs and handle routing. |
| ScyllaDB      | Distributed NoSQL database used as the primary persistent data store. |
| Redis         | In-memory cache used to reduce database load and improve response time. |
| Prometheus    | Monitoring tool used for collecting application metrics. |
| Swagger UI    | Interactive API documentation and testing interface. |

### Pattern Used

| Pattern        | Description |
|---------------|------------|
| Cache-Aside   | API first checks Redis; on cache miss, fetches data from ScyllaDB and stores it in Redis for future use. |

---

##  System Requirements

### API Server

| Requirement | Description |
|------------|------------|
| OS         | Ubuntu 22.04 LTS |
| RAM        | Minimum 4 GB |
| Disk       | Minimum 20 GB |
| CPU        | Dual-core processor |

### Database Server

| Requirement | Description |
|------------|------------|
| OS         | Ubuntu 22.04 LTS |
| RAM        | Minimum 8 GB |
| Disk       | Minimum 50 GB |
| CPU        | Dual-core processor |

---

##  Dependencies

### Build-Time Dependencies

| Dependency | Description |
|-----------|------------|
| Go        | Programming language used to develop the API. |
| Make      | Automation tool used for build and execution commands. |

### Run-Time Dependencies

| Dependency | Description |
|-----------|------------|
| ScyllaDB  | Primary database storing employee records. |
| Redis     | Optional caching layer for performance optimization. |
| Prometheus| Metrics collection and monitoring system. |
| Swagger   | API documentation and testing tool. |

---

##  Important Endpoints & Ports

| Port / Endpoint | Description |
|----------------|------------|
| 8080           | Application HTTP server port. |
| 9042           | ScyllaDB database port. |
| /swagger/index.html | Swagger UI documentation endpoint. |
| /health        | Health check endpoint for service monitoring. |

---

##  Monitoring

### Key Metrics

| Metric | Description |
|--------|------------|
| CPU Utilization | Measures processor usage of the API service. |
| Memory Utilization | Tracks memory consumption levels. |
| Disk Usage | Monitors disk space utilization. |
| Latency | Measures API response time (< 300ms recommended). |
| Availability | Ensures uptime target of ≥ 99.9%. |

### Log Files

| Log File Path | Description |
|--------------|------------|
| /var/log/employee-api/server.log | Application server logs. |
| /var/log/employee-api/access.log | Client access logs. |
| /var/log/employee-api/threat.log | Security and threat-related logs. |

---

##  High Availability & Disaster Recovery

| Strategy | Description |
|----------|------------|
| Load Balancer | Distributes traffic across multiple API instances. |
| ScyllaDB Replication | Ensures data redundancy and fault tolerance. |
| Redis Persistence | Maintains cached data durability where required. |
| Regular Backups | Protects against data loss during failures. |

---
## Salary API

The **Salary API** is a Spring Boot–based microservice responsible for managing employee salary records.  
It is designed for scalability, observability, and high availability within a microservices ecosystem.

##  System Requirements

| Requirement | Minimum |
|-------------|---------|
| OS | Ubuntu 22.04 |
| CPU | Dual-core |
| RAM | 4 GB |
| Disk | 20 GB |

---

###  Dependencies

#### Build-Time

| Dependency | Version | Purpose |
|------------|----------|----------|
| Java | 17+ | Runtime for Spring Boot |
| Maven | 3.8+ | Build & dependency management |

#### Run-Time

| Dependency | Version | Purpose |
|------------|----------|----------|
| ScyllaDB / Cassandra | 4.x | Primary database |
| Redis | 6.x+ | Caching layer |

---

###  Makefile

**Purpose:** Automates build, test, and Docker operations.  
Ensures consistency across developers and CI/CD pipelines using simple commands like:

```bash
make build
make test
make run
```


---

###  Architecture Workflow 


| Step | Title                        | Description |
|------|-----------------------------|------------|
| 1    | API checks Redis            | When a request comes to the Salary API, it first checks Redis to see if the required salary data is already available in cache. |
| 2    | Cache Hit                   | If the data is found in Redis, it is returned immediately. ScyllaDB is not used, so the response is fast. |
| 3    | Cache Miss                  | If the data is not found in Redis, the Salary API queries ScyllaDB to get the required salary data. |
| 4    | Database Response           | ScyllaDB returns the requested data to the Salary API. ScyllaDB acts as the main source of truth. |
| 5    | Store in Cache              | After receiving the data from ScyllaDB, the Salary API stores a copy in Redis so that future requests can be served faster. |
| 6    | Migrations                  | Migrations are used to create and update tables or schema in ScyllaDB. They run during deployment and are not part of the runtime request flow. |

#### Architecture Diagram 
<img width="700" height="700" alt="salary" src="https://github.com/user-attachments/assets/280b5057-162d-4c84-b34c-40574f3beecf" />


---

##  Important Ports

| Port | Service |
|------|---------|
| 8082 | Salary API |
| 9042 | ScyllaDB |
| 6379 | Redis |

---

##  Monitoring & Health

### Actuator Endpoints

- `/actuator/health`
- `/actuator/metrics`
- `/actuator/prometheus`

### Key Metrics

| Metric | Threshold |
|--------|------------|
| Disk Usage | > 90% |
| CPU Usage | > 70% |
| Memory Usage | > 80% |
| Latency | < 300 ms |
| Availability | ≥ 99.9% |

---

##  Logging

Configured via `application.yml`.

**Log Types:**
- Event Logs – Application lifecycle events  
- Access Logs – Incoming API requests  
- Server Logs – Internal processing  
- Threat Logs – Security-related events  

Example:

```yaml
logging:
  level:
    org.springframework.web: DEBUG

```

---
## Attendance API

---

The **Attendance API** is a Python (Flask)-based microservice for managing employee attendance records.  
It is designed with a cloud-ready architecture using PostgreSQL for persistence and Redis for caching.

---


###  Purpose

#### Business
- Manage employee attendance records.
- Provide APIs to create, search, and retrieve attendance data.
- Enable health and metrics monitoring.

#### Technical
- PostgreSQL for persistent storage.
- Redis for caching.
- Liquibase for database migrations.
- Gunicorn + Systemd for deployment.

---

###  System & Software Requirements

| Component | Minimum |
|------------|----------|
| OS | Ubuntu 22.04 LTS |
| CPU | Quad-core |
| RAM | 4 GB (8 GB recommended) |
| Disk | 20 GB |
| Python | 3.11 |
| PostgreSQL | 16 |
| Redis | 7.x |
| Liquibase | 4.24.0 |

---

###  Architecture Workflow 

| Step | Title                     | Description |
|------|---------------------------|------------|
| 1    | API checks Redis          | When a request comes, the Attendance API first checks Redis to see if the data is already there. |
| 2    | Cache Hit                 | If the data is found in Redis, Redis sends it back immediately. The database is not used, so the response is fast. |
| 3    | Cache Miss                | If the data is not found in Redis, the API asks PostgreSQL for the data. |
| 4    | Database Response         | PostgreSQL returns the data to the API. |
| 5    | Store in Cache            | The API stores a copy of the data in Redis so that next time it can be returned quickly. |


#### Architecture Diagram 
<img width="700" height="700" alt="Screenshot from 2026-02-17 19-02-18" src="https://github.com/user-attachments/assets/b4eb8535-ff53-4047-8205-2724fa80061f" />

---

### Components
- Bastion Host (Public Subnet)
- DB Server (PostgreSQL + Redis)
- API Server (Attendance API)

### Flow
Client → API (8080) → Redis (cache check)  
                     → PostgreSQL (on cache miss)

### Monitoring Endpoints
- `/api/v1/attendance/health`
- `/api/v1/attendance/health/detail`
- `/metrics`
- `/apidocs` (Swagger)

---
##  Dependencies

### Runtime

| Dependency | Purpose |
|------------|----------|
| PostgreSQL | Primary relational database |
| Redis | Caching layer |
| Gunicorn | WSGI server |
| Liquibase | Database migrations |

### Other

| Tool | Purpose |
|------|----------|
| make | Build automation |
| pylint | Code linting |
| default-jre | Required for Liquibase |

---

##  Important Ports

| Port | Service |
|------|----------|
| 22 | SSH |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | Attendance API |

---

##  Deployment Summary

### Infrastructure
- VPC with Public (Bastion) & Private (DB/API) subnets
- Secure Security Group rules between API and DB

### Database Setup
- Install PostgreSQL 16
- Enable remote access
- Create `attendance_db`
- Install & configure Redis

---

## 12. Health Checks

| Service         | Health Endpoint                     | Port  | Purpose                |
|----------------|--------------------------------------|-------|------------------------|
| Employee API   | `/api/v1/employee/health`            | 8080  | Service availability   |
| Attendance API | `/api/v1/attendance/health`          | 8081  | Service availability   |
| Salary API     | `/actuator/health`                   | 8082  | Spring Boot health     |
| Frontend       | `http://<Frontend_Public_IP>`        | 80    | UI accessibility check |

### Sample Commands

```bash
curl http://localhost:8080/api/v1/employee/health
curl http://localhost:8081/api/v1/attendance/health
curl http://localhost:8082/actuator/health
```

---

## 13. Security Measures

| Control Area                 | Implementation Detail                          | Objective                              |
|------------------------------|-----------------------------------------------|----------------------------------------|
| Network Isolation            | Private subnet architecture                   | Restrict direct public access         |
| Outbound Traffic Control     | NAT Gateway                                   | Secure internet access from private subnet |
| SSH Access Control           | Bastion host only                             | Centralized administrative access     |
| API Access Management        | NGINX reverse proxy                           | Eliminates CORS and centralizes routing |
| Database Security            | Authentication enabled                        | Enforce credential-based access       |
| Database Exposure            | No public DB ports                            | Prevent external database access      |

## 14. Architecture Flow

<img width="600" height="600" alt="Screenshot from 2026-02-15 14-21-50" src="https://github.com/user-attachments/assets/b85af3b1-4be2-4b55-8da9-08190b501c0a" />

---

## 15. Conclusion 
This project demonstrates a structured, production-oriented microservices deployment on AWS using network segmentation, secure access controls, reverse proxy routing, and service-level isolation.

---

## 16. Contact Information
| Name            | Team    | Contact Type | Details |
| --------------- | ------- | ------------ | ------- |
| Shreyas Awasthi | Saarthi | Email        | shreyas.awasthi.snaatak@mygurukulam.CO |

---

## 17. References

| Description | Link |
|-------------|------|
| Notification Service Documentation | [Notification README](https://github.com/Snaatak-Saarthi/Saarthi_Sprint1/blob/SCRUM-85-suraj/OT_MS_Understanding/Notification/Documentation/README.md) |
| Salary Service Documentation | [Salary README](https://github.com/Snaatak-Saarthi/Saarthi_Sprint1/blob/SCRUM-99-shreyas/OT_MS_Understanding/Salary/Documentation/README.md) |
| Redis Documentation | [Redis README](https://github.com/Snaatak-Saarthi/Saarthi_Sprint1/blob/SCRUM-106-abhinav/OT_MS_Understanding/Redis/Documentation/README.md) |
| PostgreSQL Documentation | [PostgreSQL README](https://github.com/Snaatak-Saarthi/Saarthi_Sprint1/blob/SCRUM-111-mukesh/OT_MS_Understanding/PostgreSQL/Documentation/README.md) |

---
