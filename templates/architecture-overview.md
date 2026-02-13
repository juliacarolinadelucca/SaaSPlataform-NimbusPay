# Architecture Overview

## 1. System Overview
**Brief description of the platform, its purpose and primary users.**
- Definir claramente o que o sistema é (categoria + natureza técnica).
- Explicar o problema que resolve.
- Identificar os primary users (personas técnicas ou de negócio).
- Delimitar o escopo (o que NÃO é responsabilidade da plataforma, se relevante).

*structure*
- 1 parágrafo de definição técnica
- 1 parágrafo de propósito / problema resolvido
- 1 parágrafo curto sobre usuários principais

*insumos*
- Product documentation
- README do repositório
- Pitch comercial / website
- Conversa com engenheiros
- Diagramas existentes
- Código
- Tickets de arquitetura
- RFCs
- Reuniões técnicas

## 2. High-Level Architecture
**Diagram and summary of core components.**

Descrever:
- Componentes principais
- Como eles se comunicam
- Fluxo principal de requisição
- Dependências externas
- Camadas do sistema

*insumos*

1️⃣ Código
- Estrutura de pastas
- Serviços existentes
- Dependências

2️⃣ Infraestrutura
- Kubernetes manifests
- Terraform
- AWS console
- Load balancer

3️⃣ Pipeline
- Onde deploya
- Como sobe container
- Como escala

4️⃣ Conversa com engenheiro
Pergunta-chave:
“What happens when a payment request hits the system?”

**Um diagrama High-Level profissional deve conter:**

- Client
- Load Balancer
- Kubernetes Cluster
  - API Service
- Database (RDS)
- External Payment Processor
- Event mechanism
- Webhook outbound
- Observability stack

Não precisa mostrar pods.
Não precisa mostrar node groups.
Não precisa ser infra low-level.

É diagrama de entendimento, não de Terraform.

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

