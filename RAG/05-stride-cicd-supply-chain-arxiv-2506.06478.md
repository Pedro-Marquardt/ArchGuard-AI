# Enhancing Software Supply Chain Security Through STRIDE-Based Threat Modelling of CI/CD Pipelines

> **Fonte:** arXiv:2506.06478v1  
> **Autores:** Sowmiya Dhandapani  
> **Ano:** Junho de 2025  
> **URL:** https://arxiv.org/abs/2506.06478  
> **Tipo:** Artigo científico (pré-publicação)  
> **Relevância para o projeto:** Muito Alta — framework STRIDE para análise de pipelines CI/CD com alinhamento NIST SSDF + SLSA + OWASP

---

## Resumo

Este artigo propõe uma metodologia estruturada baseada em STRIDE para segurança da **cadeia de suprimentos de software** (*software supply chain*), com foco em pipelines CI/CD. O trabalho decompõe sistematicamente as etapas de um pipeline moderno, aplica análise STRIDE a cada etapa, e mapeia ameaças identificadas a controles de frameworks reconhecidos (NIST SP 800-218/SSDF, OWASP Top 10 CI/CD, SLSA). O objetivo é habilitar **threat modeling contínuo e automatizado** como parte nativa dos processos de DevSecOps.

---

## 1. Contexto: A Crescente Ameaça à Supply Chain de Software

Ataques à cadeia de suprimentos de software tornaram-se um dos vetores mais críticos de ameaça nos últimos anos. Casos como:
- **SolarWinds (2020)**: Comprometimento do pipeline de build infectou milhares de clientes
- **Codecov (2021)**: Script de upload comprometido expôs tokens de CI/CD
- **XZ Utils (2024)**: Backdoor inserido diretamente em código open source via contribuidor malicioso

Demonstram que proteger o código-fonte não é suficiente — todo o pipeline de entrega deve ser tratado como superfície de ataque.

---

## 2. Decomposição do Pipeline CI/CD

O artigo decompõe um pipeline típico nas seguintes **etapas canônicas**:

```
┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Source  │───▶│ Build & Test │───▶│Containerização│───▶│  Artifact    │───▶│  Deployment  │
│  (Git)   │    │(Jenkins/GHA) │    │  (Docker)    │    │  Storage     │    │ (Kubernetes) │
└──────────┘    └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### Ativos críticos por etapa:

| Etapa | Ativos Críticos |
|---|---|
| **Source** | Código-fonte, configurações de CI, secrets no repositório |
| **Build & Test** | Ambiente de build, dependências, tokens de acesso |
| **Containerização** | Imagens de container, Dockerfiles, base images |
| **Artifact Storage** | Pacotes construídos, SBOMs, assinaturas |
| **Deployment** | Secrets de produção, configurações K8s, RBAC |

---

## 3. Análise STRIDE por Etapa do Pipeline

### 3.1 Etapa: Source (Repositório Git)

| Categoria STRIDE | Ameaça | Exemplo Concreto |
|---|---|---|
| **Spoofing** | Identidade de committer falsa | git commit com GPG key comprometida |
| **Tampering** | Modificação de código ou CI config | Push malicioso para branch protegida |
| **Repudiation** | Ausência de assinatura em commits | Commits anônimos sem rastreabilidade |
| **Information Disclosure** | Secrets hardcoded no repositório | API keys em arquivos .env comitados |
| **Denial of Service** | Abuso de webhooks para trigger de builds | Flood de events no repositório |
| **Elevation of Privilege** | Acesso indevido a branches protegidas | Bypass de branch protection rules |

**Controles:**
- Commits assinados com GPG obrigatórios
- Branch protection rules com revisão obrigatória
- Varredura automática de secrets (GitLeaks, TruffleHog)
- MFA obrigatório para todos os colaboradores

---

### 3.2 Etapa: Build & Test

| Categoria STRIDE | Ameaça | Exemplo Concreto |
|---|---|---|
| **Spoofing** | Dependência maliciosa simulando pacote legítimo | Dependency confusion attack |
| **Tampering** | Modificação do script de build em runtime | Comprometimento do build runner |
| **Repudiation** | Logs de build ausentes ou manipuláveis | Sem trilha auditável do processo de build |
| **Information Disclosure** | Exposição de secrets em logs de CI | Variáveis de ambiente impressas em debug |
| **Denial of Service** | Builds infinitos ou consumo excessivo de recursos | Loops infinitos em testes ou memory leaks |
| **Elevation of Privilege** | Runner com permissões excessivas | CI runner com acesso a produção |

**Controles:**
- Pinning de dependências com lock files verificados
- Build em ambientes efêmeros e isolados (ephemeral runners)
- Mascaramento automático de secrets nos logs
- Verificação de proveniência de pacotes (SLSA provenance)

---

### 3.3 Etapa: Containerização

| Categoria STRIDE | Ameaça | Exemplo Concreto |
|---|---|---|
| **Spoofing** | Uso de base image maliciosa ou não verificada | `FROM ubuntu:latest` sem digest verificado |
| **Tampering** | Modificação de camadas da imagem em trânsito | Registry sem TLS ou sem assinatura |
| **Information Disclosure** | Secrets embutidos em camadas da imagem | Credenciais copiadas durante build |
| **Elevation of Privilege** | Container executando como root | Ausência de instrução `USER` no Dockerfile |

**Controles:**
- Uso de digest imutável em vez de tags (`FROM ubuntu@sha256:...`)
- Image signing com Cosign/Notary
- Scanning de vulnerabilidades em imagens (Trivy, Grype)
- Dockerfile com usuário não-privilegiado obrigatório

---

### 3.4 Etapa: Artifact Storage

| Categoria STRIDE | Ameaça | Exemplo Concreto |
|---|---|---|
| **Tampering** | Substituição de artefato no registry após publicação | Ataque ao Artifactory ou ECR |
| **Information Disclosure** | Acesso não autorizado a artefatos proprietários | Registry público por erro de configuração |
| **Repudiation** | Ausência de SBOM (Software Bill of Materials) | Impossibilidade de auditar componentes |

**Controles:**
- Assinatura de artefatos (Sigstore/Cosign)
- Geração automática de SBOM (Syft, CycloneDX)
- Controle de acesso granular ao registry
- Imutabilidade de artefatos publicados

---

### 3.5 Etapa: Deployment (Kubernetes)

| Categoria STRIDE | Ameaça | Exemplo Concreto |
|---|---|---|
| **Spoofing** | Impersonação de service accounts | Roubo de tokens de SA do Kubernetes |
| **Tampering** | Modificação de manifests K8s durante deploy | Alteração de deployment.yaml em trânsito |
| **Elevation of Privilege** | RBAC excessivamente permissivo | `ClusterRole: cluster-admin` para workloads |
| **Information Disclosure** | Secrets do K8s em base64 sem criptografia | etcd sem criptografia at-rest |

**Controles:**
- RBAC granular com princípio do menor privilégio
- OPA/Gatekeeper para políticas de admissão
- Criptografia de secrets no etcd
- Network Policies para isolamento de pods
- Rotation automática de service account tokens

---

## 4. Alinhamento com Frameworks de Segurança

### 4.1 NIST SP 800-218 (SSDF — Secure Software Development Framework)

O SSDF organiza práticas de segurança em quatro grupos:

1. **PO (Prepare the Organization)**: Estabelecer práticas de desenvolvimento seguro
2. **PS (Protect the Software)**: Mecanismos para proteger código de modificações não autorizadas
3. **PW (Produce Well-Secured Software)**: Testes automatizados, análise estática/dinâmica
4. **RV (Respond to Vulnerabilities)**: Procedimentos de resposta a vulnerabilidades descobertas

**Sinergia STRIDE + SSDF:** STRIDE identifica ameaças sistêmicas; SSDF fornece práticas operacionais para implementar mitigações técnicas e processuais.

### 4.2 SLSA (Supply-chain Levels for Software Artifacts)

SLSA define um modelo de maturidade em 4 níveis para garantia de proveniência de artefatos:

| Nível | Requisitos |
|---|---|
| **SLSA 1** | Processo de build documentado, geração de provenance |
| **SLSA 2** | Build service autenticado, provenance assinada |
| **SLSA 3** | Build em ambiente isolado, provenance verificável por terceiros |
| **SLSA 4** | Builds reproduzíveis, revisão dupla de código |

**Mapeamento STRIDE ↔ SLSA:**
- Tampering na cadeia de build → SLSA 3+ (isolamento + provenance verificável)
- Spoofing de identidade → SLSA 2+ (builds autenticados)
- Repudiation → SLSA 1+ (provenance documentada)

### 4.3 OWASP Top 10 CI/CD Security Risks

O artigo mapeia as ameaças STRIDE às categorias do OWASP:

| OWASP Risco | Categoria STRIDE Correspondente |
|---|---|
| CICD-SEC-1: Insufficient Flow Control | Tampering, Elevation of Privilege |
| CICD-SEC-2: Inadequate Identity Management | Spoofing |
| CICD-SEC-3: Dependency Chain Abuse | Tampering (supply chain) |
| CICD-SEC-4: Poisoned Pipeline Execution | Tampering |
| CICD-SEC-6: Insufficient Credential Hygiene | Information Disclosure |
| CICD-SEC-9: Improper Artifact Integrity Validation | Tampering |

---

## 5. Implementação Prática: Security-as-Code

O artigo propõe codificar o threat model usando **OWASP pytm** — biblioteca Python que permite descrever sistemas e gerar análises STRIDE automaticamente:

```python
from pytm import TM, Server, Process, Dataflow, Boundary

# Definir o threat model como código
tm = TM("CI/CD Pipeline Threat Model")

github = Server("GitHub Repository")
ci_runner = Process("CI/CD Runner")
artifact_registry = Server("Artifact Registry")

# Definir fluxos de dados e analisar ameaças automaticamente
source_to_build = Dataflow(github, ci_runner, "Source Code + Config")
build_to_registry = Dataflow(ci_runner, artifact_registry, "Built Artifact + SBOM")

tm.process()  # Gera automaticamente ameaças STRIDE + mitigações
```

---

## 6. Automação no Pipeline CI/CD

```yaml
# Exemplo de integração em GitHub Actions
name: Security Threat Model Check
on: [push, pull_request]

jobs:
  threat-model:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run OWASP pytm Threat Model
        run: |
          pip install pytm
          python threat_model.py --report threats.json
          
      - name: Check for High-Risk Threats
        run: |
          # Gate: bloquear merge se ameaças críticas não mitigadas
          python check_threats.py --input threats.json --fail-on CRITICAL
          
      - name: Secret Scanning
        uses: gitleaks/gitleaks-action@v2
        
      - name: Container Image Scanning
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE_TAG }}
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
```

---

## 7. Implicações para o ArchGuard-AI

1. **Análise de arquiteturas CI/CD**: O framework fornece vocabulário e estrutura para o ArchGuard-AI analisar pipelines de entrega
2. **Integração com ArchGuard**: A abordagem Security-as-Code com pytm poderia se integrar ao ArchGuard para análise automática
3. **Relatórios estruturados**: O mapeamento STRIDE → SSDF → SLSA → OWASP fornece template para relatórios de análise multi-framework
4. **Gates de qualidade**: A lógica de gates por severidade pode ser implementada no sistema de scoring do ArchGuard-AI

---

## Referências

- **Paper original:** https://arxiv.org/abs/2506.06478
- NIST SP 800-218 (SSDF): https://csrc.nist.gov/publications/detail/sp/800-218/final
- SLSA Framework: https://slsa.dev/
- OWASP Top 10 CI/CD Security Risks: https://owasp.org/www-project-top-10-ci-cd-security-risks/
- OWASP pytm: https://github.com/OWASP/pytm
- Sigstore/Cosign: https://www.sigstore.dev/
