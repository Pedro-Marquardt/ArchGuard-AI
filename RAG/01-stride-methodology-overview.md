# STRIDE Threat Modeling Methodology — Overview Completo

> **Fonte:** Microsoft (criação original), OWASP, Threat Modeling Manifesto  
> **Tipo:** Revisão conceitual / referência metodológica  
> **Relevância para o projeto:** Base conceitual para o motor de análise do ArchGuard-AI

---

## 1. O Que é STRIDE?

STRIDE é uma metodologia de *threat modeling* (modelagem de ameaças) criada pela Microsoft no final dos anos 1990 por Loren Kohnfelder e Praerit Garg. O objetivo é ajudar equipes de segurança e desenvolvimento a identificar sistematicamente categorias de ameaças em um sistema de software antes que vulnerabilidades sejam exploradas.

A sigla representa seis categorias de ameaças:

| Letra | Categoria | Propriedade Violada | Descrição |
|-------|-----------|---------------------|-----------|
| **S** | Spoofing (Falsificação) | Autenticidade | Impersonar um usuário, serviço ou componente legítimo |
| **T** | Tampering (Adulteração) | Integridade | Modificar dados ou código sem autorização |
| **R** | Repudiation (Repúdio) | Não-repúdio | Negar ter realizado uma ação (falta de trilha de auditoria) |
| **I** | Information Disclosure (Divulgação) | Confidencialidade | Expor informações a partes não autorizadas |
| **D** | Denial of Service (Negação de Serviço) | Disponibilidade | Tornar um sistema ou recurso indisponível |
| **E** | Elevation of Privilege (Elevação de Privilégio) | Autorização | Obter permissões além das autorizadas |

---

## 2. O Processo de Threat Modeling com STRIDE

O processo tipicamente segue quatro perguntas fundamentais (Shostack's 4 Question Frame):

1. **O que estamos construindo?** → Diagrama de fluxo de dados (DFD)
2. **O que pode dar errado?** → Enumeração de ameaças (STRIDE)
3. **O que vamos fazer a respeito?** → Mitigações e controles
4. **Fizemos um bom trabalho?** → Validação e revisão contínua

### 2.1 Data Flow Diagrams (DFDs)

O ponto de partida do STRIDE é a construção de um DFD que represente:

- **Processos** (círculos): componentes que processam dados
- **Armazenamentos** (retângulos com duas linhas): bancos de dados, arquivos
- **Entidades externas** (retângulos): usuários, sistemas externos
- **Fluxos de dados** (setas): movimentação de informações entre componentes
- **Limites de confiança** (linhas tracejadas): fronteiras de segurança

### 2.2 Aplicação por Elemento do DFD

Cada tipo de elemento do DFD é vulnerável a subconjuntos específicos do STRIDE:

| Elemento DFD | S | T | R | I | D | E |
|---|---|---|---|---|---|---|
| Processo | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Armazenamento | ✗ | ✓ | ✓* | ✓ | ✓ | ✗ |
| Fluxo de Dados | ✗ | ✓ | ✗ | ✓ | ✓ | ✗ |
| Entidade Externa | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ |

*Apenas logs de auditoria

---

## 3. Análise por Categoria de Ameaça

### 3.1 Spoofing (Falsificação)

**Definição:** Um atacante finge ser outro usuário, processo ou sistema para obter acesso não autorizado.

**Exemplos:**
- Roubo de tokens de autenticação (JWT, session cookies)
- Falsificação de endereço IP (IP spoofing) em comunicações internas
- Ataques Man-in-the-Middle (MiTM) em APIs
- Abuse de credenciais federadas (OAuth tokens)
- Impersonação de serviços em arquiteturas de microsserviços

**Mitigações:**
- Autenticação multifator (MFA)
- Certificados TLS mútuos (mTLS)
- Tokens de curta duração com rotação automática
- Validação rigorosa de identidade em chamadas entre serviços
- Zero Trust Architecture principles

---

### 3.2 Tampering (Adulteração)

**Definição:** Modificação não autorizada de dados, código ou configurações.

**Exemplos:**
- Injeção de código malicioso em pipelines CI/CD
- Adulteração de parâmetros de requisição HTTP
- Modificação de registros em banco de dados
- Ataques à cadeia de suprimentos de software (supply chain attacks)
- Alteração de arquivos de configuração em ambientes cloud

**Mitigações:**
- Assinatura digital de artefatos (code signing, SLSA)
- Controles de integridade de arquivos (checksums, hashes)
- Acesso baseado no princípio do menor privilégio
- Imutabilidade de artefatos em ambientes produtivos
- Validação de entrada e saída em APIs

---

### 3.3 Repudiation (Repúdio)

**Definição:** Capacidade de um agente negar ter realizado uma ação, geralmente devido à ausência de logs ou trilhas de auditoria.

**Exemplos:**
- Ausência de logs de acesso a dados sensíveis
- Falta de rastreamento de ações administrativas
- Logs que podem ser apagados ou modificados pelo próprio usuário
- Ausência de timestamps confiáveis em transações críticas

**Mitigações:**
- Logs imutáveis (append-only) com timestamps criptográficos
- Centralização de logs em sistemas auditados (SIEM)
- Non-repudiation em transações através de assinaturas digitais
- Trilhas de auditoria completas com identificação de sessão

---

### 3.4 Information Disclosure (Divulgação de Informação)

**Definição:** Exposição de informações confidenciais a partes não autorizadas.

**Exemplos:**
- S3 buckets públicos ou mal configurados na AWS
- Mensagens de erro com detalhes técnicos excessivos (stack traces)
- Dados sensíveis em logs de aplicação
- Tráfego não criptografado em redes internas
- Exposure de secrets/API keys em código-fonte (GitHub leaks)

**Mitigações:**
- Criptografia de dados em repouso e em trânsito (TLS 1.2+)
- Políticas de controle de acesso (IAM) baseadas no menor privilégio
- Sanitização de mensagens de erro
- Varredura automática de repositórios para secrets expostos
- Classificação e tratamento diferenciado de dados sensíveis (PII)

---

### 3.5 Denial of Service (Negação de Serviço)

**Definição:** Tornar um sistema, serviço ou recurso indisponível para usuários legítimos.

**Exemplos:**
- Ataques DDoS (Distributed Denial of Service)
- Esgotamento de recursos (CPU, memória, conexões de banco de dados)
- Rate limit bypass em APIs públicas
- Slow HTTP attacks contra servidores web
- Resource exhaustion em funções serverless (Lambda)

**Mitigações:**
- Rate limiting e throttling em APIs
- Circuit breakers e bulkheads em microsserviços
- CDN e WAF (Web Application Firewall)
- Auto-scaling baseado em métricas de carga
- Limites de recursos em containers (CPU/memory limits)

---

### 3.6 Elevation of Privilege (Elevação de Privilégio)

**Definição:** Obter permissões ou acessos além dos autorizados para a conta ou processo.

**Exemplos:**
- Container escape em ambientes Kubernetes
- Escalada de privilégios via SUID binaries no Linux
- Configurações excessivamente permissivas de IAM roles na AWS
- SQL injection que permite acesso de administrador
- RBAC mal configurado em clusters Kubernetes

**Mitigações:**
- Princípio do menor privilégio em todas as camadas
- Revisão periódica de permissões (access review)
- Segurança de containers (rootless, seccomp, AppArmor)
- Separação de ambientes (dev, staging, produção)
- Monitoramento de ações privilegiadas em tempo real

---

## 4. Ferramentas para Suporte ao STRIDE

| Ferramenta | Fabricante | Descrição |
|---|---|---|
| **Microsoft Threat Modeling Tool** | Microsoft | Ferramenta gráfica oficial com templates e geração automática de ameaças STRIDE |
| **OWASP Threat Dragon** | OWASP | Open source, baseado em browser, suporte a DFDs |
| **OWASP pytm** | OWASP | Threat modeling as code (Python) com geração de relatórios |
| **IriusRisk** | IriusRisk | Plataforma enterprise para threat modeling automatizado |
| **ThreatSpec** | ThreatSpec | Threat modeling no código-fonte via anotações |
| **Cairis** | Bournemouth University | Ferramenta acadêmica com suporte a múltiplos frameworks |

---

## 5. STRIDE no Contexto Moderno

### 5.1 Integração com DevSecOps

A tendência moderna é integrar o STRIDE diretamente nos pipelines de CI/CD:
- Threat modeling como código (TM-as-Code)
- Geração automática de DFDs a partir de IaC (Infrastructure as Code)
- Gates de segurança baseados em ameaças STRIDE identificadas
- Revisão contínua do threat model a cada mudança arquitetural

### 5.2 STRIDE + Outros Frameworks

STRIDE é frequentemente combinado com:
- **DREAD**: Para quantificação e priorização de riscos
- **PASTA**: Para análise orientada ao atacante
- **ATT&CK (MITRE)**: Para mapeamento a táticas e técnicas reais de adversários
- **CVSS**: Para scoring de vulnerabilidades identificadas

### 5.3 STRIDE para Sistemas de IA/ML

Com a proliferação de LLMs e sistemas de IA, surgiram adaptações do STRIDE:
- **STRIDE-AI**: Adapta as categorias para falhas específicas de ML (data poisoning, model extraction)
- **ASTRIDE**: Adiciona categoria "A" (Agent-specific) para ataques a sistemas agênticos (prompt injection, goal manipulation)

---

## 6. Limitações do STRIDE

1. **Subjetividade**: A classificação de ameaças pode variar entre analistas
2. **Escalabilidade**: Difícil de aplicar manualmente em sistemas muito grandes
3. **Foco em design**: Melhor aplicado no início do desenvolvimento; difícil adaptar em sistemas legados
4. **Não cobre todos os vetores**: Ameaças físicas, engenharia social e ameaças de negócio estão fora do escopo
5. **Requer expertise**: Analistas precisam de conhecimento técnico e de segurança para identificar ameaças relevantes

---

## 7. Referências

- Shostack, A. (2014). *Threat Modeling: Designing for Security*. Wiley.
- Microsoft Security Development Lifecycle (SDL): https://www.microsoft.com/en-us/securityengineering/sdl
- OWASP Threat Modeling: https://owasp.org/www-community/Threat_Modeling
- Threat Modeling Manifesto (2020): https://www.threatmodelingmanifesto.org/
- CISA Threat Modeling Guidance: https://www.cisa.gov/sites/default/files/2023-11/Joint_CSA_Threat_Modeling_Methodology.pdf
- NIST SP 800-218 (SSDF): https://csrc.nist.gov/publications/detail/sp/800-218/final
