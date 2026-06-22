# LLMs' Suitability for Network Security: A Case Study of STRIDE Threat Modeling

> **Fonte:** arXiv:2505.04101v2  
> **Autores:** Pesquisadores da Carleton University (NGN Security Group)  
> **Ano:** Maio de 2025  
> **URL:** https://arxiv.org/abs/2505.04101  
> **Tipo:** Artigo científico com estudo de caso empírico  
> **Relevância para o projeto:** Alta — avalia empiricamente a capacidade de LLMs em realizar classificação STRIDE, com insights diretos para o motor de análise do ArchGuard-AI

---

## Resumo

Este trabalho investiga a adequação de Large Language Models (LLMs) para automatizar a classificação de ameaças segundo o framework STRIDE, usando como estudo de caso **ameaças e vulnerabilidades de redes 5G** (padronizadas pelo 3GPP). O estudo avalia quatro estratégias de prompting diferentes em cinco LLMs distintos, revelando padrões sistemáticos de acerto e falha nos modelos de linguagem para tarefas de segurança.

---

## 1. Motivação e Pergunta de Pesquisa

**Pergunta central:** LLMs são adequados para automatizar a classificação de ameaças de segurança usando STRIDE?

Com o crescimento do interesse em automação de processos de segurança via IA, entender as capacidades e limitações dos LLMs para threat modeling é essencial para:
- Determinar onde a automação pode ser confiável
- Identificar onde supervisão humana ainda é necessária
- Orientar o design de sistemas como o ArchGuard-AI

---

## 2. Metodologia

### 2.1 Dataset

O estudo utilizou ameaças e vulnerabilidades de segurança de redes 5G documentadas pelo **3GPP (3rd Generation Partnership Project)** — TR 33.926 — como baseline ground truth. Este conjunto fornece classificações STRIDE oficialmente estabelecidas por especialistas técnicos do setor.

### 2.2 Estratégias de Prompting Avaliadas

| Estratégia | Código | Descrição |
|---|---|---|
| Zero-Shot Base | ZS_Base | Prompt simples sem exemplos |
| Zero-Shot com contexto da internet | ZS_Internet | Prompt com instruções detalhadas |
| Few-Shot Base | FS_Base | Prompt com exemplos de classificação |
| Few-Shot com contexto da internet | FS_Internet | Exemplos + contexto adicional |

### 2.3 LLMs Avaliados

O estudo incluiu cinco modelos de linguagem de grandes fornecedores, incluindo modelos da família GPT (OpenAI), Gemini (Google) e outros. O **Gemini 2.5 Pro** apresentou o melhor desempenho geral.

---

## 3. Resultados e Insights Principais

### 3.1 Perspectiva de Ameaça Incorreta

**Caso: "AMF impersonation on N1 interface"**

- **Classificação baseline (3GPP):** Information Disclosure
- **Classificação pelos LLMs:** Unanimemente classificada como Spoofing
- **Razão identificada:** Os LLMs classificam a ameaça da perspectiva do **AMF** (o componente atacante), enquanto o 3GPP classifica da perspectiva do **usuário da rede 5G** (vítima)

**Insight crítico:** A perspectiva de análise (atacante vs. vítima) deve ser explicitamente definida no prompt para obter classificações corretas. Este é um **parâmetro implícito** que os LLMs assumem de forma inconsistente.

---

### 3.2 Falha na Identificação de Ameaças de Segunda Ordem

**Caso: "Eavesdropping on F1 interface"**

- **Classificação baseline (3GPP):** Information Disclosure + Spoofing + Tampering (múltiplas categorias)
- **Classificação pelos LLMs:** Quase exclusivamente Information Disclosure
- **Razão identificada:** LLMs não consideram sistematicamente os **efeitos de segunda ordem** — o fact de que a escuta pode também habilitar ataques de spoofing e tampering subsequentes

**Exemplo adicional — "5G-GUTI and IMEI correlation on N1 interface":**
- LLMs identificam corretamente como Information Disclosure
- Falham em reconhecer que este Information Disclosure pode levar a Tampering subsequente

**Insight crítico:** Para classificação STRIDE completa e precisa, LLMs precisam de instrução explícita para **raciocinar sobre cadeias de ameaças** e efeitos secundários.

---

### 3.3 Melhoria com Few-Shot Prompting

**Caso: "Bidding down on Xn-handover"**

- **ZS_Base:** Desempenho inconsistente
- **FS_Base:** 100% de acurácia com todos os LLMs testados
- **Razão:** O prompt few-shot incluía um exemplo similar ("Bidding down of Security features on N1 interface") com a mesma classificação STRIDE

**Insight crítico:** Few-shot prompting com exemplos semanticamente similares melhora dramaticamente o desempenho. Para o ArchGuard-AI, isso sugere que uma **base de casos de referência** (RAG) é fundamental para qualidade de análise.

---

### 3.4 Falha em Categoria de Repúdio em Ataques MiTM

**Caso: "MiTM attack on the N3 interface"**

- **Classificação baseline (3GPP):** Spoofing + Tampering + Information Disclosure + Repudiation
- **Classificação pelos LLMs:** Maioritariamente Spoofing + Tampering + Information Disclosure
- **Falha consistente:** Repudiation não identificado

**Raciocínio esperado (não realizado pelos LLMs):**
> Em um ataque MiTM sem medidas de segurança adequadas, o atacante que intercepta e modifica pacotes em trânsito torna difícil identificar qual entidade foi responsável pela modificação, criando uma situação de repúdio.

**Insight crítico:** O Repudiation é a categoria STRIDE mais difícil para LLMs identificarem de forma autônoma, especialmente quando é uma consequência indireta de outro ataque.

---

## 4. Análise de Desempenho por Estratégia de Prompting

```
Desempenho de Acurácia (aproximado, por estratégia):

ZS_Base     ████░░░░░░  ~40-50%  (baseline fraco)
ZS_Internet ██████░░░░  ~55-65%  (melhora com contexto)
FS_Base     ████████░░  ~75-85%  (grande salto com exemplos)
FS_Internet █████████░  ~80-90%  (melhor resultado geral)
```

**Observação geral:** A passagem de Zero-Shot para Few-Shot representa o maior ganho de desempenho. O contexto adicional ("Internet") ajuda, mas o principal driver é a presença de exemplos de referência.

---

## 5. Padrões Sistemáticos de Falha

| Tipo de Falha | Descrição | Frequência |
|---|---|---|
| **Perspectiva incorreta** | Análise do ponto de vista errado (atacante vs. vítima) | Alta |
| **Ameaças de segunda ordem** | Não considera efeitos cascata de ameaças | Alta |
| **Repudiation omitido** | Categoria de Repúdio consistentemente subidentificada | Muito Alta |
| **Categorização múltipla incompleta** | Identifica apenas a ameaça mais óbvia, omitindo secundárias | Alta |

---

## 6. Implicações para Design de Sistemas de Análise de Segurança por IA

### 6.1 Necessidade de Especificação de Perspectiva

Qualquer sistema que use LLMs para STRIDE deve:
- Especificar explicitamente qual perspectiva de análise usar (do componente atacado)
- Definir claramente o contexto do sistema sendo analisado
- Fornecer o nível de abstração adequado para a análise

### 6.2 Importância da Base de Referência (RAG)

Os resultados de few-shot prompting demonstram que:
- Uma base de casos de referência STRIDE bem curada melhora significativamente o desempenho
- Exemplos semanticamente similares são mais eficazes que exemplos genéricos
- **Arquiteturas RAG** são portanto a abordagem preferida para sistemas de threat modeling por IA

### 6.3 Human-in-the-Loop para Casos Complexos

Para ameaças que envolvem:
- Múltiplas categorias STRIDE simultaneamente
- Efeitos de segunda ordem não triviais
- Perspectivas de análise ambíguas

A supervisão humana deve ser mantida, com o LLM servindo como assistente em vez de árbitro final.

### 6.4 Foco em Repudiation

A categoria Repudiation requer tratamento especial:
- Incluir exemplos específicos de Repudiation no contexto few-shot
- Adicionar instrução explícita para considerar rastreabilidade e auditoria
- Verificar separadamente a cobertura desta categoria

---

## 7. Aplicações Diretas para o ArchGuard-AI

1. **Design de prompts**: Incorporar as melhores práticas identificadas (few-shot, perspectiva explícita, instrução para segunda ordem)
2. **Métricas de qualidade**: Monitorar especialmente a taxa de identificação de Repudiation
3. **Base RAG**: Este paper confirma a necessidade crítica de uma base de casos de referência curada
4. **Confiança calibrada**: O sistema deve indicar nível de confiança menor para classificações que tipicamente falham (Repudiation, efeitos secundários)
5. **Validação**: Usar datasets com ground truth de especialistas para avaliar e calibrar o modelo

---

## Referências

- **Paper original:** https://arxiv.org/abs/2505.04101
- Carleton University NGN Security Group: https://carleton.ca/ngn/2025/llm-stride/
- 3GPP TR 33.926: Security Assurance Specification (SCAS) threats and critical assets
- STRIDE methodology: Microsoft SDL (Security Development Lifecycle)
