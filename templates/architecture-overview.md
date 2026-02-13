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

O diagrama:
- Mostra fronteiras (AWS vs External)
- Separa cluster do restante da infra
- Mostra dependências externas
- Mostra fluxo principal
- Mostra observability
- Não entra em detalhe irrelevante (pods, subnets, etc.)

## 3. Core Components

### 3.1 API Service
- Description
- Responsibilities
- Key Characteristics

### 3.2 Database
- Description
- Responsibilities
- Key Characteristics

### 3.3 External Payment Processor
- Description
- Responsibilities
- Key Characteristics

### 3.4 Webhook Delivery
- Description
- Responsibilities
- Key Characteristics

### 3.5 Observability Stack
- Description
- Responsibilities
- Key Characteristics

*insumos*

1️⃣ Code (Primary Source)
Inspect the repository to identify real service boundaries.
Questions to answer:
- What services exist?
- Is there only one API service?
- Is there a background worker?
- Is there a consumer?
- Is there a scheduler?
- Is there an event publisher?
- Is there a separate webhook dispatcher?

Interpretation:
- If there is only one `api` service → it accumulates multiple responsibilities.
- If there is `api + worker` → they are distinct architectural components.

Principle:  
Code reveals structural truth.

2️⃣ Infrastructure (Secondary Source)
Inspect infrastructure configuration to understand operational separation.
Look at:
- Kubernetes manifests
- Helm charts
- Terraform modules
- AWS console configuration

Questions to answer:
- How many deployments exist?
- How many services exist?
- Is autoscaling configured?
- Is there a queue?
- Is there a message broker?
- Are workloads separated by namespace or cluster?

Principle: 
Infrastructure reveals operational boundaries.

3️⃣ Engineering (Conceptual Source)
Validate component responsibility through architectural reasoning.
Ask:
"If I remove this component, what stops working?"
This clarifies real system dependency.

Examples:
- If PostgreSQL is removed → no state persists → critical component.
- If webhook delivery is removed → core payment still works → secondary component.

Principle:  
Responsibility defines the component — not the tool used.

## 4. Infrastructure & Hosting

### 4.1 Cloud Provider & Region Model
### 4.2 Compute & Orchestration
### 4.3 Networking Model
### 4.4 Data Infrastructure
### 4.5 Environment Isolation
### 4.6 Scaling Model

*insumos*

1️⃣ Infraestrutura como Código (Fonte Primária)
Onde buscar
- Repositório de Terraform
- Repositório de Helm
- Manifests do Kubernetes
- Templates CloudFormation (se aplicável)

O que extrair
- Cloud provider
- Região(ões)
- Tipo de cluster (EKS, GKE, AKS, etc.)
- Node groups
- Configuração do RDS
- Load balancers
- Configuração de autoscaling
- Estratégia de namespaces por ambiente
- Modelo de VPC / subnets (se relevante)

Perguntas que você responde
- Existe um cluster por ambiente ou isolamento via namespaces?
- O banco é compartilhado ou isolado entre ambientes?
- O sistema é multi-region?
- Existe autoscaling configurado?
- Alta disponibilidade (HA) está explicitamente configurada?

Princípio: Infraestrutura como Código é a fonte de verdade mais confiável.

2️⃣ Console Cloud (Fonte de Validação)
Infraestrutura como Código pode estar desatualizada. Sempre valide.
Verifique em
- AWS Console
- EKS
- RDS
- ALB
- CloudWatch

Confirme
- Região ativa
- Quantidade de instâncias
- Configuração real de autoscaling
- Configuração Multi-AZ
- Estado real da infraestrutura em execução

3️⃣ DevOps / Platform Engineer (Fonte Decisória)
Ferramentas mostram configuração. Engenheiros explicam intenção.
Pergunte
- “Como os ambientes são isolados?”
- “O scaling é automático ou manual?”
- “O sistema é single-region?”
- “Qual é o domínio de falha?”
- “Qual é a estratégia de alta disponibilidade?”

Engenheiros fornecem decisões arquiteturais. Você documenta a intenção arquitetural.

4️⃣ Pipeline de CI/CD (Fonte Operacional)

A configuração de deploy revela o comportamento dos ambientes.

Inspecione
- GitHub Actions (ou GitLab CI)
- Targets de deploy
- Mapeamento de namespaces
- Variáveis de ambiente
- Fluxos de promoção entre ambientes

Isso revela
- Como os ambientes são promovidos
- Se o isolamento é real ou apenas lógico
- Se existem aprovações manuais
- Como a configuração varia entre ambientes

Modelo Mental

A documentação de Infrastructure & Hosting deve descrever claramente:
- Onde o sistema roda
- Como ele é provisionado
- Como os ambientes são isolados
- Como ele escala
- Onde ele pode falhar

Esta seção descreve limites de execução e realidades operacionais.

Pergunta Crítica

“Se a AWS us-east-1 ficar indisponível, o que acontece?”

Se ninguém souber responder com clareza, você identificou um gap arquitetural.

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

