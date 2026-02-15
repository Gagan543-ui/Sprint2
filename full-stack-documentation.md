# OT Microservices Architecture

##  Overview

This project demonstrates a complete end-to-end deployment of a secure microservices-based architecture on AWS using:

- Employee API (Go + ScyllaDB)
- Attendance API (Python + PostgreSQL + Redis)
- Salary API (Spring Boot + ScyllaDB)
- React Frontend (NGINX Reverse Proxy)
- Bastion Host for secure SSH access

Architecture follows a **multi-tier VPC design**:


---

#  Infrastructure Architecture

## 1 VPC Configuration

| Resource | Name | CIDR |
|----------|------|------|
| VPC | ot_micro_vpc | 10.0.0.0/24 |

---

## 2 Subnet Design

| Subnet | AZ | CIDR | Type |
|--------|----|-------|------|
| ot_micro_subnet_entry | ap-south-1a | 10.0.0.0/26 | Public |
| ot_micro_subnet_api | ap-south-1b | 10.0.0.64/26 | Private |
| ot_micro_subnet_db | ap-south-1c | 10.0.0.128/26 | Private |
| ot_micro_subnet_frontend | ap-south-1b | 10.0.0.192/26 | Public |

---

## 3 Gateway Configuration

### Internet Gateway
- Name: `ot_micro_igw`
- Attached to: `ot_micro_vpc`

### NAT Gateway
- Name: `ot_micro_nat`
- Attached to: `ot_micro_subnet_entry`
- Purpose: Enables outbound internet for private subnets

---

## 5 Route Tables

### Public Route Table
Associated with:
- ot_micro_subnet_entry
- ot_micro_subnet_frontend

Route: 0.0.0.0/0 → Internet Gateway


### Private Route Table
Associated with:
- ot_micro_subnet_api
- ot_micro_subnet_db

Route: 0.0.0.0/0 → NAT Gateway


---

#  EC2 Instances

## Bastion Host

| Parameter | Value |
|------------|--------|
| AMI | Ubuntu 22.04 |
| Type | t3.micro |
| Subnet | Public |
| Purpose | SSH Access |

---

## Backend Server

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

## Database Server

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

#  SSH Access

### Local → Bastion

```bash
chmod 400 ot_micro_service.pem
ssh -i ot_micro_service.pem ubuntu@<Bastion_Public_IP>
```
#### Bastion → Private Instances
```bash
ssh -i ot_micro_service.pem ubuntu@<Private_IP>
```
---

## Database Setup

### PostgreSQL

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

### Redis
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

## Backend Services

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

## Frontend Deployment

### Technology Stack
- React
- NodeJS
- NGINX

---

### Build Process

```bash
npm install
npm run build
```

---

### Deployment Steps

```bash
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
```

---

## NGINX Reverse Proxy Configuration

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


## 9. Health Checks

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

## 10. Security Measures

| Control Area                 | Implementation Detail                          | Objective                              |
|------------------------------|-----------------------------------------------|----------------------------------------|
| Network Isolation            | Private subnet architecture                   | Restrict direct public access         |
| Outbound Traffic Control     | NAT Gateway                                   | Secure internet access from private subnet |
| SSH Access Control           | Bastion host only                             | Centralized administrative access     |
| API Access Management        | NGINX reverse proxy                           | Eliminates CORS and centralizes routing |
| Database Security            | Authentication enabled                        | Enforce credential-based access       |
| Database Exposure            | No public DB ports                            | Prevent external database access      |

## Architecture Flow
