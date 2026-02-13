# TW best practices (Cloud&DevOps)

**Senior Technical Writers em Cloud/DevOps escrevem em camadas:**
- Modelo arquitetural
- Capacidade do sistema
- Decisão técnica
- Ferramenta usada

Ferramenta é sempre o último nível, não o primeiro.

🧠 O que isso significa na prática

1️⃣ Sempre começar pelo modelo
- O sistema é stateless ou stateful?
- É síncrono, assíncrono ou híbrido?
- Onde está o source of truth?
- Onde há acoplamento?
- Onde há risco?
- Isso define o raciocínio estrutural do documento.

2️⃣ Documentar propriedades, não infraestrutura
- Infraestrutura muda.
- Propriedades arquiteturais não deveriam mudar facilmente.
Exemplo:
- ❌ "Runs on Kubernetes"
- ✅ "Application services are stateless and horizontally scalable."

3️⃣ Tornar explícitos os invariantes
- Invariantes são aquilo que não pode falhar:
- O banco é a fonte de verdade?
- Webhooks só disparam após commit?
- Logs são obrigatórios para toda requisição?
- Deploy é sempre via pipeline automatizado?
- Se isso não estiver claro, o documento está superficial.

4️⃣ Explicitar decisões e trade-offs

Cloud/DevOps envolve escolhas:

- Por que multi-tenant?
- Por que JWT?
- Por que orquestração via Kubernetes?
- Por que webhook baseado em evento persistido?
- Documentação madura registra intenção arquitetural.

5️⃣ Ferramentas vêm por último
- Ferramenta é detalhe de implementação.
- O documento deve sobreviver à troca de ferramenta.
- Se Kubernetes virar ECS, a arquitetura não deve precisar ser reescrita do zero.

🔥 Resumo Final

**Senior TW em Cloud/DevOps escreve para responder:**

- Como o sistema foi pensado?
- Onde ele é consistente?
- Onde ele escala?
- Onde ele pode falhar?
- Como ele se mantém observável?
- Quais decisões moldaram essa arquitetura?

Ele transforma infraestrutura em raciocínio estruturado.
