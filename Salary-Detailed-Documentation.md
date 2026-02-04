# Salary API – Application Documentation

---

## Author & Version Information

### Document Details

---

| Author          | Created    | Version | Last updated by | Last Edited On | L0 Reviewer  | L1 Reviewer | L2 Reviewer |
| --------------- | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- |
| Shreyas Awasthi | 2026-02-04 | 1.0     | Shreyas Awasthi | 2026-02-03     | Divya Mishra |  Pritam           |             |

---

## Purpose  
The purpose of the Salary API is to manage salary-related data using a REST-based service. This application was needed to store, retrieve, and manage salary records in a structured way using Cassandra as the database and Redis for caching. It helps standardize salary data handling and provides monitoring and health visibility.

---

## Pre-requisites  
Before deploying the Salary API, ensure the required hardware, software, and security setup is available.

---

## System Requirements  

### Hardware Specifications

| Component | Minimum Recommendation |
|---------|------------------------|
| Processor | Dual-core |
| RAM | 4 GB |
| Disk | 20 GB |
| OS | Ubuntu 22.04 |

---

## Dependencies  

### Build Time Dependencies

| Name | Version | Description |
|----|--------|-------------|
| Maven | Default | Used to build the Salary API artifact |
| Java | Compatible with Spring Boot | Required to compile the application |

---

### Run Time Dependencies

| Name | Version | Description |
|----|--------|-------------|
| Java | JRE compatible | Required to run the JAR |
| Cassandra (ScyllaDB) | 9042 | Database for salary data |
| Redis | 6379 | Used for caching |

---

### Other Dependencies

| Name | Version | Description |
|----|--------|-------------|
| Docker | Latest | Used for container image build |
| systemd | Default | Used to run Salary API as a service |

---

## Important Ports  

### Inbound Traffic

| Port | Description |
|----|-------------|
| 9042 | Used by Cassandra / ScyllaDB |
| 6379 | Used by Redis |
| 8082 | Salary API application port |

---

### Outbound Traffic

| Port | Description |
|----|-------------|
| 8082 | API access and health checks |

---

## Others  

| Others | Description |
|------|-------------|
| Logging | Enabled using Spring logging |
| Swagger | Enabled for API documentation |

---

## Architecture  
The Salary API follows a Spring Boot–based architecture where the application interacts with Cassandra for data persistence and Redis for caching. Monitoring is enabled using Spring Boot Actuator and Prometheus endpoints.

---

## Dataflow Diagram  
Client sends API requests to the Salary API → Salary API processes requests → Data is stored or fetched from Cassandra → Redis is used for caching → Response is returned to the client.  
Health and metrics data are exposed via Actuator endpoints.

---

## Step-by-step Installation of Salary API  

### Step 1: Installation of Software Dependencies  

#### Build Dependency
```bash
sudo apt install maven
```
- Run Time Dependency

       sudo apt install openjdk-17-jre

#### Other Dependency

Cassandra (ScyllaDB) and Redis must be running and reachable at the following endpoints:

- Cassandra (ScyllaDB): 10.0.1.25:9042

- Redis: 10.0.1.25:6379
---

### Step 2: Build / Artifact Generation

#### Clone the repository:

    git clone <salary-api-repository-url>
    cd salary-api

#### Build the application:

    mvn clean install -DskipTests
---

### Step 3: Application Deployment

#### Run the application:

    java -jar target/salary-0.1.0-RELEASE.jar

#### Verify application status:

    ss -tulpn | grep 8082
---

## Monitoring

Monitoring is enabled using Spring Boot Actuator. It provides visibility into application health, metrics, and performance to ensure reliability and stability.

---
## Monitoring Metrics & Thresholds

The following table defines key monitoring parameters, their business priority, and alert thresholds to ensure application stability, performance, and security.

| Parameter          | Description            | Priority | Threshold        |
|--------------------|------------------------|----------|------------------|
| Disk Utilization   | Disk space usage       | High     | > 90%            |
| Availability       | Application uptime     | High     | ≥ 99.9%          |
| Memory Utilization | Memory consumption     | Medium   | > 80%            |
| CPU Utilization    | CPU usage              | Medium   | > 70%            |
| Network Traffic    | Network bandwidth use  | Medium   | Varies           |
| Latency            | API response time      | High     | < 300 ms         |
| Errors             | Application error rate | High     | > 5 errors/min   |
| Throughput         | Requests processed     | High     | > 1000 req/min   |
| Security           | Auth & access controls | High     | Continuous check |
---

## Health Check
| Name       | Type           | InitialDelaySeconds | PeriodSeconds | TimeoutSeconds | SuccessThreshold | FailureThreshold |
| ---------- | -------------- | ------------------- | ------------- | -------------- | ---------------- | ---------------- |
| Salary API | ReadinessProbe | 10                  | 10            | 5              | 1                | 3                |
| Salary API | LivenessProbe  | 10                  | 10            | -              | 5                | 1                |


#### Health check command:

    curl http://10.0.2.75:8082/actuator/health

---
## Logging

| Log Type    | Location               | Description        |
| ----------- | ---------------------- | ------------------ |
| Event Logs  | location/to/event.log  | Application events |
| Auth Logs   | location/to/access.log | Authorization logs |
| Server Logs | location/to/server.log | Server activity    |
| Threat Logs | location/to/threat.log | Security events    |

---
## Disaster Recovery

In case of failure, redeploy the application and reconnect it to Cassandra and Redis. Data recovery is handled at the database level.

---

## High Availability

High availability can be achieved by running multiple instances of the Salary API behind a load balancer.

## Troubleshooting

| Issue               | Possible Cause  | Resolution       |
| ------------------- | --------------- | ---------------- |
| App not starting    | Port conflict   | Free port 8082   |
| DB connection error | Cassandra down  | Verify DB status |
| Health check fails  | Service stopped | Restart service  |

---

FAQs

Is this application free?
Yes, it is open-source.

Can it be deployed on any cloud?
Yes, on any cloud platform.

Is there an enterprise version?
No enterprise version available.

---
## Contact Information 
| Name            | Team    | Contact Type | Details |
| --------------- | ------- | ------------ | ------- |
| Shreyas Awasthi | Saarthi | Email        | shreyas.awasthi.snaatak@mygurukulam.CO |

--- 
## References

| Descriptions | Links |
|-------------|------|
| Spring Boot documentation | [Spring Boot Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/) |
| Git documentation | [Git Docs](https://git-scm.com/docs) |
| Documentation format reference | [AWS Documentation](https://docs.aws.amazon.com/) |



