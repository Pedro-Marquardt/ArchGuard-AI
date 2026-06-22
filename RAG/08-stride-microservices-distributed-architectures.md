# STRIDE Aplicado a Microsserviços: Framework para Análise de Arquiteturas Distribuídas

> **Fonte:** Síntese de literatura acadêmica: Kanani & Dhenia (WJARR, 2022); IJNRD (2023); Nanjing University ACM (2022)  
> **Referências:**  
> - "Threat Modeling for APIs in microservices architectures" — WJARR-2022-0458  
> - "Advanced Threat Modeling Techniques for Microservices Architectures" — IJNRD2304737  
> - "An Automated Approach to Constructing STRIDE Threat Rule Model" — ACM DL 10.1145/3772283  
> **Tipo:** Revisão de literatura + síntese aplicada  
> **Relevância para o projeto:** Alta — framework para análise de arquiteturas de microsserviços que o ArchGuard-AI deve analisar

---

## 1. O Desafio do Threat Modeling em Microsserviços

Arquiteturas de microsserviços apresentam desafios únicos para o threat modeling que não existiam em sistemas monolíticos:

### 1.1 Complexidade Exponencial

```
Monolito: 1 aplicação × 6 categorias STRIDE = 6 análises

Microsserviços: N serviços × M interações × 6 categorias = complexidade quadrática
Exemplo: 20 serviços × 190 interações × 6 = 22.800 combinações potenciais
```

### 1.2 Características que Complicam a Análise

| Característica | Impacto no Threat Modeling |
|---|---|
| **Comunicação via rede** | Todo serviço-a-serviço precisa ser analisado como potencial vetor de ataque |
| **Múltiplos limites de confiança** | Cada chamada de API cruza um limite de confiança implícito |
| **Natureza efêmera** | Containers sobem e descem, IPs mudam — threat model precisa ser dinâmico |
| **Dados distribuídos** | Dados de um domínio estão distribuídos em múltiplos serviços e bases de dados |
| **Falhas parciais** | Um serviço comprometido pode afetar outros de forma não-óbvia |
| **Diversidade tecnológica** | Cada serviço pode usar linguagem e framework diferentes, com vetores distintos |

---

## 2. Adaptação do STRIDE para Microsserviços

### 2.1 Análise em Múltiplos Níveis

A análise STRIDE em microsserviços deve ser aplicada em pelo menos três níveis:

**Nível 1: Sistema (visão macro)**
```
[Cliente] ──▶ [API Gateway] ──▶ [Cluster de Microsserviços] ──▶ [Databases]
```
Análise: Ameaças ao sistema como um todo, boundaries externas

**Nível 2: Serviço Individual**
```
[Order Service] ──▶ [Payment Service] ──▶ [Notification Service]
```
Análise: Cada interação entre serviços como potencial vetor de ataque

**Nível 3: Componente Interno**
```
[API Handler] ──▶ [Business Logic] ──▶ [Repository] ──▶ [Database]
```
Análise: Fluxo interno do serviço, validação de entrada, acesso a dados

---

### 2.2 Spoofing em Microsserviços

**Vetores específicos:**

| Vetor | Descrição | Severidade |
|---|---|---|
| **Service impersonation** | Serviço malicioso se passa por serviço legítimo | Crítica |
| **JWT manipulation** | Forjamento ou replay de tokens entre serviços | Alta |
| **DNS poisoning** | Roteamento de chamadas para serviço falso | Alta |
| **Sidecar bypass** | Contornar proxy de service mesh para evitar autenticação | Alta |

**Padrões de mitigação:**

```yaml
# Exemplo: mTLS entre serviços via Istio
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # Nenhuma comunicação sem mTLS
```

**Controles:**
- mTLS (mutual TLS) obrigatório entre todos os serviços
- Service mesh (Istio, Linkerd) para gerenciamento centralizado de identidade de serviço
- SPIFFE/SPIRE para identidade criptográfica de workloads
- Rotação automática de certificados de serviço (SVID)

---

### 2.3 Tampering em Microsserviços

**Vetores específicos:**

| Vetor | Descrição | Severidade |
|---|---|---|
| **Message queue poisoning** | Injeção de mensagens maliciosas em Kafka/RabbitMQ | Alta |
| **Event sourcing tampering** | Adulteração do event log histórico | Crítica |
| **Service mesh config tampering** | Modificação de regras de routing/policies | Alta |
| **Shared database compromise** | Um serviço adulterando dados de domínio de outro | Alta |

**Antipadrões a evitar:**
- ❌ Shared databases entre serviços (violação de bounded context que facilita tampering)
- ❌ Mensagens em queues sem assinatura digital
- ❌ Event stores sem controle de integridade

**Padrões de mitigação:**
- Cada serviço com sua própria base de dados (database-per-service pattern)
- Assinatura digital de eventos publicados em message queues
- Imutabilidade de eventos no event sourcing (append-only)
- Validação de schema em consumers de eventos

---

### 2.4 Repudiation em Microsserviços

**Desafio único:** Em um sistema com 20+ serviços, correlacionar logs distribuídos para rastrear uma transação de ponta a ponta é intrinsecamente difícil.

**Vetores específicos:**

| Vetor | Descrição |
|---|---|
| **Distributed trace gaps** | Transação que atravessa múltiplos serviços sem correlation ID |
| **Inconsistent logging** | Diferentes formatos e níveis de detalhe por serviço |
| **Log aggregation failure** | Perda de logs durante falha de rede para agregador central |
| **Missing saga audit** | Falta de registro de compensating transactions em sagas |

**Padrão de mitigação — Correlation ID obrigatório:**

```
GET /orders/123
X-Correlation-ID: a7f3-9d2b-c1e4  ◀── Gerado no gateway, propagado por TODOS os serviços
X-Request-ID: req-456
X-B3-TraceId: abc123  ◀── OpenTelemetry trace ID

[API Gateway] ──correlation-id──▶ [Order Service] ──correlation-id──▶ [Payment Service]
     │                                    │                                    │
  [Logs]                               [Logs]                              [Logs]
     └──────────────────────────────────────────────────────────────────────┘
                                [Agregador Central: OpenSearch/Splunk]
                                [Correlacionado por correlation-id]
```

**Controles:**
- Correlation ID obrigatório gerado no API Gateway e propagado por toda a cadeia
- OpenTelemetry para distributed tracing padronizado
- Log aggregation centralizada (ELK Stack, OpenSearch, Splunk)
- Retenção de traces de transações críticas

---

### 2.5 Information Disclosure em Microsserviços

**Vetores específicos:**

| Vetor | Descrição | Exemplo |
|---|---|---|
| **API contract exposure** | Swagger/OpenAPI público expõe estrutura interna | Endpoints internos visíveis externamente |
| **Service-to-service data leak** | Um serviço expõe dados de outro sem necessidade | Order service retorna dados completos de Customer |
| **Sidecar/proxy log exposure** | Proxy de service mesh logando payloads completos | Envoy logando corpos de requisição |
| **Debug endpoints** | Endpoints de saúde/debug com informação excessiva | `/actuator/env` expondo variáveis de ambiente |

**Princípio de Need-to-Know em APIs:**
```
❌ Ruim: Order service retorna objeto Customer completo
{
  "customerId": "123",
  "name": "João Silva",
  "cpf": "123.456.789-00",    ← Order não precisa disso
  "creditCardNumber": "****",  ← Order NUNCA deveria ter isso
  "email": "joao@email.com",  ← Talvez necessário, talvez não
  "address": {...}
}

✅ Bom: Order service recebe apenas o que precisa
{
  "customerId": "123",
  "customerName": "João Silva"  ← Apenas o necessário para exibição
}
```

---

### 2.6 Denial of Service em Microsserviços

**Vetores específicos e padrões de resiliência:**

| Vetor | Padrão de Mitigação |
|---|---|
| **Cascading failure** | Circuit Breaker (Resilience4j, Hystrix) |
| **Resource starvation** | Bulkhead — isolamento de thread pools por dependência |
| **Slow consumer** | Timeout configurado + retry com exponential backoff |
| **Message queue flood** | Back pressure e consumer groups com rate limiting |
| **Noisy neighbor em K8s** | Resource limits (CPU/memory requests e limits) obrigatórios |

**Exemplo: Circuit Breaker Pattern**
```
[Service A] ──calls──▶ [Service B]
                              │
              [Service B começa a falhar]
                              │
              [Circuit Breaker abre após N falhas]
                              │
              [Service A retorna fallback response]
              [Service B tem tempo para se recuperar]
              [Circuit Breaker fecha após timeout]
```

---

### 2.7 Elevation of Privilege em Microsserviços

**Vetores específicos:**

| Vetor | Descrição | Severidade |
|---|---|---|
| **Lateral movement** | Serviço comprometido acessa outros serviços além do escopo | Crítica |
| **Namespace escape K8s** | Pod acessa recursos fora de seu namespace | Alta |
| **Over-permissive RBAC** | Service account com `cluster-admin` desnecessário | Alta |
| **Inter-service trust abuse** | Serviço interno com acesso excessivo por ser "interno" | Alta |

**Zero Trust para Microsserviços:**
```
Premissa incorreta: "serviços internos confiam uns nos outros"
Premissa correta: "cada serviço verifica a identidade de cada chamada"

Implementação:
1. Autenticação: mTLS + SPIFFE identity verification
2. Autorização: Authorization policies no service mesh
3. Auditoria: Access logs para todas as chamadas inter-serviço
```

---

## 3. Automação do Threat Modeling em Microsserviços

### 3.1 Abordagem da Nanjing University (ACM 2022)

O paper da Nanjing University propõe uma abordagem de **construção automática de regras STRIDE**:

**Método:**
1. Coleta de vulnerabilidades do CNNVD (China National Vulnerability Database)
2. Treinamento de modelo de Machine Learning para classificação STRIDE
3. Geração automática de regras de ameaças
4. Atualização contínua da base de regras

**Resultados:**
- Aumento significativo de precisão vs. métodos manuais
- Redução da dependência de especialistas para atualização de regras
- Escalabilidade para ambientes com evolução rápida

### 3.2 Threat Modeling as Code (TM-as-Code)

```python
# Exemplo com OWASP pytm para microsserviços
from pytm import TM, Server, Process, Dataflow, Boundary, Actor

tm = TM("E-commerce Microservices")
tm.description = "Análise de ameaças para plataforma de e-commerce em microsserviços"

# Boundaries
internet = Boundary("Internet")
internal_network = Boundary("Internal Network")
data_store_boundary = Boundary("Data Store Boundary")

# Atores externos
customer = Actor("Customer")
customer.inBoundary = internet

# API Gateway
api_gateway = Process("API Gateway")
api_gateway.inBoundary = internal_network

# Microsserviços
order_service = Process("Order Service")
payment_service = Process("Payment Service")
inventory_service = Process("Inventory Service")

# Dados
orders_db = Server("Orders DB")
payment_db = Server("Payment DB")
orders_db.inBoundary = data_store_boundary

# Fluxos de dados
customer_to_gateway = Dataflow(customer, api_gateway, "HTTPS Request")
gateway_to_order = Dataflow(api_gateway, order_service, "Internal gRPC")
order_to_payment = Dataflow(order_service, payment_service, "Payment Request")

# Geração automática do threat model
tm.process()
```

---

## 4. DevSecOps Integration: STRIDE Contínuo

### 4.1 Integração em Pipeline CI/CD

```
Código Push
    │
    ▼
[Static Analysis]
├── SAST (código-fonte)
├── Dependency scanning
└── IaC scanning (Terraform/K8s manifests)
    │
    ▼
[Threat Model Update]
├── Detecção de novos endpoints/serviços
├── Detecção de novas dependências
├── Re-análise STRIDE automática para mudanças
└── Comparação com baseline aprovado
    │
    ▼
[Security Gate]
├── Aprovação automática: sem novas ameaças críticas
├── Review obrigatório: novas ameaças de média/alta severidade
└── Bloqueio: novas ameaças críticas não mitigadas
```

### 4.2 Monitoramento Contínuo (Runtime Threat Detection)

```
Produção ──eventos──▶ [SIEM/SOAR]
                            │
                    [Mapeamento para STRIDE]
                            │
                 [Alerta se padrão = ameaça conhecida]
                            │
                    [Resposta automática]
                    ├── Isolar serviço comprometido
                    ├── Bloquear IP de origem
                    └── Notificar equipe de segurança
```

---

## 5. Checklist STRIDE para Review de Arquitetura de Microsserviços

### ✅ Spoofing
- [ ] mTLS habilitado entre todos os serviços
- [ ] Service identity com SPIFFE/SPIRE
- [ ] JWT validado em cada serviço (não apenas no gateway)
- [ ] Rotação automática de credenciais de serviço

### ✅ Tampering
- [ ] Database-per-service pattern implementado
- [ ] Mensagens em queues assinadas digitalmente
- [ ] Validação de schema em todos os consumers de eventos
- [ ] Event stores imutáveis (append-only)

### ✅ Repudiation
- [ ] Correlation ID propagado por toda a cadeia de chamadas
- [ ] OpenTelemetry configurado para distributed tracing
- [ ] Logs centralizados e imutáveis
- [ ] Retenção de traces de transações críticas ≥ 90 dias

### ✅ Information Disclosure
- [ ] APIs seguem need-to-know (sem dados além do necessário)
- [ ] Debug endpoints desabilitados em produção
- [ ] Service mesh não loga payloads completos
- [ ] Criptografia de dados sensíveis em trânsito e repouso

### ✅ Denial of Service
- [ ] Circuit breakers configurados para todas as dependências externas
- [ ] Resource limits (CPU/memory) definidos para todos os pods K8s
- [ ] Rate limiting no API Gateway
- [ ] Timeouts configurados em todas as chamadas HTTP/gRPC

### ✅ Elevation of Privilege
- [ ] Network Policies K8s bloqueando comunicação não necessária
- [ ] RBAC granular para service accounts
- [ ] Pods sem `privileged: true` ou montagem de host volumes
- [ ] Zero Trust: verificação de identidade em cada chamada inter-serviço

---

## Referências

- Kanani & Dhenia. "Threat Modeling for APIs in microservices architectures: A practical framework." WJARR, 2022. https://wjarr.com/sites/default/files/fulltext_pdf/WJARR-2022-0458.pdf
- "Advanced Threat Modeling Techniques for Microservices Architectures." IJNRD, 2023. https://www.ijnrd.org/papers/IJNRD2304737.pdf
- "An Automated Approach to Constructing STRIDE Threat Rule Model and Updating Rule Base." ACM DL, 2022. https://dl.acm.org/doi/epdf/10.1145/3772283
- OWASP Microservices Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Microservices_Security_Cheat_Sheet.html
- NIST SP 800-204: Security Strategies for Microservices-based Application Systems
