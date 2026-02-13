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

flowchart LR

    Client[Client Application]

    subgraph AWS
        ALB[AWS Application Load Balancer]

        subgraph EKS Cluster
            Ingress[Kubernetes Ingress]
            API[API Service (Stateless)]
        end

        RDS[(PostgreSQL - RDS)]
        CW[CloudWatch Logs]
        Prom[Prometheus Metrics]
    end

    Processor[External Payment Processor]
    Webhook[Client Webhook Endpoint]

    Client -->|HTTPS Request| ALB
    ALB --> Ingress
    Ingress --> API

    API -->|Create / Update Transaction| RDS
    API -->|Payment Execution| Processor
    Processor -->|Response| API

    API -->|Emit Event| Webhook

    API -->|Structured Logs| CW
    API -->|Metrics| Prom


## 3. Core Components

### 3.1 API Service

#### Description
The API Service is the single, stateless application entry point for NimbusPay’s runtime behavior. It exposes REST endpoints and orchestrates the full synchronous transaction lifecycle. Statelessness is enforced at the service layer; all durable state is persisted in the database.

#### Responsibilities
- Accept and process payment-related API requests (e.g., create payment, query status).
- Authenticate requests by validating JWTs issued via the OAuth2 flow.
- Validate request payloads and enforce API-level invariants.
- Create and update transaction records in the persistence layer.
- Invoke the external payment processor and normalize its response into platform transaction states.
- Emit internal transaction events after committed state transitions.
- Initiate webhook delivery when client notifications are configured.
- Produce structured logs and metrics for monitoring and troubleshooting.

#### Key Characteristics
- Stateless runtime; horizontal scaling is handled at the orchestration layer.
- Orchestrates the full transaction lifecycle within a single synchronous execution path.
- Runs as containerized workloads orchestrated by Kubernetes (EKS) behind ingress routing.
- Central convergence point for security enforcement and request-level auditing.


---

### 3.2 Persistence Layer

#### Description
The Persistence Layer is the platform’s durable system of record for transaction state and related metadata. It is implemented as a managed PostgreSQL instance hosted on AWS RDS.

#### Responsibilities
- Persist canonical transaction lifecycle state (created, updated, final outcome).
- Support read/write patterns for API operations and operational support use cases.
- Guarantee consistency for state transitions that drive downstream event and webhook behavior.

#### Key Characteristics
- PostgreSQL (RDS) acts as the authoritative store; the database state is the single source of truth.
- Designed for durability and consistency rather than in-memory optimization (no cache layer present).
- All downstream behaviors (events, webhooks, client queries) are derived strictly from committed database state.


---

### 3.3 External Payment Processor

#### Description
The External Payment Processor is a downstream system invoked by NimbusPay to execute payment actions. It represents an architectural boundary outside NimbusPay’s operational control and must be treated as a reliability and latency risk surface.

#### Responsibilities
- Receive payment execution requests from NimbusPay.
- Return execution outcomes that can be mapped to internal transaction statuses.

#### Key Characteristics
- Operates outside NimbusPay’s control; availability and response time directly affect request latency.
- Integrated synchronously within the API request flow (no separate worker or broker).
- Requires defensive handling (timeouts, retries, error normalization) to protect platform stability.


---

### 3.4 Webhook Delivery Mechanism

#### Description
The Webhook Delivery Mechanism provides client-facing event notifications derived from committed transaction state. After a transaction state change is persisted, NimbusPay triggers an outbound HTTPS request to the client’s configured webhook endpoint.

#### Responsibilities
- Translate internal transaction events into standardized webhook payloads.
- Deliver outbound notifications when client endpoints are configured.
- Ensure notifications reflect committed state transitions rather than in-flight operations.

#### Key Characteristics
- Event-driven within the boundaries of the API service (no dedicated broker or worker component).
- Delivery occurs only after persistence is updated, reducing risk of notifying on uncommitted state.
- Reliability characteristics are constrained by synchronous execution; no asynchronous queue currently exists.


---

### 3.5 Observability Layer

#### Description
The Observability Layer provides operational visibility required to run and support NimbusPay in production environments. Logging is centralized in CloudWatch, while metrics are collected via Prometheus and used to detect regressions and trigger alerts.

#### Responsibilities
- Centralize application and platform logs for investigation, auditing, and support.
- Collect service and infrastructure metrics (latency, error rates, throughput, resource usage).
- Trigger alerts through CloudWatch Alarms based on defined thresholds and failure conditions.

#### Key Characteristics
- Operational visibility is externalized; no critical system state depends solely on in-memory runtime inspection.
- Logs are centralized for cross-environment troubleshooting.
- Metrics provide time-series visibility into performance and reliability characteristics.
- Alerts surface operational degradation without requiring manual log inspection.


## 4. Infrastructure & Hosting

### 4.1 Cloud Provider & Region Model

NimbusPay is hosted on AWS and currently operates under a single-region deployment model. The architecture does not declare multi-region replication or cross-region failover. As a result, a regional outage would constitute a platform-wide availability event unless additional regional strategies are introduced beyond the current design.

Availability is therefore bounded by the operational health of the single AWS region in use.

---

### 4.2 Compute & Orchestration

Application workloads run as Docker containers orchestrated by Kubernetes on Amazon EKS. The runtime model assumes stateless application instances, with all durable state persisted externally in the database.

A single EKS cluster serves all environments, with logical isolation enforced through Kubernetes namespaces. Provisioning and lifecycle management are driven via the CI/CD pipeline (GitHub Actions), which deploys application workloads into environment-specific namespaces.

This model centralizes execution within a shared cluster substrate while relying on logical isolation for separation of concerns.

---

### 4.3 Networking Model

Inbound traffic enters through an AWS Application Load Balancer (ALB) over HTTPS and is forwarded into the Kubernetes cluster via the ingress controller.

This establishes a perimeter boundary at the load balancer, separating internet-facing ingress traffic from internal east-west communication within the cluster. Once inside the cluster, requests are routed to application services through Kubernetes networking.

Failure domains at this layer include:
- ALB availability or misconfiguration
- Ingress routing rules
- Cluster-level networking policies
- Internal service discovery failures

---

### 4.4 Data Infrastructure

Persistent transaction data is hosted on AWS RDS using PostgreSQL. The database serves as the platform’s single source of truth and is accessed by workloads running in EKS.

Because the database is the authoritative state store, database availability directly determines platform consistency guarantees. There is no cache layer present, meaning database performance and connectivity directly affect API responsiveness.

Potential failure modes include:
- RDS instance degradation or outage
- Connection pool exhaustion
- Schema migration issues
- Network connectivity failures between EKS and RDS

---

### 4.5 Environment Isolation

NimbusPay supports dev, staging, and production environments. These environments are logically isolated within the same EKS cluster using Kubernetes namespaces rather than separate clusters.

Namespace isolation separates workloads and associated Kubernetes resources; however, environments share underlying cluster capacity and cluster-level components (e.g., nodes, control plane, ingress).

As a result, cluster-level misconfiguration or resource exhaustion can impact multiple environments simultaneously.

---

### 4.6 Scaling Model

Scaling is achieved primarily through horizontal scaling of container replicas within EKS. Since application services are stateless, replicas can be increased or decreased to meet demand.

However, the absence of asynchronous buffering mechanisms (e.g., message queues or worker tiers) means backpressure propagates directly through the synchronous request path.

Throughput is therefore bounded by:
- API service replica capacity
- RDS performance limits
- External payment processor latency
- Cluster resource constraints (CPU, memory)

Observability is provided through centralized logs in CloudWatch and metrics collected via Prometheus, enabling capacity analysis and identification of scaling bottlenecks.

---

### 4.7 Failure Domains

Given the current deployment model, NimbusPay’s primary availability risks concentrate around a small set of shared dependencies:

- The single AWS region
- The shared EKS cluster and ingress path
- The RDS PostgreSQL instance
- The external payment processor

Because execution is synchronous and no queue-based buffering is present, prolonged downstream failures or high latency in external dependencies can directly increase API latency and error rates.

Availability is therefore constrained by the narrowest dependency in the synchronous execution chain.


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

