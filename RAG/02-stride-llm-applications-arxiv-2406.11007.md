# Threat Modelling and Risk Analysis for Large Language Model (LLM)-Powered Applications

> **Fonte:** arXiv:2406.11007v1  
> **Autores:** Não especificados na extração  
> **Ano:** Junho de 2024  
> **URL:** https://arxiv.org/abs/2406.11007  
> **Tipo:** Artigo científico (pré-publicação)  
> **Relevância para o projeto:** Alta — framework STRIDE+DREAD para análise de ameaças em sistemas baseados em LLMs

---

## Resumo

Este artigo propõe um framework completo para modelagem de ameaças e análise de riscos especificamente voltado a **aplicações baseadas em Large Language Models (LLMs)**. O trabalho adapta as metodologias STRIDE e DREAD ao contexto único de sistemas de IA, cobrindo ameaças emergentes como prompt injection, data poisoning e extração de modelos.

---

## 1. Contexto e Motivação

O desenvolvimento acelerado de aplicações baseadas em LLMs trouxe novos vetores de ataque que as metodologias tradicionais de segurança não contemplavam adequadamente. As organizações enfrentam desafios para:

- Integrar threat modeling como prática regular no ciclo de desenvolvimento de sistemas de IA
- Adaptar frameworks existentes (STRIDE, DREAD) para capturar ameaças específicas de LLMs
- Fazer o threat modeling acessível a desenvolvedores sem formação em segurança

O Threat Modeling Manifesto (2020), criado por 15 pesquisadores de segurança e privacidade, forneceu os princípios orientadores:

- *Uma cultura de encontrar e corrigir problemas de design* > conformidade por checklist
- *Pessoas e colaboração* > processos e ferramentas
- *Uma jornada de entendimento* > um snapshot de segurança
- *Fazer threat modeling* > falar sobre ele
- *Refinamento contínuo* > entrega única

---

## 2. Framework Aplicado: Shostack's 4 Question Frame

O artigo estrutura a análise em torno das 4 perguntas fundamentais:

1. **O que estamos construindo?** — Uma aplicação LLM para processamento de linguagem natural
2. **O que pode dar errado?** — Ameaças identificadas via STRIDE
3. **O que vamos fazer a respeito?** — Mitigações específicas por categoria
4. **Fizemos um bom trabalho?** — Validação contínua e testes de segurança

---

## 3. Aplicação do STRIDE a Aplicações LLM

### 3.1 Spoofing (Falsificação)

**Ameaça identificada:** Acesso não autorizado à aplicação ou API usando credenciais falsas.

**Análise DREAD:**
- Dano: **Alto** — comprometimento potencial de informações sensíveis
- Reprodutibilidade: **Alto** — pode ser tentado por atores maliciosos sistematicamente
- Explorabilidade: **Médio** — depende da força dos mecanismos de autenticação

**Mitigações:**
- Autenticação multifator (MFA) obrigatória
- Rate limiting em endpoints de autenticação
- Detecção de anomalias comportamentais

---

### 3.2 Tampering (Adulteração)

**Ameaça identificada:** Modificação de dados de entrada para o modelo de linguagem com propósitos maliciosos — incluindo **prompt injection**.

**Análise DREAD:**
- Dano: **Alto** — outputs alterados ou tendenciosos pelo modelo
- Reprodutibilidade: **Médio** — requer compreensão dos mecanismos de entrada
- Explorabilidade: **Alto** — vetor de ataque extremamente comum em LLMs

**Mitigações:**
- Validação e sanitização de entrada (input validation)
- Verificação de integridade de conteúdo
- Sandboxing de prompts em ambientes isolados

---

### 3.3 Repudiation (Repúdio)

**Ameaça identificada:** Negação da execução de certas ações, como a geração de outputs específicos pelo modelo de linguagem.

**Análise DREAD:**
- Dano: **Médio** — pode levar a disputas ou falta de responsabilização
- Reprodutibilidade: **Médio** — depende do tipo de repúdio
- Explorabilidade: **Baixo a Médio** — pode exigir ataques sofisticados

**Mitigações:**
- Logging robusto e mecanismos de auditoria de todas as interações
- Registro imutável de prompts enviados e respostas recebidas
- Rastreamento de sessão para identificação inequívoca de ações

---

### 3.4 Information Disclosure (Divulgação de Informação)

**Ameaça identificada:** Acesso não autorizado a informações sensíveis geradas pelo modelo de linguagem (incluindo vazamento de dados de treinamento, PII, ou informações proprietárias).

**Análise DREAD:**
- Dano: **Alto** — exposição potencial de dados sensíveis
- Reprodutibilidade: **Médio** — depende da natureza da divulgação
- Explorabilidade: **Médio** — depende dos controles de segurança implementados

**Mitigações:**
- Criptografia de dados sensíveis em repouso e em trânsito
- Controles de acesso granulares às saídas do modelo
- Filtragem de PII nas respostas geradas
- Output sanitization para remover informações sensíveis

---

### 3.5 Denial of Service (Negação de Serviço)

**Ameaça identificada:** Sobrecarga da API com grande número de requisições para interromper o funcionamento normal (incluindo ataques específicos a LLMs via prompts computacionalmente custosos).

**Análise DREAD:**
- Dano: **Alto** — downtime do serviço e impacto na experiência do usuário
- Reprodutibilidade: **Alto** — vetor de ataque comum
- Explorabilidade: **Alto** — potencial de abuso via requisições massivas

**Mitigações:**
- Rate limiting e throttling por usuário/IP
- Análise de tráfego e detecção de padrões anômalos
- Circuit breakers para proteção do backend
- Limites de tamanho de prompt e tokens de resposta

---

### 3.6 Elevation of Privilege (Elevação de Privilégio)

**Ameaça identificada:** Obtenção de acesso não autorizado a privilégios mais altos dentro da aplicação ou sistema via manipulação do LLM (e.g., jailbreaking para contornar filtros de segurança, acessar ferramentas restritas ou executar ações privilegiadas).

**Análise DREAD:**
- Dano: **Alto** — comprometimento de componentes críticos do sistema
- Reprodutibilidade: **Baixo** — pode exigir múltiplas vulnerabilidades encadeadas
- Explorabilidade: **Baixo** — depende dos controles de segurança existentes

**Mitigações:**
- Princípio do menor privilégio em todas as ferramentas acessíveis pelo LLM
- Revisões regulares de controle de acesso
- Isolamento de capacidades do agente de IA
- Human-in-the-loop para ações de alto impacto

---

## 4. Ataques de Prompt Injection (Tabela de Referência)

O artigo detalha os principais tipos de ataques de prompt injection em aplicações LLM:

| Nome do Ataque | Descrição | 
|---|---|
| **Naive Attack** | Concatenação de dados alvo, instrução injetada e dados injetados diretamente no prompt |
| **Escape Characters** | Adição de caracteres especiais como `\n` e `\t` para quebrar o contexto do prompt |
| **Context Ignoring** | Texto de mudança de contexto para iludir o LLM a ignorar instruções originais |
| **Fake Completion** | Adição de uma resposta falsa para convencer o LLM que a tarefa alvo já foi concluída |
| **Combined Attack** | Combinação de Escape Characters, Context Ignoring e Fake Completion para máxima eficácia |

---

## 5. Lições Aprendidas e Contribuições

1. **Adaptação do STRIDE para LLMs**: O framework tradicional pode ser mapeado para ameaças de IA, mas requer extensões para cobrir vetores como prompt injection e model extraction.

2. **DREAD como quantificador**: A combinação STRIDE+DREAD fornece não apenas categorização, mas também priorização de riscos, essencial para times com recursos limitados.

3. **Shift Left obrigatório**: A segurança de aplicações LLM deve ser considerada desde o design, não como afterthought.

4. **Modelagem contínua**: Dada a rápida evolução das técnicas de ataque contra LLMs, o threat model deve ser revisado a cada atualização significativa do sistema.

5. **Acessibilidade**: O threat modeling precisa ser desmistificado e tornado acessível a desenvolvedores, não apenas especialistas em segurança.

---

## 6. Implicações para o ArchGuard-AI

Este paper é diretamente relevante para o projeto pois:

- Fornece um template de análise STRIDE aplicável ao próprio ArchGuard-AI (que é um sistema LLM-powered)
- Os vetores de ataque identificados (prompt injection, data poisoning) são riscos que o sistema precisa detectar em arquiteturas analisadas
- A combinação STRIDE+DREAD pode ser implementada como parte do pipeline de scoring de riscos do sistema
- O framework de 4 perguntas pode estruturar as interações do chatbot de análise arquitetural

---

## Referências

- **Paper original:** https://arxiv.org/abs/2406.11007
- Shostack, A. (2014). *Threat Modeling: Designing for Security*. Wiley.
- Threat Modeling Manifesto: https://www.threatmodelingmanifesto.org/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
