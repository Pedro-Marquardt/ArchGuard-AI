# RAG Knowledge Base — STRIDE Threat Modeling

> Base de conhecimento para Retrieval-Augmented Generation (RAG) do ArchGuard-AI  
> **Total de artigos:** 12  
> **Período coberto:** 2022–2026  
> **Fontes:** arXiv (cs.CR), CISA, literatura acadêmica revisada por pares  
> **Última atualização:** Junho de 2025

---

## Índice por Categoria

### 📚 Fundamentos e Metodologia

| Arquivo | Título | Ano | Fonte |
|---------|--------|-----|-------|
| [01](./01-stride-methodology-overview.md) | **STRIDE Methodology — Overview Completo** | Referência | Microsoft/OWASP/Manifesto |
| [06](./06-stride-cloud-security-cisa-aligned.md) | **STRIDE para Cloud Security — Guia CISA/NSA** | 2024 | CISA/NSA |

---

### 🤖 LLMs e Automação do STRIDE

| Arquivo | Título | arXiv | Ano | Destaque |
|---------|--------|-------|-----|----------|
| [02](./02-stride-llm-applications-arxiv-2406.11007.md) | Threat Modelling para Aplicações LLM (STRIDE+DREAD) | 2406.11007 | 2024 | Framework completo LLM security |
| [04](./04-llms-stride-network-security-arxiv-2505.04101.md) | LLMs para STRIDE em Redes 5G — Case Study | 2505.04101 | 2025 | ⭐ Insights críticos sobre falhas de LLMs |
| [10](./10-threatmodeling-llm-banking-arxiv-2411.17058.md) | ThreatModeling-LLM para Sistemas Bancários | 2411.17058 | 2024 | CoT + OPRO + LoRA fine-tuning |
| [12](./12-domain-adapted-llm-stride-empirical-arxiv-2605.10808.md) | Domain-Adapted LLMs para STRIDE — Avaliação Empírica | 2605.10808 | 2026 | ⭐ 52 configurações testadas, achados contraintuitivos |

---

### 🧠 STRIDE Extendido para Sistemas de IA

| Arquivo | Título | arXiv | Ano | Destaque |
|---------|--------|-------|-----|----------|
| [03](./03-stride-ai-framework-arxiv-2605.17163.md) | STRIDE-AI — Framework para Avaliação de IA Generativa | 2605.17163 | 2025/2026 | EU AI Act + NIST AI RMF alignment |
| [07](./07-astride-agentic-ai-arxiv-2512.04785.md) | ASTRIDE — Threat Modeling para Sistemas Agênticos | 2512.04785 | 2024 | ⭐ Categoria "A" para ataques de agentes |

---

### 🏗️ Arquiteturas Específicas

| Arquivo | Título | arXiv/Fonte | Ano | Destaque |
|---------|--------|-------------|-----|----------|
| [05](./05-stride-cicd-supply-chain-arxiv-2506.06478.md) | STRIDE em Pipelines CI/CD e Supply Chain | 2506.06478 | 2025 | NIST SSDF + SLSA + OWASP CI/CD |
| [08](./08-stride-microservices-distributed-architectures.md) | STRIDE para Microsserviços e Arquiteturas Distribuídas | Literatura 2022-2023 | 2022-2023 | Decomposição em 3 níveis, mTLS |

---

### 🔗 Frameworks Híbridos

| Arquivo | Título | arXiv | Ano | Destaque |
|---------|--------|-------|-----|----------|
| [09](./09-isadm-stride-attack-d3fend-arxiv-2512.18751.md) | ISADM — STRIDE + ATT&CK + D3FEND Híbrido | 2512.18751 | 2025 | Frequency-based scoring com TTPs reais |
| [11](./11-aegisshield-democratizing-threat-modeling-arxiv-2509.10482.md) | AegisShield — Democratizando Threat Modeling com GenAI | 2509.10482 | 2025 | NVD + OTX em tempo real, 85.4% ATT&CK accuracy |

---

## Papers Identificados Ainda Não Convertidos em Artigos Completos

> Papers encontrados na pesquisa — fontes para expansão futura da base RAG

| # | Título | arXiv | Ano | Relevância |
|---|--------|-------|-----|------------|
| A | From Prompt to Physical Actuation: STRIDE for LLM-Enabled Robots | 2604.27267 | 2026 | Alta — STRIDE DFD em edge-cloud com trust boundaries |
| B | ThreatGPT: Agentic AI for Multi-Framework Threat Modeling | 2509.05379 | 2025 | Média-Alta — STRIDE+ATT&CK+CVE+NIST+CISA em uma ferramenta |
| C | Multi-Cloud STRIDE+DREAD Risk Analysis | 2306.01862 | 2023 | Alta — STRIDE para AWS/Azure/GCP combinados, APIs como maior risco |
| D | An Integrity-Focused Threat Model for Software Development Pipelines | N/A | 2022 | Média — STRIDE para pipeline em 5 estágios |
| E | MCP-38: Threat Taxonomy for Model Context Protocol | N/A | 2026 | Alta — 38 categorias mapeadas a STRIDE+OWASP para AI agents |
| F | Taxonomy of AISecOps Threat Modeling for Cloud Medical Chatbots | N/A | 2023 | Média — STRIDE para cloud AI em setor regulado |

---

## Mapa de Conceitos Cross-Artigo

### Tema: LLMs como Ferramentas de STRIDE

**Consenso emergente dos papers 04, 10, 12:**
1. Zero-shot é insuficiente para produção (accuracy < 60%)
2. Few-shot com exemplos similares é o maior ganho de qualidade
3. **Greedy decoding** (temperatura = 0) é essencial para consistência
4. Repudiation é a categoria STRIDE mais difícil para LLMs
5. Domain adaptation não garante melhoria — pode ser prejudicial
6. Fine-tuning (LoRA) + CoT é a combinação que atinge melhor accuracy

### Tema: Perspectiva de Análise em STRIDE

**Papers 04, 12 convergem:**
- A perspectiva de análise (atacante vs. vítima vs. componente específico) deve ser **explícita** no prompt
- Perspectiva incorreta é a causa mais comum de classificações erradas por LLMs
- Recomendação: sempre especificar "analyze from the perspective of [component being attacked]"

### Tema: Extensões do STRIDE para IA

**Progressão identificada nos papers 02, 03, 07:**
- **STRIDE clássico** → ameaças de software determinístico
- **STRIDE + LLM threats** (paper 02) → adiciona prompt injection, model extraction
- **STRIDE-AI** (paper 03) → reformula todas as 6 categorias para falhas de ML
- **ASTRIDE** (paper 07) → adiciona 7ª categoria "A" específica para sistemas agênticos

### Tema: Integração com Outros Frameworks

**Tendência unificada (papers 05, 09, 10, 11):**
- STRIDE sozinho é **necessário mas não suficiente**
- Combinações mais robustas: STRIDE + ATT&CK (contexto real) + D3FEND (contramedidas)
- Alinhamento a compliance standards (NIST, ISO, OWASP) aumenta adoção prática

---

## Guia de Uso para RAG

### Queries Relevantes e Artigos Correspondentes

| Query | Artigos Relevantes |
|---|---|
| "O que é STRIDE?" | 01 |
| "Como aplicar STRIDE em cloud?" | 06, 08 |
| "STRIDE para microsserviços" | 08 |
| "STRIDE para CI/CD e DevSecOps" | 05 |
| "LLMs para threat modeling automatizado" | 02, 04, 10, 12 |
| "Como LLMs falham no STRIDE?" | 04, 12 |
| "STRIDE para sistemas de IA/LLMs" | 03, 07 |
| "Prompt injection como ameaça" | 02, 07 |
| "STRIDE + MITRE ATT&CK" | 09, 11 |
| "Scoring e priorização de ameaças" | 09, 11 |
| "STRIDE e compliance (NIST, EU AI Act)" | 03, 05, 10 |
| "Fine-tuning de LLMs para segurança" | 10, 12 |
| "Sistemas agênticos e ameaças de segurança" | 07 |
| "Democratizando threat modeling" | 11 |

---

## Fontes de Referência Adicionais

- **arXiv cs.CR:** https://arxiv.org/list/cs.CR/recent
- **CISA Threat Modeling Guide:** https://www.cisa.gov/sites/default/files/2023-11/Joint_CSA_Threat_Modeling_Methodology.pdf
- **MITRE ATT&CK:** https://attack.mitre.org/
- **MITRE D3FEND:** https://d3fend.mitre.org/
- **OWASP Threat Modeling:** https://owasp.org/www-community/Threat_Modeling
- **NIST AI RMF:** https://airc.nist.gov/
- **Google Scholar — termos sugeridos:**
  - "automated threat modeling STRIDE"
  - "cloud security architecture threat modeling"
  - "LLM for security compliance"
  - "STRIDE microservices DevSecOps"
