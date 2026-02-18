# OT Microservices Documentation

---

# Table of Contents

1. [Overview](#1-overview)  
2. [VPC Configuration](#2-vpc-configuration)  
3. [Subnet Design](#3-subnet-design)  

<details>
<summary><strong>4. Gateway Configuration</strong></summary>

- 4.1 [Internet Gateway](#41-internet-gateway)  
- 4.2 [NAT Gateway](#42-nat-gateway)

</details>

<details>
<summary><strong>5. Route Tables</strong></summary>

- 5.1 [Public Route Table](#51-public-route-table)  
- 5.2 [Private Route Table](#52-private-route-table)

</details>

6. [Backend Services](#9-backend-services)

<details>
<summary><strong>6.1 Employee API</strong></summary>

- 6.1.1 [Overview](#overview)  
- 6.1.2 [Architecture](#architecture)  
- 6.1.3 [Core Components](#core-components)  
- 6.1.4 [Pattern Used](#pattern-used)  
- 6.1.5 [System Requirements](#system-requirements)  
- 6.1.6 [Dependencies](#dependencies)  
- 6.1.7 [Important Endpoints & Ports](#important-endpoints--ports)  
- 6.1.8 [Monitoring](#monitoring)  
- 6.1.9 [High Availability & Disaster Recovery](#high-availability--disaster-recovery)

</details>

<details>
<summary><strong>6.2 Salary API</strong></summary>

- 6.2.1 [System Requirements](#system-requirements-1)  
- 6.2.2 [Dependencies](#dependencies-1)  
- 6.2.3 [Makefile](#makefile)  
- 6.2.4 [Architecture](#architecture-1)  
- 6.2.5 [Important Ports](#important-ports)  
- 6.2.6 [Monitoring & Health](#monitoring--health)  
- 6.2.7 [Logging](#logging)

</details>

<details>
<summary><strong>6.3 Attendance API</strong></summary>

- 6.3.1 [Purpose](#purpose)  
- 6.3.2 [System & Software Requirements](#system--software-requirements)  
- 6.3.3 [Architecture](#architecture-2)  
- 6.3.4 [Dependencies](#dependencies-2)  
- 6.3.5 [Important Ports](#important-ports-1)  
- 6.3.6 [Deployment Summary](#deployment-summary)

</details>

7. [Health Checks](#12-health-checks)  
8. [Security Measures](#13-security-measures)  
9. [Architecture Flow](#14-architecture-flow)  
10. [Conclusion](#15-conclusion)  
11. [Contact Information](#16-contact-information)  
12. [References](#17-references)  

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



## 6. Backend Services

### 6.1 Employee API

###  6.1.1 Overview

The **Employee API** is a Golang-based microservice designed to manage employee data through scalable REST endpoints.  
It follows a microservices architecture using **Redis (cache)** and **ScyllaDB (primary database)**.

---

### 6.1.2 Architecture Workflow

| Component        | Description |
|------------------|-------------------------|
| Employee API     |  Handles client requests and coordinates between cache and database. Acts as the control layer. |
| Redis            | In-memory store used for fast data retrieval. Reduces load on the primary database. |
| ScyllaDB         | Distributed NoSQL database. Serves as the source of truth for persistent employee data. |
| Migrations       | Manages schema creation and structural updates in ScyllaDB during deployment. |

### 6.1.3 Architecture Diagram 
<img width="700" height="700" alt="employee" src="https://github.com/user-attachments/assets/12912c76-fba9-427e-9103-efcf333f03ff" />

### 6.1.4 Core Components

| Component      | Description |
|---------------|------------|
| Gin           | Lightweight HTTP framework used to build REST APIs and handle routing. |
| ScyllaDB      | Distributed NoSQL database used as the primary persistent data store. |
| Redis         | In-memory cache used to reduce database load and improve response time. |
| Prometheus    | Monitoring tool used for collecting application metrics. |
| Swagger UI    | Interactive API documentation and testing interface. |

### 6.1.5 Pattern Used

| Pattern        | Description |
|---------------|------------|
| Cache-Aside   | API first checks Redis; on cache miss, fetches data from ScyllaDB and stores it in Redis for future use. |

---

##  6.1.6 System Requirements

### 6.1.7 API Server

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

### Architecture Workflow

| Component          | Description |
|-------------------|-------------|
| API checks Redis  | When a request comes to the Salary API, it first checks Redis to see if the required salary data is already available in cache. |
| Cache Hit         | If the data is found in Redis, it is returned immediately. ScyllaDB is not used, so the response is fast. |
| Cache Miss        | If the data is not found in Redis, the Salary API queries ScyllaDB to get the required salary data. |
| Database Response | ScyllaDB returns the requested data to the Salary API. ScyllaDB acts as the main source of truth. |
| Store in Cache    | After receiving the data from ScyllaDB, the Salary API stores a copy in Redis so that future requests can be served faster. |
| Migrations        | Migrations are used to create and update tables or schema in ScyllaDB. They run during deployment and are not part of the runtime request flow. |


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
| Component             | Description |
|----------------------|-------------|
| API checks Redis     | When a request comes, the Attendance API first checks Redis to see if the data is already there. |
| Cache Hit            | If the data is found in Redis, Redis sends it back immediately. The database is not used, so the response is fast. |
| Cache Miss           | If the data is not found in Redis, the API asks PostgreSQL for the data. |
| Database Response    | PostgreSQL returns the data to the API. |
| Store in Cache       | The API stores a copy of the data in Redis so that next time it can be returned quickly. |



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

## Working of All APIs
### Architecture Workflow 

This architecture contains three APIs:

- Employee API  
- Salary API  
- Attendance API  

All APIs follow a **cache-first approach** using Redis to improve performance.

---

### Employee API (`/api/v1/employee`)

- The request goes to the Employee API.  
- The API checks Redis (cache) for the data.  
- If data is found in Redis (cache hit), it is returned immediately.  
- If data is not found (cache miss), the API fetches it from **ScyllaDB**.  
- The data is then stored in Redis for future requests.  
- The response is sent back to the user.  



### Salary API (`/api/v1/salary`)

- The request goes to the Salary API.  
- It first checks Redis.  
- If data exists in Redis, it returns the response directly.  
- If not, it fetches the data from **ScyllaDB**.  
- The result is cached in Redis.  
- The response is sent to the user.  



### Attendance API (`/api/v1/attendance`)

- The request goes to the Attendance API.  
- It checks Redis for cached data.  
- If found, the response is returned immediately.  
- If not found, the API fetches data from **PostgreSQL**.  
- The data is stored in Redis.  
- The response is sent back to the user.  


## Architecture Diagram 
<img width="700" height="700" alt="frontend" src="https://github.com/user-attachments/assets/a142407c-28a6-44b8-a45e-74f9feaf4e33" />

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
