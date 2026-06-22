# STRIDE para Segurança em Cloud: Guia Alinhado às Diretrizes CISA/NSA

> **Fonte:** CISA (Cybersecurity and Infrastructure Security Agency) — Cloud Security Best Practices + literatura de referência  
> **Publicações base:** CISA/NSA Cloud Security Information Sheets (Março 2024); CSOH Cloud Threat Modeling Guide  
> **URL CISA:** https://www.cisa.gov/news-events/alerts/2024/03/07/cisa-and-nsa-release-cybersecurity-information-sheets-cloud-security-best-practices  
> **Tipo:** Síntese de diretrizes governamentais + melhores práticas setoriais  
> **Relevância para o projeto:** Alta — fornece base para análise de arquiteturas cloud com STRIDE, especialmente relevante para sistemas SaaS/PaaS/IaaS

---

## 1. Contexto: Por Que STRIDE para Cloud?

Ambientes cloud introduzem **superfícies de ataque únicas** que não existem em arquiteturas on-premises tradicionais:

- **Responsabilidade compartilhada**: A fronteira entre o que o provedor protege e o que o cliente deve proteger é frequentemente mal compreendida
- **APIs como superfície primária**: Toda interação com recursos cloud é feita via APIs, criando novos vetores de Spoofing e Tampering
- **Identidade como novo perímetro**: Com o desaparecimento do perímetro de rede tradicional, a identidade torna-se o controle de acesso primário
- **Escala e elasticidade**: A natureza dinâmica do cloud cria desafios para Denial of Service e gerenciamento de acesso
- **Multi-tenancy**: Riscos de Information Disclosure entre tenants de um mesmo provedor

A CISA e a NSA reforçam que o uso do STRIDE nesse contexto ajuda a estruturar a análise de riscos alinhando-a às categorias universalmente reconhecidas pelo setor.

---

## 2. STRIDE Aplicado a Ambientes Cloud

### 2.1 Spoofing em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Exemplo |
|---|---|---|
| **Credential theft** | Roubo de chaves de acesso estáticas | AWS Access Key vazada no GitHub |
| **Token hijacking** | Interceptação de tokens OAuth/JWT temporários | Session token roubado via XSS |
| **Federated identity abuse** | Exploração de trust relationships entre contas | Cross-account role assumption não autorizado |
| **Instance metadata abuse** | Acesso à API de metadados EC2 para obter credenciais | SSRF → http://169.254.169.254/latest/meta-data/ |
| **Service impersonation** | Falsificação de serviços internos em microsserviços | Serviço malicioso respondendo como serviço legítimo |

**Mitigações (CISA-aligned):**
- ✅ Eliminar chaves de acesso estáticas — usar roles IAM e credenciais temporárias
- ✅ MFA obrigatório para todos os usuários, especialmente administradores
- ✅ Implementar IMDSv2 (Instance Metadata Service v2) para bloquear SSRF → metadata
- ✅ Monitoramento de anomalias de autenticação (AWS GuardDuty, Azure Sentinel)
- ✅ Zero Trust Architecture: verificar identidade em cada requisição, nunca confiar implicitamente

---

### 2.2 Tampering em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Exemplo |
|---|---|---|
| **Storage tampering** | Modificação de objetos em storage cloud | S3 object overwrite por IAM excessivamente permissivo |
| **IaC poisoning** | Adulteração de templates Terraform/CloudFormation | Modificação maliciosa antes do apply |
| **Supply chain attack** | Comprometimento de dependências ou imagens de container | Base image maliciosa no Docker Hub |
| **DNS hijacking** | Alteração de registros DNS gerenciados em cloud | Route53 ou Cloud DNS comprometido |
| **Config drift** | Desvio da configuração segura esperada | Security group modificado manualmente em produção |

**Mitigações (CISA-aligned):**
- ✅ Object versioning + Object Lock (WORM) em S3 para dados críticos
- ✅ Drift detection automático (AWS Config, Azure Policy)
- ✅ GitOps para gerenciamento de IaC — toda mudança via pull request com aprovação
- ✅ Assinatura de artefatos e imagens (Cosign, SLSA provenance)
- ✅ Alertas automáticos para mudanças em configurações de segurança críticas

---

### 2.3 Repudiation em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Exemplo |
|---|---|---|
| **Audit trail gaps** | Ausência ou lacunas em logs de atividade | CloudTrail desabilitado em algumas regiões |
| **Log tampering** | Modificação ou deleção de logs de auditoria | Usuário com permissão para deletar CloudTrail logs |
| **Insufficient detail** | Logs sem contexto suficiente para atribuição | Log sem identificação de sessão ou IP de origem |
| **Short retention** | Período de retenção de logs insuficiente | Logs de 7 dias quando investigações levam 30+ dias |

**Mitigações (CISA-aligned):**
- ✅ Ativar CloudTrail/Azure Activity Log em **todas as regiões**, incluindo global
- ✅ Proteger logs contra deleção (S3 Object Lock, CloudTrail Log File Validation)
- ✅ Centralizar logs em conta de segurança dedicada com acesso restrito
- ✅ Retenção mínima de 90 dias online + 1 ano em archive frio
- ✅ Incluir todos os campos necessários para forense: IP, user-agent, session, timestamp

---

### 2.4 Information Disclosure em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Risco |
|---|---|---|
| **Public S3 Buckets** | Buckets configurados como públicos inadvertidamente | Crítico |
| **Overpermissive IAM** | Roles com wildcard (`*`) em resources e actions | Alto |
| **Unencrypted data** | Dados sensíveis armazenados sem criptografia | Alto |
| **Verbose error messages** | Stack traces ou detalhes internos expostos em APIs | Médio |
| **Metadata exposure** | Tags de recursos com informações sensíveis | Médio |
| **Cross-tenant leakage** | Dados de um tenant acessíveis por outro | Crítico |

**Dados do setor:** 
> Pesquisa de 2023 revelou que **40% das organizações** tiveram pelo menos um bucket S3 exposto publicamente de forma não intencional em algum momento. (Source: Accurics/Tenable)

**Mitigações (CISA-aligned):**
- ✅ Habilitar S3 Block Public Access no nível da conta (não apenas dos buckets)
- ✅ Criptografia at-rest com CMK (Customer Managed Keys) para dados sensíveis
- ✅ TLS 1.2+ obrigatório para todos os endpoints (HTTPS only, HSTS)
- ✅ Política de IAM com explicitação de resources e actions (zero wildcards)
- ✅ Macie (AWS) ou Purview (Azure) para descoberta e classificação de dados sensíveis
- ✅ Sanitização de mensagens de erro — nunca expor stack traces em produção

---

### 2.5 Denial of Service em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Exemplo |
|---|---|---|
| **DDoS volumétrico** | Flood de tráfego para esgotar capacidade de rede | Ataque UDP flood contra IP público |
| **Resource exhaustion** | Esgotamento de cotas de serviços cloud | Lambda concurrency limit atingido por ataque |
| **API rate limit bypass** | Bypass de rate limits via múltiplas contas/IPs | Rotação de IPs para evitar rate limiting |
| **Cost amplification** | Ataques que geram custos excessivos | Trigger de processamento custoso repetitivamente |
| **Serverless timeout attacks** | Requisições que forçam timeouts máximos | Funções Lambda penduradas por 15 minutos |

**Mitigações (CISA-aligned):**
- ✅ AWS Shield Standard (gratuito) / AWS Shield Advanced para DDoS volumétrico
- ✅ CloudFront/CDN na frente de workloads críticos
- ✅ WAF (Web Application Firewall) com rate limiting por IP e regras gerenciadas
- ✅ Budget alerts e billing anomaly detection
- ✅ Auto-scaling configurado adequadamente com limites máximos sensatos

---

### 2.6 Elevation of Privilege em Cloud

**Ameaças específicas:**

| Ameaça | Descrição | Exemplo |
|---|---|---|
| **IAM privilege escalation** | Encadeamento de permissões para obter admin access | `iam:CreateRole` + `iam:AttachRolePolicy` = admin |
| **Container escape** | Breakout de container para host subjacente | Privileged container com acesso ao host Docker socket |
| **Cross-account trust exploitation** | Abuso de trust relationships entre contas AWS | Assume role de conta dev para conta produção |
| **Workload identity abuse** | Uso indevido de identidade de serviço cloud | Pod K8s acessando recursos além de sua função |
| **Metadata credential theft** | SSRF para roubar credenciais do instance metadata | Acesso a role com permissões excessivas |

**Mitigações (CISA-aligned):**
- ✅ Análise de IAM para detecção de caminhos de privilege escalation (ferramentas: Cloudsplaining, PMapper)
- ✅ AWS Organizations SCPs (Service Control Policies) para guardrails organizacionais
- ✅ Kubernetes: PSA (Pod Security Admission), seccomp, AppArmor, rootless containers
- ✅ Least privilege rigoroso em service accounts e workload identities
- ✅ Privileged Access Management (PAM) para acesso administrativo com just-in-time provisioning

---

## 3. STRIDE em Diferentes Modelos de Serviço Cloud

### 3.1 IaaS (Infrastructure as a Service)

```
Responsabilidade do cliente: OS, runtime, middleware, aplicação, dados
Superfície de ataque STRIDE: Ampla — toda a stack acima da infraestrutura física
Foco: Hardening de VMs, network security groups, patch management
```

### 3.2 PaaS (Platform as a Service)

```
Responsabilidade do cliente: Aplicação e dados
Superfície de ataque STRIDE: Moderada — foco em aplicação e configuração
Foco: Configuração segura de serviços gerenciados, app-level security
```

### 3.3 SaaS (Software as a Service)

```
Responsabilidade do cliente: Dados e acesso
Superfície de ataque STRIDE: Menor, mas Information Disclosure e Elevation of Privilege são críticos
Foco: IAM, controle de acesso a dados, integrações e APIs de terceiros
```

---

## 4. Data Flow Diagram (DFD) Típico para Análise Cloud

```
                    [Internet]
                        │
                    ════╪════ Limite de Confiança Externo
                        │
              [CDN/WAF/CloudFront]
                        │
                    ════╪════ Limite DMZ
                        │
                 [API Gateway/LB]
                   /           \
           [Service A]      [Service B]
           (ECS/Lambda)     (ECS/Lambda)
                │                │
           ════╪════════════════╪════ Limite de Dados
                │                │
           [RDS/Aurora]    [DynamoDB]
           [ElastiCache]   [S3 Bucket]
                        │
                   [Secrets Manager]
                   [Parameter Store]
```

**Elementos a analisar com STRIDE:**
- Cada **fluxo de dados** (setas): Spoofing, Tampering, Information Disclosure, DoS
- Cada **processo** (caixas): todos os 6 elementos STRIDE
- Cada **armazenamento**: Tampering, Repudiation, Information Disclosure, DoS
- Cada **limite de confiança**: foco em Spoofing e Elevation of Privilege

---

## 5. Ferramentas de Análise STRIDE para Cloud

| Categoria | Ferramenta | Cloud | Função |
|---|---|---|---|
| **Threat Modeling** | Microsoft Threat Modeling Tool | Multi | Criação de DFDs e análise STRIDE |
| **CSPM** | AWS Security Hub | AWS | Detecção de misconfigurations |
| **CSPM** | Microsoft Defender for Cloud | Azure | Postura de segurança cloud |
| **IAM Analysis** | Cloudsplaining | AWS | Análise de privilege escalation paths |
| **IaC Scanning** | Checkov, tfsec | Multi | Análise de Terraform/CloudFormation |
| **Secret Detection** | GitLeaks, TruffleHog | Multi | Information Disclosure via repositórios |
| **Container Security** | Trivy, Grype | Multi | Análise de vulnerabilidades em imagens |
| **DLP** | AWS Macie | AWS | Classificação e proteção de dados sensíveis |

---

## 6. Checklist de Avaliação STRIDE para Cloud

### ✅ Spoofing
- [ ] MFA habilitado para todos os usuários
- [ ] Sem chaves de acesso estáticas de longa duração
- [ ] IMDSv2 habilitado em todas as instâncias EC2
- [ ] mTLS entre serviços internos

### ✅ Tampering
- [ ] Object versioning habilitado em buckets críticos
- [ ] CloudTrail com log file validation ativo
- [ ] IaC gerenciado via GitOps com aprovações
- [ ] Assinatura de artefatos no pipeline CI/CD

### ✅ Repudiation
- [ ] CloudTrail/Activity Log ativo em todas as regiões
- [ ] Logs protegidos contra deleção (Object Lock)
- [ ] Retenção de logs ≥ 90 dias online
- [ ] SIEM centralizado recebendo todos os logs

### ✅ Information Disclosure
- [ ] S3 Block Public Access habilitado no nível da conta
- [ ] Criptografia at-rest com CMK em dados sensíveis
- [ ] TLS 1.2+ obrigatório em todos os endpoints
- [ ] Zero wildcards em políticas IAM

### ✅ Denial of Service
- [ ] AWS Shield ou equivalente ativo
- [ ] WAF configurado com rate limiting
- [ ] Budget alerts e billing anomaly detection
- [ ] Auto-scaling com limites adequados

### ✅ Elevation of Privilege
- [ ] SCPs restritivas na AWS Organization
- [ ] Análise periódica de IAM para privilege escalation paths
- [ ] Containers executando sem root e com seccomp
- [ ] Just-in-time privileged access

---

## Referências

- CISA/NSA Cloud Security Best Practices (2024): https://www.cisa.gov/secure-cloud-business-applications
- CISA Threat Modeling Methodology (2023): https://www.cisa.gov/sites/default/files/2023-11/Joint_CSA_Threat_Modeling_Methodology.pdf
- NIST SP 800-144: Guidelines on Security and Privacy in Public Cloud Computing
- AWS Well-Architected Framework — Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/
- Microsoft Azure Security Benchmark: https://learn.microsoft.com/en-us/security/benchmark/azure/
- CSOH Cloud Threat Modeling (STRIDE, PASTA, ATT&CK): https://csoh.org/threat-modeling.html
