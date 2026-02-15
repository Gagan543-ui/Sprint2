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
| VPC | ot_micro_vpc | 10.0.0.0/24 |

---

## 3 Subnet Design

| Subnet | AZ | CIDR | Type |
|--------|----|-------|------|
| ot_micro_subnet_entry | ap-south-1a | 10.0.0.0/26 | Public |
| ot_micro_subnet_api | ap-south-1b | 10.0.0.64/26 | Private |
| ot_micro_subnet_db | ap-south-1c | 10.0.0.128/26 | Private |
| ot_micro_subnet_frontend | ap-south-1b | 10.0.0.192/26 | Public |

---

## 4 Gateway Configuration

### 4.1 Internet Gateway
- Name: `ot_micro_igw`
- Attached to: `ot_micro_vpc`

### 4.2 NAT Gateway
- Name: `ot_micro_nat`
- Attached to: `ot_micro_subnet_entry`
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

| Service Name        | Technology Stack      | Port  | Database Used         | Build Command                              | Deployment Method        |
|---------------------|----------------------|-------|-----------------------|--------------------------------------------|--------------------------|
| **Employee API**    | Go                   | 8080  | ScyllaDB              | N/A                                        | systemd service          |
| **Attendance API**  | Python + Flask       | 8081  | PostgreSQL + Redis    | N/A                                        | Gunicorn + systemd       |
| **Salary API**      | Spring Boot (Java)   | 8082  | ScyllaDB              | `./mvnw clean package -DskipTests`         | systemd (JAR execution)  |

---

Run:
```bash
java -jar target/salary-0.1.0-RELEASE.jar --server.port=8082

```

## 10. Frontend Deployment

### Technology Stack
- React
- NodeJS
- NGINX

---

### 10.1 Build Process

```bash
npm install
npm run build
```

---

### 10.2 Deployment Steps

```bash
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
```

---

## 11. NGINX Reverse Proxy Configuration

### Configuration File Location

```
/etc/nginx/sites-available/default
```

### Server Configuration

```nginx
server {
    listen 80 default_server;
    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/v1/employee/ {
        proxy_pass http://10.0.0.97:8080/api/v1/employee/;
    }

    location /api/v1/attendance/ {
        proxy_pass http://10.0.0.97:8081/api/v1/attendance/;
    }

    location /api/v1/salary/ {
        proxy_pass http://10.0.0.97:8082/api/v1/salary/;
    }
}
```

---

### Restart NGINX

```bash
sudo nginx -t
sudo systemctl restart nginx
```

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
