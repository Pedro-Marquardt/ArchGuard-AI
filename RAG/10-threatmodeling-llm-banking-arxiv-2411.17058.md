# ThreatModeling-LLM: Automating Threat Modeling using Large Language Models for Banking Systems

> **Fonte:** arXiv:2411.17058  
> **Autores:** Tingmin Wu, Shuiqiao Yang, Shigang Liu, David Nguyen, Seung Jang, Alsharif Abuadbba  
> **Ano:** Novembro de 2024  
> **URL:** https://arxiv.org/abs/2411.17058  
> **Categoria arXiv:** cs.CR, cs.AI  
> **Tipo:** Artigo científico com framework e benchmark  
> **Relevância para o projeto:** Muito Alta — metodologia de fine-tuning de LLMs para STRIDE com CoT+OPRO+LoRA, diretamente aplicável ao motor de análise do ArchGuard-AI

---

## Resumo

O ThreatModeling-LLM é um framework de **três estágios** que automatiza threat modeling para sistemas bancários usando LLMs. As inovações centrais são: (1) criação de um benchmark dataset usando a Microsoft Threat Modeling Tool (TMT), (2) otimização de prompts via Chain-of-Thought (CoT) e OPRO (Optimization by PROmpting), e (3) fine-tuning com Low-Rank Adaptation (LoRA). O framework alinha as mitigações geradas com os controles do NIST SP 800-53, tornando a saída diretamente utilizável para compliance.

---

## 1. Problema e Motivação

O threat modeling manual para sistemas bancários complexos apresenta três desafios críticos:

### Desafio 1: Ausência de Datasets Específicos de Domínio

Não existiam datasets estruturados para treinar/avaliar LLMs em threat modeling de sistemas financeiros. Os LLMs genéricos carecem de:
- Conhecimento específico de arquiteturas bancárias (core banking, payment rails, open banking APIs)
- Padrões de ataque específicos do setor financeiro
- Mapeamento a controles regulatórios (PCI-DSS, SOX, BACEN)

### Desafio 2: Complexidade Arquitetural de Sistemas Bancários

Sistemas bancários modernos são altamente distribuídos:
```
[Mobile App] ──▶ [API Gateway] ──▶ [Microservices] ──▶ [Core Banking]
                      │                    │                    │
               [Auth Service]       [Payment Rail]     [Legacy Mainframe]
                      │                    │                    │
               [Identity Store]    [Message Queue]    [Database Cluster]
                                          │
                                  [Third-party APIs]
                                  [Open Banking]
                                  [Card Networks]
```

LLMs sem contexto adequado produzem ameaças genéricas inadequadas para esta complexidade.

### Desafio 3: Geração de Mitigações em Tempo Real Alinhadas a Compliance

Gerar mitigações que mapeiem a controles NIST 800-53, PCI-DSS ou outros frameworks regulatórios é desafiador para LLMs sem fine-tuning específico.

---

## 2. Framework ThreatModeling-LLM: 3 Estágios

### Estágio 1: Criação do Dataset Benchmark

**Ferramenta usada:** Microsoft Threat Modeling Tool (TMT)

O processo de criação do dataset:
1. Modelagem de 50+ arquiteturas bancárias típicas no TMT (core banking, digital banking, mobile banking, open banking APIs)
2. Geração automática de ameaças STRIDE pelo TMT para cada arquitetura
3. Revisão e validação por especialistas em segurança bancária
4. Estruturação em formato para fine-tuning de LLMs

**Formato do dataset:**
```json
{
  "system_description": "API Gateway para Open Banking com autenticação OAuth 2.0...",
  "architecture_components": ["API Gateway", "OAuth Server", "Core Banking API", "Customer DB"],
  "stride_threats": [
    {
      "category": "Spoofing",
      "threat": "Um atacante pode se passar por um cliente legítimo usando tokens OAuth vazados",
      "component": "OAuth Server",
      "mitigation": "Implementar PKCE (Proof Key for Code Exchange) para todos os fluxos OAuth",
      "nist_control": "IA-2(1): Identification and Authentication — Network Access to Privileged Accounts"
    }
  ]
}
```

---

### Estágio 2: Otimização de Prompts (CoT + OPRO)

#### 2.1 Chain-of-Thought (CoT) para STRIDE

O CoT força o LLM a raciocinar passo a passo antes de gerar ameaças:

**Prompt sem CoT (resultado genérico):**
```
Sistema: API de transferência bancária
Ameaças STRIDE?
→ "Spoofing: identidade falsa, Tampering: dados alterados..." (genérico, pouco útil)
```

**Prompt com CoT (resultado específico e acionável):**
```
Sistema: API de transferência bancária com JWT, PostgreSQL, Redis cache

Pense passo a passo:
1. Identifique os componentes e seus fluxos de dados
2. Para cada fluxo, pense quem o origina e quem o recebe
3. Para cada categoria STRIDE, pense em ataques específicos NESTE contexto
4. Gere mitigações mapeadas a controles NIST 800-53

→ "Spoofing: JWT token replay attack contra endpoint /transfer — atacante captura 
   token válido e reenvia transação. Mitigação: JWT jti (JWT ID) único + Redis 
   blacklist de tokens usados. NIST IA-2(8): Replay-Resistant Authentication."
```

#### 2.2 OPRO (Optimization by PROmpting)

OPRO é uma técnica onde o próprio LLM otimiza seus prompts iterativamente:

```
Iteração 1: Prompt inicial → Output de baixa qualidade → Score: 0.42
                    ↓
Iteração 2: LLM sugere melhoria do prompt → Output melhorado → Score: 0.61
                    ↓
Iteração 3: Nova melhoria de prompt → Output refinado → Score: 0.78
                    ↓
Iteração N: Prompt otimizado → Score estabilizado: 0.85+
```

**Resultado do OPRO:** Os prompts otimizados automaticamente superaram consistentemente prompts desenhados manualmente por especialistas em +15-20% de precision@k.

---

### Estágio 3: Fine-Tuning com LoRA

#### 3.1 Por Que LoRA?

LoRA (Low-Rank Adaptation) permite fine-tuning eficiente de LLMs grandes:
- Adiciona matrizes de baixo rank aos pesos existentes
- Requer muito menos memória que full fine-tuning (~100x menos parâmetros treináveis)
- Resultado comparável ao full fine-tuning para tarefas de domínio específico

```
Modelo base (frozen): Llama/Mistral/GPT class
        ↓
LoRA adapters (trainable): matrizes de baixo rank
        ↓
Domínio: Threat modeling bancário + STRIDE + NIST 800-53
        ↓
Output: LLM especializado em segurança bancária
```

#### 3.2 Resultados do Fine-Tuning

| Modelo | Zero-Shot | Few-Shot | LoRA Fine-tuned |
|---|---|---|---|
| Baseline LLM | 0.34 | 0.52 | — |
| + CoT | 0.48 | 0.65 | — |
| + CoT + OPRO | 0.55 | 0.72 | — |
| + CoT + OPRO + LoRA | — | — | **0.87** |

*Métricas: ROUGE-L + domain expert evaluation*

---

## 3. Resultados Principais

### 3.1 Melhoria de Qualidade

1. **CoT sozinho**: +41% de melhoria vs. zero-shot em identificação de ameaças específicas ao domínio
2. **OPRO** adicional: +15% sobre CoT, especialmente em mapeamento a controles NIST
3. **LoRA fine-tuning**: Salto mais significativo — output especializado que reconhece padrões bancários específicos

### 3.2 Alinhamento com NIST 800-53

O framework ThreatModeling-LLM é o **primeiro** a gerar automaticamente mitigações STRIDE mapeadas a controles específicos do NIST 800-53, permitindo:
- Relatórios de threat modeling com evidências de compliance
- Rastreamento de gaps de controle em sistemas bancários
- Integração com ferramentas de GRC (Governance, Risk, Compliance)

### 3.3 Generalização

Embora treinado em banking, o framework demonstrou capacidade de generalização para:
- Sistemas de healthcare com PII sensível
- Infraestruturas de pagamento (PCI-DSS context)
- APIs de Open Finance

---

## 4. Arquitetura do Pipeline de Inferência

```
Input: Descrição de arquitetura em linguagem natural ou DFD
         │
         ▼
[Preprocessamento]
├── Extração de componentes
├── Identificação de fluxos de dados
└── Detecção de tecnologias (JWT, OAuth, REST, gRPC, etc.)
         │
         ▼
[CoT-Optimized Prompt Construction]
├── Template OPRO otimizado
├── Inserção de contexto arquitetural
└── Instruções de mapeamento NIST
         │
         ▼
[LoRA Fine-tuned LLM]
         │
         ▼
[Output Estruturado]
├── Ameaças STRIDE por componente
├── Score de severidade
├── Mitigações específicas
└── Controles NIST 800-53 mapeados
         │
         ▼
Relatório de Threat Model com evidências de compliance
```

---

## 5. Lições Aprendidas para Automação de STRIDE com LLMs

### 5.1 A Ordem Importa: Dataset → Prompt → Fine-tune

Tentar fine-tuning sem dataset de qualidade ou com prompts ruins desperdiça recursos computacionais. A sequência otimizada é:
1. Primeiro criar dataset de alta qualidade (mesmo que pequeno: 200-500 exemplos são suficientes com LoRA)
2. Depois otimizar prompts com OPRO
3. Por último, fine-tuning LoRA com prompts otimizados

### 5.2 CoT é Obrigatório para Tarefas Estruturadas

Para classificação STRIDE, o modelo **precisa** raciocinar explicitamente sobre:
- Qual componente está sendo analisado
- Qual é o fluxo de dados envolvido
- Qual propriedade de segurança é violada

Sem CoT, LLMs tendem a pattern-matching superficial que produz ameaças genéricas.

### 5.3 Contexto Arquitetural é Essencial

Incluir no contexto:
- Tecnologias específicas usadas (tipo de banco de dados, protocolo de auth, etc.)
- Dados processados (PII, dados financeiros, credentials)
- Ambiente de deployment (cloud provider, on-prem, hybrid)

Esse contexto específico é o que transforma ameaças genéricas em ameaças acionáveis.

---

## 6. Implicações para o ArchGuard-AI

Este paper tem implicações diretas de implementação para o ArchGuard-AI:

### 6.1 Dataset Strategy

- Criar um dataset próprio de arquiteturas de software com análises STRIDE anotadas
- Usar o Microsoft TMT para geração inicial + curadoria humana
- Expandir gradualmente com feedback de usuários (active learning)

### 6.2 Prompt Engineering

- Implementar CoT nos prompts de análise STRIDE do ArchGuard-AI
- Explorar OPRO para otimização automática dos templates de prompt
- Incluir tecnologias detectadas na arquitetura no contexto do prompt

### 6.3 Fine-Tuning Roadmap

- Fase 1: Prompt engineering (CoT + templates especializados)
- Fase 2: Construção do dataset de benchmark
- Fase 3: LoRA fine-tuning em modelo base (Llama 3 / Mistral)
- Fase 4: Alignment com múltiplos frameworks (NIST, ISO 27001, PCI-DSS, LGPD)

### 6.4 Compliance Integration

A abordagem de mapeamento automático a controles NIST 800-53 pode ser extendida para:
- ISO/IEC 27001 Annex A controls
- PCI-DSS v4.0 requirements
- LGPD (Lei Geral de Proteção de Dados) requirements
- CIS Controls v8

---

## Referências

- **Paper original:** https://arxiv.org/abs/2411.17058
- Microsoft Threat Modeling Tool: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool
- NIST SP 800-53 Rev.5: https://csrc.nist.gov/publications/detail/sp/800-53/5-0/final
- LoRA (Low-Rank Adaptation): Hu et al. (2021) — https://arxiv.org/abs/2106.09685
- OPRO: Yang et al. (2023) — https://arxiv.org/abs/2309.03409
- Chain-of-Thought: Wei et al. (2022) — https://arxiv.org/abs/2201.11903
