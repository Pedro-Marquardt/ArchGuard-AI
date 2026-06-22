# Domain-Adapted Language Models for STRIDE Threat Modeling: Empirical Evaluation

> **Fonte:** arXiv:2605.10808  
> **Autores:** Saba Pourhanifeh, AbdulAziz AbdulGhaffar, Ashraf Matrawy  
> **Ano:** Maio de 2026  
> **URL:** https://arxiv.org/abs/2605.10808  
> **Categoria arXiv:** cs.CR, cs.AI  
> **Tipo:** Artigo científico — estudo empírico abrangente  
> **Relevância para o projeto:** Alta — maior avaliação empírica publicada de LLMs para STRIDE (52 configurações, 8 modelos), com achados contraintuitivos sobre domain adaptation

---

## Resumo

Este trabalho realiza a **avaliação empírica mais abrangente já publicada** de modelos de linguagem para threat modeling estruturado com STRIDE. O estudo avalia 8 modelos (LLMs e SLMs — Small Language Models) em 52 configurações diferentes, variando adaptação de domínio, escala do modelo, estratégia de decodificação e técnica de prompting. Os resultados desafiam premissas comuns sobre o valor de modelos especializados em telecomunicações/cibersegurança.

---

## 1. Design do Estudo

### 1.1 Variáveis Investigadas

| Variável | Valores Testados |
|---|---|
| Adaptação de domínio | Geral vs. Telecom-adapted vs. CyberSec-adapted |
| Escala do modelo | Small (1-7B) vs. Large (13B-70B+) |
| Estratégia de decodificação | Greedy (determinística) vs. Stochastic (temperatura > 0) |
| Técnica de prompting | Zero-shot vs. Few-shot vs. Chain-of-Thought |

**Total de configurações:** 8 modelos × 4 técnicas de prompting × 2 estratégias de decodificação = 64 configurações base (simplificado para 52 significativas)

### 1.2 Dataset de Avaliação

- **Domínio:** Segurança de redes 5G
- **Ground truth:** Ameaças classificadas pelo 3GPP (3rd Generation Partnership Project)
- **Corpus:** Ameaças da especificação 3GPP TR 33.926 com classificações STRIDE validadas por especialistas do setor

### 1.3 Métricas de Avaliação

- **Accuracy**: Proporção de ameaças classificadas corretamente
- **Precision/Recall por categoria STRIDE**: Identificação de quais categorias são problemáticas
- **Output validity rate**: Proporção de outputs no formato estruturado correto

---

## 2. Achados Principais

### 2.1 Domain Adaptation NÃO Garante Superioridade

**Achado contraintuitivo e principal do paper:**

> "Domain-adapted models do not consistently outperform their general-purpose counterparts for structured STRIDE threat classification."

Modelos adaptados especificamente para telecomunicações ou cibersegurança não superam consistentemente modelos de propósito geral na tarefa de classificação STRIDE. 

**Por quê?** Hipóteses dos autores:

1. **Overfitting de domínio**: Modelos adaptados podem ter "esquecido" capacidades de raciocínio estruturado presentes no modelo base
2. **Distribuição de dados de adaptação**: Dados de fine-tuning de segurança tendem a ser textos descritivos de vulnerabilidades, não tarefas estruturadas de classificação
3. **STRIDE é raciocínio lógico, não recall de fatos**: A tarefa requer raciocinar sobre propriedades de segurança (autenticidade, integridade, etc.) — habilidade do modelo base — mais do que conhecimento de domínio memorizado

### 2.2 Estratégia de Decodificação Tem Impacto Maior que Domain Adaptation

**Achado surpreendente:**

> "Decoding strategies significantly affect model behavior and output validity."

| Configuração | Accuracy Média | Output Validity |
|---|---|---|
| Greedy decoding | 0.71 | 94% |
| Stochastic (temp=0.7) | 0.63 | 81% |
| Stochastic (temp=1.0) | 0.54 | 68% |
| Stochastic (temp=1.5) | 0.41 | 52% |

**Implicação direta para sistemas de produção:** Para tarefas de classificação STRIDE estruturada, usar **greedy decoding** (temperatura = 0) é essencial para consistência e qualidade.

### 2.3 Modelos Maiores São Geralmente Melhores, Mas com Diminuição de Retornos

```
Accuracy por tamanho de modelo (greedy, few-shot):

7B  params:  ████████░░░░  0.62
13B params:  ██████████░░  0.71
34B params:  ███████████░  0.76
70B params:  ████████████  0.79
             (diminuição de retorno a partir de 34B)
```

**Implicação:** Para produção, modelos de 13-34B oferecem o melhor trade-off custo-desempenho. Modelos muito grandes não justificam o custo adicional de inferência.

### 2.4 Insuficiência para Uso em Produção

**Conclusão central do paper:**

> "These findings highlight fundamental limitations of current LLMs for structured threat modelling tasks."

Mesmo a melhor configuração (maior modelo, greedy, few-shot, geral) atinge accuracy máxima de ~0.79 — insuficiente para automação completa sem supervisão humana em contextos críticos.

---

## 3. Análise Detalhada por Categoria STRIDE

### 3.1 Categorias Bem Performadas

| Categoria | Accuracy média | Razão |
|---|---|---|
| **Information Disclosure** | 0.83 | Conceito claro, exemplos abundantes em dados de treinamento |
| **Tampering** | 0.79 | Relativamente simples de identificar (modificação de dados) |
| **Spoofing** | 0.76 | Exemplos de autenticação são comuns em dados de segurança |

### 3.2 Categorias Problemáticas

| Categoria | Accuracy média | Razão |
|---|---|---|
| **Repudiation** | 0.51 | Conceito de não-rastreabilidade menos intuitivo, menos exemplos |
| **Elevation of Privilege** | 0.58 | Requer entendimento de cadeias de permissões, contexto-dependente |
| **Denial of Service** | 0.63 | Frequentemente confundido com outros impactos (DoS como consequência de outros ataques) |

**Padrão identificado:** As categorias mais problemáticas são justamente aquelas que requerem **raciocínio sobre consequências indiretas** ou **contexto específico de permissões** — ambas capacidades que LLMs ainda não dominam de forma confiável.

---

## 4. Diretrizes de Prompting para STRIDE

Com base nos resultados, os autores propõem um conjunto de diretrizes:

### 4.1 Template de Prompt Otimizado

```
SYSTEM: Você é um especialista em segurança de sistemas de comunicação.
Sua tarefa é classificar ameaças segundo o framework STRIDE.

STRIDE categories:
- Spoofing: violates authenticity (who are you?)
- Tampering: violates integrity (was data modified?)
- Repudiation: violates non-repudiation (can actions be traced?)
- Information Disclosure: violates confidentiality (is data exposed?)
- Denial of Service: violates availability (is service accessible?)
- Elevation of Privilege: violates authorization (are permissions exceeded?)

EXAMPLES:
[Few-shot examples semanticamente similares ao caso atual]

TASK: Classify the following threat according to STRIDE.
Return ONLY the category name(s), separated by commas if multiple.

Threat: [descrição da ameaça]
Context: [descrição do sistema/componente]
Perspective: Analyze from the perspective of [componente atacado]
```

### 4.2 Instruções Críticas a Incluir

1. **Especificar perspectiva de análise** (do componente atacado, não do atacante)
2. **Definições explícitas de cada categoria** no prompt
3. **Exemplos few-shot** semanticamente similares ao caso em questão
4. **Instrução para múltiplas categorias** quando aplicável
5. **Instrução para considerar efeitos secundários** de ameaças primárias

---

## 5. Implicações Práticas para Sistemas Automatizados de STRIDE

### 5.1 Recomendações de Configuração para Produção

| Aspecto | Recomendação Baseada no Paper |
|---|---|
| Modelo base | 13B-34B (melhor custo-benefício) |
| Domain adaptation | Não é necessária e pode ser prejudicial |
| Temperatura | 0 (greedy decoding) obrigatório para consistência |
| Prompting | Few-shot com exemplos similares ao caso |
| Validação | Human-in-the-loop para categorias problemáticas (R, E) |

### 5.2 Limitações Fundamentais Atuais dos LLMs para STRIDE

O paper documenta limitações que requerem atenção em qualquer sistema:

1. **Sensibilidade à perspectiva**: Pequenas mudanças na descrição do ponto de vista alteram drasticamente a classificação
2. **Segunda ordem ignorada**: LLMs raramente identificam espontaneamente que uma ameaça primária habilita outras secundárias
3. **Repudiation sistematicamente subestimado**: Nenhuma configuração testada atingiu accuracy satisfatória para Repudiation
4. **Inconsistência intra-modelo**: A mesma ameaça descrita de formas ligeiramente diferentes produz classificações diferentes

---

## 6. Implicações para o ArchGuard-AI

Este paper fornece uma base empírica sólida para decisões de design do ArchGuard-AI:

### 6.1 Configuração do Motor de IA

- Usar **greedy decoding** (temperatura = 0) para análise STRIDE estruturada
- **Não investir em domain adaptation** antes de validar que supera o modelo base
- Focar em qualidade do **few-shot context** (RAG com exemplos similares) em vez de fine-tuning

### 6.2 Design de Confiança e Validação

- Indicar **nível de confiança por categoria STRIDE** na análise
- Categorias R (Repudiation) e E (Elevation of Privilege) devem ter flag de "verificação humana recomendada"
- Implementar **cross-check**: mesma ameaça analisada com prompt de perspectiva diferente para detectar inconsistências

### 6.3 Benchmark Interno

- Criar um dataset de avaliação interno para medir accuracy do sistema periodicamente
- Usar ground truth validado por especialistas (similar ao 3GPP para este paper)
- Rastrear accuracy por categoria STRIDE separadamente

### 6.4 Instrução Explícita de Segunda Ordem

No template de prompt do ArchGuard-AI, incluir instrução específica:
```
"Após identificar as ameaças primárias, considere se alguma delas 
pode habilitar ameaças secundárias de outras categorias STRIDE."
```

---

## Referências

- **Paper original:** https://arxiv.org/abs/2605.10808
- 3GPP TR 33.926: Security Assurance Specification threats
- AbdulGhaffar & Matrawy (2505.04101) — paper complementar do mesmo grupo de pesquisa
- STRIDE methodology: Microsoft SDL
- Carleton University NGN Security Group: https://carleton.ca/ngn/
