# STRIDE-AI: A Threat Modeling Framework for Generative AI Security Assessment

> **Fonte:** arXiv:2605.17163v1  
> **Autores:** Não especificados na extração (pré-publicação 2025)  
> **Ano:** Maio de 2025  
> **URL:** https://arxiv.org/abs/2605.17163  
> **Tipo:** Artigo científico com ferramenta operacional  
> **Relevância para o projeto:** Alta — extensão do STRIDE especificamente para sistemas de IA/ML, alinhada com EU AI Act e NIST AI RMF

---

## Resumo

Este artigo apresenta o **STRIDE-AI**, uma adaptação do framework STRIDE clássico especificamente projetada para avaliar a segurança de sistemas de Inteligência Artificial, com foco em Large Language Models (LLMs). O trabalho introduz um ciclo de vida de avaliação em seis fases que unifica standards modernos (EU AI Act, NIST AI RMF, OWASP LLM Top 10) em um workflow executável, validado por estudos de caso e operacionalizado por uma ferramenta web dedicada.

---

## 1. Contexto e Motivação

### 1.1 O Problema da Lacuna de Segurança em IA

O crescimento exponencial de sistemas ML e LLMs transformou o ecossistema de cibersegurança. Os sistemas de IA migraram de pilotos experimentais para componentes centrais de infraestrutura, porém as metodologias de segurança não acompanharam esse ritmo. 

**Diferença fundamental** entre sistemas determinísticos e sistemas de IA:
- Sistemas tradicionais: entradas produzem saídas previsíveis
- Sistemas de IA: são **probabilísticos e dependentes de dados** — um código seguro não garante um modelo seguro se os dados de treinamento estiverem envenenados (data poisoning) ou se o modelo for suscetível a perturbações adversariais

### 1.2 Imperativo Regulatório

- O **EU AI Act** (2024) exige avaliações de risco rigorosas para sistemas de IA de "Alto Risco", criando um imperativo de conformidade para auditorias estruturadas
- O **2025 AI Threat Landscape Report** (HiddenLayer) revelou que:
  - 61% das organizações que implantam IA carecem de uma estratégia de segurança dedicada
  - Ataques adversariais aumentaram **30% ano a ano**

---

## 2. Contribuições Principais do STRIDE-AI

O framework STRIDE-AI apresenta três contribuições centrais:

1. **Ciclo de vida de avaliação em seis fases** que unifica standards modernos em um workflow executável
2. **Formalização do STRIDE-AI** que mapeia ameaças clássicas de software para modos de falha específicos de ML
3. **Ferramenta web operacional** que implementa a metodologia

---

## 3. Mapeamento STRIDE para Ameaças de IA/ML

### 3.1 Spoofing → Impersonação em Sistemas de IA

**Ameaças específicas de ML:**
- **Model Impersonation**: Substituição de um modelo legítimo por um adversarial
- **Data Source Spoofing**: Injeção de dados de treinamento maliciosos simulando fontes confiáveis
- **Identity Spoofing em APIs de IA**: Interceptação de chamadas a APIs de modelos

**Controles:**
- Verificação de proveniência de modelos (model cards, checksums)
- Autenticação mútua em endpoints de inferência
- Assinatura digital de datasets de treinamento

---

### 3.2 Tampering → Adulteração em Sistemas de IA

**Ameaças específicas de ML:**
- **Data Poisoning**: Envenenamento do conjunto de dados de treinamento para induzir comportamentos maliciosos
- **Model Tampering**: Modificação direta dos pesos do modelo
- **Adversarial Examples**: Perturbações nos inputs que causam erros sistemáticos de classificação
- **Prompt Injection**: Injeção de instruções maliciosas que alteram o comportamento do LLM

**Controles:**
- Validação de integridade do dataset (checksums, análise estatística de distribuição)
- Controle de versão e assinatura de artefatos de modelo
- Detecção de adversarial examples via input preprocessing
- Input sanitization e output filtering

---

### 3.3 Repudiation → Não-Rastreabilidade em Sistemas de IA

**Ameaças específicas de ML:**
- **Decision Repudiation**: Impossibilidade de rastrear qual versão do modelo tomou uma decisão específica
- **Training Data Repudiation**: Incapacidade de provar quais dados foram usados no treinamento
- **Prediction Repudiation**: Ausência de logs confiáveis de inferências críticas

**Controles:**
- MLOps logging completo de experimentos e deployments
- Model versioning com rastreamento de linhagem de dados
- Audit trails imutáveis para decisões críticas do modelo
- Explicabilidade (XAI) para justificação de predições

---

### 3.4 Information Disclosure → Vazamento de Informação em Sistemas de IA

**Ameaças específicas de ML:**
- **Training Data Extraction**: Técnicas para extrair dados de treinamento de LLMs (memorização)
- **Model Inversion**: Reconstrução de dados de treinamento a partir das saídas do modelo
- **Membership Inference**: Determinação se um dado específico foi usado no treinamento
- **System Prompt Leakage**: Exposição de prompts de sistema confidenciais

**Controles:**
- Differential Privacy no treinamento
- Técnicas de machine unlearning para remoção de dados sensíveis
- Limitação de informações retornadas em mensagens de erro
- Proteção de system prompts via mecanismos de segurança no serving

---

### 3.5 Denial of Service → Indisponibilidade em Sistemas de IA

**Ameaças específicas de ML:**
- **Sponge Attacks**: Inputs cuidadosamente construídos para maximizar o tempo de inferência
- **Model Extraction como DoS**: Consultas massivas para roubar o modelo esgotam recursos
- **Training Pipeline DoS**: Interrupção do processo de treinamento

**Controles:**
- Limites de tokens por requisição e por sessão
- Rate limiting inteligente baseado em custo computacional
- Detecção de padrões anômalos de consulta
- Infraestrutura escalável com circuit breakers

---

### 3.6 Elevation of Privilege → Escalada em Sistemas de IA

**Ameaças específicas de ML:**
- **Jailbreaking**: Técnicas para contornar filtros de segurança e acessar capacidades restritas
- **Goal Hijacking**: Manipulação dos objetivos de agentes de IA autônomos
- **Tool Misuse em Agentes**: LLMs com acesso a ferramentas sendo induzidos a realizar ações privilegiadas
- **Indirect Prompt Injection**: Injeção via conteúdo externo consumido pelo LLM (e.g., páginas web)

**Controles:**
- Least privilege para ferramentas acessíveis a agentes de IA
- Human-in-the-loop para ações de alto impacto
- Sandboxing de ambientes de execução de agentes
- Monitoramento comportamental de agentes em produção

---

## 4. Ciclo de Vida de Avaliação STRIDE-AI (6 Fases)

```
┌─────────────────────────────────────────────────────────────┐
│                    STRIDE-AI Assessment Lifecycle            │
├──────────┬──────────┬──────────┬──────────┬──────────┬──────┤
│ Phase 1  │ Phase 2  │ Phase 3  │ Phase 4  │ Phase 5  │ P6   │
│ System   │ Threat   │ Risk     │ Control  │ Tool     │Cont. │
│ Scoping  │ Enum.    │ Rating   │ Mapping  │ Testing  │Mon.  │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────┘
```

### Fase 1: System Scoping
- Identificar componentes do sistema de IA (dados, modelo, infraestrutura, interfaces)
- Mapear fluxos de dados e limites de confiança
- Documentar casos de uso e usuários/atores

### Fase 2: Threat Enumeration
- Aplicar cada categoria STRIDE-AI a cada componente
- Referenciar OWASP LLM Top 10 para cobertura adicional
- Documentar ameaças identificadas com contexto

### Fase 3: Risk Rating
- Avaliar cada ameaça usando critérios de impacto e probabilidade
- Alinhar com critérios do EU AI Act para sistemas de alto risco
- Priorizar ameaças para mitigação

### Fase 4: Control Mapping
- Mapear controles técnicos e processuais a cada ameaça
- Referenciar NIST AI RMF, ISO 42001, OWASP
- Identificar gaps de controle

### Fase 5: Tool-Based Testing
- Executar testes automatizados de segurança (red-teaming)
- Validar eficácia dos controles implementados
- Documentar resultados e evidências

### Fase 6: Continuous Monitoring
- Monitorar indicadores de segurança em produção
- Atualizar o threat model com novas ameaças identificadas
- Ciclo contínuo de melhoria

---

## 5. Alinhamento com Standards

| Standard | Cobertura no STRIDE-AI |
|---|---|
| **EU AI Act** | Avaliações de risco para sistemas de alto risco |
| **NIST AI RMF** | Funções: Map, Measure, Manage, Govern |
| **OWASP LLM Top 10** | Cobertura das 10 principais vulnerabilidades de LLMs |
| **ISO/IEC 42001** | Sistema de gestão de IA |
| **NIST SP 800-53** | Controles de segurança da informação |

---

## 6. Implicações para o ArchGuard-AI

O STRIDE-AI é especialmente relevante para o projeto porque:

1. **Framework de análise dual**: O ArchGuard-AI é simultaneamente um sistema de IA (sujeito às ameaças STRIDE-AI) e uma ferramenta para analisar outros sistemas
2. **Módulo de análise de compliance**: O mapeamento para EU AI Act e NIST AI RMF pode ser incorporado como funcionalidade de verificação de conformidade
3. **Expansão do motor de análise**: As categorias STRIDE-AI fornecem vocabulário específico para análise de arquiteturas de IA
4. **Red-teaming automatizado**: A Fase 5 do ciclo de vida pode inspirar casos de teste para o sistema

---

## Referências

- **Paper original:** https://arxiv.org/abs/2605.17163
- EU AI Act: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689
- NIST AI RMF: https://airc.nist.gov/RMF/Overview
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- HiddenLayer AI Threat Landscape Report 2025: https://hiddenlayer.com/threatreport/
