# Architecture Overview

## 1. System Overview

NimbusPay is a B2B SaaS payment platform that exposes digital payment capabilities through a RESTful API. The system is designed as a multi-tenant architecture, with containerized backend services running on Kubernetes. It uses PostgreSQL as its primary data store and is hosted on AWS. Authentication is implemented using JWT tokens within an OAuth2 authorization flow.

The platform enables companies to integrate card payments and bank transfers into their applications without building or maintaining their own payment infrastructure. NimbusPay centralizes transaction orchestration, event notifications via webhooks, and secure API access, abstracting infrastructure management, scaling, and security concerns from client systems.

Primary users include backend engineers integrating the API into external systems, DevOps engineers responsible for deployment and infrastructure operations, and the internal support team that monitors transactions, logs, and system health across development, staging, and production environments.

## 2. High-Level Architecture

NimbusPay follows a stateless API-driven architecture with a single authoritative persistence layer. Synchronous payment requests are processed by the API service, while state transitions trigger asynchronous event propagation and optional webhook delivery. The database is the single source of truth for transaction state, and external payment processors are treated as downstream dependencies.

### Entry Layer
Client requests arrive via HTTPS at an AWS Application Load Balancer (ALB), which forwards traffic to the Kubernetes ingress controller. The ingress layer routes requests to the API service running inside the cluster.

### Application Layer
The API service acts as the synchronous transaction orchestrator. It validates JWT tokens and request payloads, creates the initial transaction record, invokes the external payment processor, and updates transaction state accordingly.

### Persistence Layer
All transaction state is stored in PostgreSQL (Amazon RDS). The database is the authoritative store for lifecycle state and reconciliation operations.

### Integration Layer
The external payment processor executes the payment action. Processor responses are normalized and persisted before any client-facing notification occurs.

### Event & Notification Layer
State changes emit internal events. If configured, webhook notifications are dispatched based on committed transaction state, ensuring consistency between stored data and client notifications.

### Observability Layer
Structured logs are exported to CloudWatch. Application and infrastructure metrics are collected via Prometheus. Alerting policies monitor latency, error rates, and dependency failures.


## 3. Core Components
### 3.1 API Layer
### 3.2 Application Services
### 3.3 Database
### 3.4 External Integrations

## 4. Infrastructure & Hosting
- Cloud provider
- Regions
- Networking model
- Containerization / Orchestration

## 5. Environments
- Development
- Staging
- Production
- Source of truth

## 6. Authentication & Authorization
- Auth method
- Token flow
- Role model

## 7. Observability
- Logging
- Metrics
- Monitoring tools
- Alerting

## 8. Scalability & Reliability
- Horizontal/vertical scaling
- HA strategy
- Backup strategy

## 9. Development & Repository
- Where code lives
- Branching model reference
- Local setup link

## 10. Key Architectural Decisions
- Tradeoffs
- Constraints
- Known limitations

