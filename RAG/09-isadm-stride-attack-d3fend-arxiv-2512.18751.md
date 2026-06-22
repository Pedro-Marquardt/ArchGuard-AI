# ISADM: Integrated STRIDE, ATT&CK, and D3FEND Model for Threat Modeling Against Real-World Adversaries

> **Fonte:** arXiv:2512.18751  
> **Autores:** Khondokar Fida Hasan, Hasibul Hossain Shajeeb, Chathura Abeydeera, Benjamin Turnbull, Matthew Warren  
> **Ano:** Dezembro de 2025  
> **URL:** https://arxiv.org/abs/2512.18751  
> **Categoria arXiv:** cs.CR  
> **Tipo:** Artigo científico — metodologia híbrida  
> **Relevância para o projeto:** Alta — combina STRIDE com inteligência de ameaças reais (ATT&CK) e contramedidas estruturadas (D3FEND), com validação em contexto FinTech/cloud

---

## Resumo

O ISADM (Integrated STRIDE-ATT&CK-D3FEND Model) é uma metodologia híbrida que funde três frameworks complementares para suprir as lacunas de cada um individualmente:

- **STRIDE**: Classificação de ameaças centrada em ativos do sistema
- **MITRE ATT&CK**: Catálogo de táticas, técnicas e procedimentos (TTPs) observados em ataques reais
- **MITRE D3FEND**: Base de conhecimento estruturada de contramedidas defensivas

O trabalho valida a abordagem no setor FinTech, onde conectividade crescente, APIs ubíquas e infraestrutura digital global criam desafios únicos de segurança.

---

## 1. Motivação: As Limitações de Frameworks Individuais

### 1.1 O que STRIDE faz bem — e o que não faz

**STRIDE é excelente para:**
- Decomposição sistemática de ameaças por categoria
- Análise de componentes específicos do sistema (DFD-driven)
- Identificação de propriedades de segurança violadas

**Limitações do STRIDE isolado:**
- Não diz *quem* vai atacar nem *como* — apenas *o quê* pode ser atacado
- Não prioriza ameaças por frequência real de ocorrência no mundo real
- Não conecta diretamente a contramedidas estruturadas
- Pode produzir listas exaustivas sem distinguir ameaças teóricas de ameaças ativas

### 1.2 O que MITRE ATT&CK adiciona

ATT&CK é uma base de conhecimento de táticas adversariais observadas em ataques reais, organizada em:
- **14 táticas** (Initial Access → Exfiltration → Impact)
- **Centenas de técnicas** com exemplos de grupos APT que as utilizam
- **Frequência de uso** observada em incidentes documentados

**Contribuição ao ISADM:** ATT&CK conecta as categorias abstratas do STRIDE a ataques concretos que grupos adversariais realmente empregam, permitindo priorização baseada em evidências.

### 1.3 O que D3FEND adiciona

D3FEND (MITRE) é o contraponto defensivo ao ATT&CK — uma base de conhecimento de **técnicas defensivas** estruturadas que mitigam técnicas de ataque específicas.

**Contribuição ao ISADM:** Para cada ameaça STRIDE mapeada a TTPs do ATT&CK, D3FEND fornece contramedidas específicas e validadas, saindo de mitigações genéricas para ações defensivas concretas.

---

## 2. Arquitetura do ISADM

```
┌─────────────────────────────────────────────────────────────────┐
│                        ISADM Framework                          │
├─────────────────┬───────────────────┬───────────────────────────┤
│   STRIDE Layer  │   ATT&CK Layer    │   D3FEND Layer            │
│                 │                   │                           │
│  "O QUE pode   │  "COMO atacantes  │  "COMO defender           │
│   ser atacado" │   fazem na prática"│   efetivamente"           │
│                 │                   │                           │
│  • Spoofing    │  • Initial Access │  • Credential Hardening   │
│  • Tampering   │  • Persistence    │  • Message Analysis       │
│  • Repudiation │  • Priv. Esc.     │  • Platform Monitoring    │
│  • Info Discl. │  • Defense Evasion│  • User Behavior Analysis │
│  • DoS         │  • Lateral Mov.   │  • Execution Isolation    │
│  • EoP         │  • Exfiltration   │  • Network Analysis       │
├─────────────────┴───────────────────┴───────────────────────────┤
│              Frequency-Based Scoring Mechanism                   │
│  Prioridade = f(STRIDE severity × ATT&CK frequency × D3FEND    │
│                  coverage)                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Mecanismo de Scoring por Frequência

A inovação central do ISADM é o **frequency-based scoring** que quantifica a prevalência de TTPs:

### 3.1 Fórmula de Score de Risco

```
ISADM_Score(ameaça) = STRIDE_Severity × ATT&CK_Frequency × (1 / D3FEND_Coverage)

Onde:
- STRIDE_Severity: valor 1-5 baseado na propriedade de segurança violada
- ATT&CK_Frequency: frequência normalizada da técnica nos dados do ATT&CK (0-1)
- D3FEND_Coverage: número de contramedidas disponíveis para a técnica (1-N)
```

### 3.2 Vantagem Sobre Scoring Tradicional

| Abordagem | Problema |
|---|---|
| **CVSS puro** | Foca na vulnerabilidade, não no contexto do atacante |
| **DREAD subjetivo** | Estimativas de probabilidade sem base em dados reais |
| **STRIDE sem scoring** | Lista de ameaças sem priorização |
| **ISADM frequency-based** | Priorização baseada em frequência observada de TTPs reais |

---

## 4. Mapeamento STRIDE → ATT&CK → D3FEND (Exemplos FinTech)

### 4.1 Spoofing → Credential Access (ATT&CK) → Credential Hardening (D3FEND)

**STRIDE:** Ameaça de falsificação de identidade em sistemas bancários

**ATT&CK TTPs mapeados (alta frequência em FinTech):**
- T1078: Valid Accounts (frequência: muito alta em campanhas APT financeiras)
- T1556: Modify Authentication Process
- T1539: Steal Web Session Cookie

**D3FEND Contramedidas:**
- D3-MFA: Multi-Factor Authentication
- D3-SCF: Strong Cryptographic Functions
- D3-SPP: Strong Password Policy

**ISADM Score:** Alto — T1078 é uma das técnicas mais frequentes em ataques financeiros

---

### 4.2 Information Disclosure → Exfiltration (ATT&CK) → Message Analysis (D3FEND)

**STRIDE:** Exposição de dados financeiros de clientes

**ATT&CK TTPs mapeados:**
- T1041: Exfiltration Over C2 Channel
- T1048: Exfiltration Over Alternative Protocol
- T1567: Exfiltration Over Web Service

**D3FEND Contramedidas:**
- D3-NTA: Network Traffic Analysis
- D3-DNSTA: DNS Traffic Analysis
- D3-PHDURA: Per Host Download-Upload Ratio Analysis

**ISADM Score:** Muito Alto — exfiltração é a fase final de quase todos os ataques financeiros bem-sucedidos

---

### 4.3 Elevation of Privilege → Privilege Escalation (ATT&CK) → Execution Isolation (D3FEND)

**STRIDE:** Escalada de privilégios em APIs bancárias

**ATT&CK TTPs mapeados:**
- T1548: Abuse Elevation Control Mechanism
- T1134: Access Token Manipulation
- T1611: Escape to Host (container environments)

**D3FEND Contramedidas:**
- D3-EI: Execution Isolation (sandboxing, containers)
- D3-UAP: User Account Permissions restriction
- D3-HBPI: Host-Based Process Isolation

---

## 5. Validação: Case Studies em FinTech

### 5.1 Sistema de Banking API

O ISADM foi aplicado a um sistema típico de open banking com:
- APIs REST para transações
- Serviço de autenticação OAuth 2.0 / OpenID Connect
- Banco de dados de contas
- Sistema de mensageria para notificações
- Serviço de KYC (Know Your Customer)

**Resultados da priorização ISADM:**
1. API Authentication Bypass (Score máximo) — T1078 + T1556, alta frequência em APT financeiro
2. Data Exfiltration via API (Score alto) — T1567, comum em breaches de Open Banking
3. Transaction Tampering (Score alto) — T1565 Data Manipulation, crítico para integridade financeira
4. Privilege Escalation via API Keys (Score médio-alto) — T1552 Unsecured Credentials

### 5.2 Correlação com Incidentes Reais

O paper demonstra que a priorização ISADM **replica padrões observados em incidentes reais** documentados em relatórios de inteligência de ameaças (FS-ISAC, IBM X-Force), validando que o scoring baseado em frequência ATT&CK efetivamente reflete o mundo real.

---

## 6. Comparação com Metodologias Existentes

| Metodologia | Asset-centric | Adversary TTPs | Contramedidas estruturadas | Score quantitativo |
|---|---|---|---|---|
| STRIDE | ✅ | ❌ | ❌ | ❌ |
| ATT&CK | ❌ | ✅ | ❌ (só via D3FEND) | Parcial |
| DREAD | ✅ | ❌ | ❌ | ✅ (subjetivo) |
| PASTA | ✅ | Parcial | ❌ | Parcial |
| **ISADM** | **✅** | **✅** | **✅** | **✅ (objetivo)** |

---

## 7. Implicações para o ArchGuard-AI

1. **Motor de análise multi-framework**: A arquitetura ISADM inspira um módulo no ArchGuard-AI que não apenas classifica ameaças via STRIDE, mas as enriquece com dados ATT&CK (frequência real) e D3FEND (contramedidas)
2. **Scoring objetivo**: O mecanismo frequency-based pode ser implementado usando a API do MITRE ATT&CK Navigator para dados atualizados de frequência de TTPs
3. **Contexto setorial**: O ArchGuard-AI pode adaptar o perfil de ameaças ATT&CK com base no setor informado (FinTech, saúde, infraestrutura crítica)
4. **Relatórios executivos**: A combinação STRIDE + score + contramedidas D3FEND gera relatórios muito mais acionáveis que listas brutas de ameaças

---

## Referências

- **Paper original:** https://arxiv.org/abs/2512.18751
- MITRE ATT&CK Framework: https://attack.mitre.org/
- MITRE D3FEND: https://d3fend.mitre.org/
- FS-ISAC (Financial Services ISAC): https://www.fsisac.com/
- IBM X-Force Threat Intelligence Index: https://www.ibm.com/reports/threat-intelligence
