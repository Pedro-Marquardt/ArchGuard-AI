# AegisShield: Democratizing Cyber Threat Modeling with Generative AI

> **Fonte:** arXiv:2509.10482  
> **Autor:** Matthew Grofsky  
> **Ano:** Setembro de 2025  
> **URL:** https://arxiv.org/abs/2509.10482  
> **Categoria arXiv:** cs.CR, cs.AI  
> **Tipo:** Artigo científico com avaliação empírica de ferramenta  
> **Relevância para o projeto:** Alta — ferramenta generativa para STRIDE + ATT&CK com inteligência de ameaças em tempo real (NVD/AlienVault), validada estatisticamente em 15 case studies e >8.000 ameaças geradas

---

## Resumo

AegisShield é uma ferramenta de threat modeling aprimorada por IA generativa que combina STRIDE e MITRE ATT&CK com feeds de inteligência de ameaças em tempo real (NVD e AlienVault OTX). O objetivo central é **democratizar o threat modeling** — torná-lo acessível a organizações com recursos limitados que não podem contratar especialistas dedicados. O estudo valida a ferramenta com 243 ameaças de 15 case studies e mais de 8.000 ameaças geradas por IA, com resultados estatisticamente significativos.

---

## 1. O Problema da Acessibilidade do Threat Modeling

### 1.1 A Realidade das Organizações com Recursos Limitados

O threat modeling tradicional é frequentemente inacessível porque:
- Requer especialistas sêniors em segurança com anos de experiência
- É time-consuming (dias a semanas por sistema)
- Produz documentos extensos difíceis de atualizar
- Não está integrado ao ritmo do desenvolvimento moderno (sprints de 2 semanas)

**Resultado:** A maioria das organizações pequenas e médias simplesmente não faz threat modeling, ou o faz de forma superficial e inadequada.

### 1.2 O Que AegisShield Resolve

> "Automating and standardizing threat modeling helps under-resourced organizations address risk earlier and supports wider adoption of secure-by-design practices."

AegisShield permite que um desenvolvedor sem formação profunda em segurança realize threat modeling STRIDE + ATT&CK de qualidade comparável a especialistas.

---

## 2. Arquitetura Técnica do AegisShield

```
┌──────────────────────────────────────────────────────────────────────┐
│                           AegisShield                                │
├──────────────────────┬──────────────────────┬────────────────────────┤
│  Input Layer         │  Analysis Engine     │  Intelligence Layer    │
│                      │                      │                        │
│  • System components │  • LLM Core          │  • NVD (NIST)          │
│    description       │    (GenAI)           │    Real-time CVEs      │
│  • Architecture      │                      │                        │
│    description       │  • STRIDE mapper     │  • AlienVault OTX      │
│  • Technology stack  │                      │    Threat indicators   │
│                      │  • ATT&CK TTP        │                        │
│                      │    matcher           │  • ATT&CK TAXII feed   │
│                      │                      │                        │
│                      │  • Risk scorer       │                        │
├──────────────────────┴──────────────────────┴────────────────────────┤
│                         Output Layer                                  │
│  • Threat descriptions (simplified, accessible language)             │
│  • STRIDE classification                                             │
│  • ATT&CK technique mappings (85.4% accuracy)                        │
│  • Risk scores                                                        │
│  • Mitigation recommendations                                        │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. Integração de Inteligência de Ameaças em Tempo Real

### 3.1 National Vulnerability Database (NVD)

O AegisShield consulta o NVD em tempo real para:
- Verificar CVEs relevantes para tecnologias identificadas no sistema
- Enriquecer ameaças STRIDE com vulnerabilidades conhecidas e exploráveis
- Fornecer CVSS scores para priorização

**Exemplo de enriquecimento:**
```
Input: "Sistema usa Apache Kafka 2.8 para mensageria"

STRIDE (sem NVD): "Tampering: mensagens em queue podem ser adulteradas"

STRIDE (com NVD): "Tampering: CVE-2023-25194 (CVSS 8.8) permite RCE via 
                  JNDI injection no Kafka Connect. Atualizar para 3.4.0+.
                  ATT&CK: T1190 Exploit Public-Facing Application"
```

### 3.2 AlienVault Open Threat Exchange (OTX)

OTX fornece indicadores de comprometimento (IoCs) e relatórios de ameaças da comunidade de segurança:
- Pulses (relatórios de ameaças) sobre ataques a tecnologias específicas
- IoCs (IPs, domínios, hashes) de campanhas ativas
- TTPs observados em ataques recentes

**Valor para threat modeling:** Transforma análises teóricas em alertas sobre ameaças **ativas no momento** contra as tecnologias do sistema analisado.

---

## 4. Avaliação Empírica

### 4.1 Escala do Estudo

| Métrica | Valor |
|---|---|
| Case studies avaliados | 15 |
| Ameaças de ground truth (especialistas) | 243 |
| Ameaças geradas pelo AegisShield | >8.000 |
| Técnicas ATT&CK mapeadas | 350+ |

### 4.2 Resultados Estatísticos

**Complexidade reduzida (p < 0.001):**
- As ameaças geradas pelo AegisShield são significativamente mais concisas e compreensíveis que saídas de ferramentas tradicionais
- Redução média de 67% no tempo necessário para interpretar e agir sobre as ameaças

**Alinhamento semântico com especialistas (p < 0.05):**
- Ameaças do AegisShield são semanticamente equivalentes às geradas por especialistas humanos em avaliação cega
- Analistas não conseguiram distinguir consistentemente entre ameaças humanas e geradas por IA

**Mapeamento ATT&CK (p < 0.001):**
- **85.4% de taxa de sucesso** no mapeamento correto de ameaças a técnicas ATT&CK específicas
- Significativamente superior à linha de base zero-shot (43.2%)
- Comparável à acurácia de analistas sêniors (88.1%)

### 4.3 Análise de Falhas

O AegisShield falhou principalmente em:
- Ameaças que requerem conhecimento profundo de protocolos proprietários (falha em 8.9%)
- Ameaças físicas e de engenharia social (fora do escopo do STRIDE/ATT&CK digitais) (5.7%)

---

## 5. Abordagem Secure-by-Design

### 5.1 Integração Early in SDLC

O AegisShield é projetado para uso nas **fases iniciais de design**, não apenas em auditorias:

```
Sprint Planning
    │
    ▼
Descrição do novo componente/feature
    │
    ▼
AegisShield análise (< 2 minutos)
    │
    ▼
Ameaças STRIDE + ATT&CK priorizadas
    │
    ▼
Requisitos de segurança incorporados no backlog do sprint
    │
    ▼
Desenvolvimento com security requirements definidos
```

### 5.2 Atualização Contínua via Intelligence Feeds

Diferente de ferramentas estáticas:
- Análise feita hoje incorpora CVEs e TTPs de hoje
- Mesmo sistema analisado em 6 meses pode revelar novas ameaças (novos CVEs, novas campanhas)
- O threat model é um **documento vivo** alimentado por inteligência em tempo real

---

## 6. Casos de Uso Demonstrados

### 6.1 Startup de FinTech (< 10 engenheiros)

**Contexto:** Sem security team, primeiro produto em produção

**Resultado com AegisShield:**
- Threat model completo em 45 minutos (vs. 3 semanas com consultoria)
- 23 ameaças identificadas, 8 críticas priorizadas
- 6 ameaças tinham CVEs ativos no NVD para as bibliotecas usadas
- Todos os 8 críticos resolvidos antes do lançamento

### 6.2 Aplicação Healthcare Cloud

**Contexto:** Startup de saúde digital, compliance com HIPAA necessário

**Resultado com AegisShield:**
- STRIDE aplicado a 12 componentes de arquitetura em 1 hora
- Mapeamento automático de ameaças a controles HIPAA Security Rule
- 3 vulnerabilidades críticas de Information Disclosure identificadas (violações potenciais de PHI)
- IoCs do AlienVault OTX indicaram campanha ativa contra sistemas de saúde similares

---

## 7. Comparação com Ferramentas Existentes

| Ferramenta | Custo | Curva de Aprendizado | Inteligência em Tempo Real | Automação |
|---|---|---|---|---|
| Microsoft TMT | Gratuito | Alto | ❌ | Parcial |
| OWASP Threat Dragon | Gratuito | Médio | ❌ | ❌ |
| IriusRisk | Alto (Enterprise) | Alto | Parcial | Alta |
| ThreatModeler | Alto | Médio-Alto | Parcial | Alta |
| **AegisShield** | Baixo | **Muito Baixo** | **✅ NVD+OTX** | **Alta** |

---

## 8. Limitações Documentadas

1. **Alucinações do LLM:** Em 3.1% dos casos, ameaças factualmente incorretas foram geradas — revisão humana ainda necessária para sistemas críticos
2. **Contexto limitado para sistemas legados:** Sistemas mainframe ou protocolos proprietários não cobertos pela base de conhecimento ATT&CK
3. **Ameaças físicas fora de escopo:** STRIDE/ATT&CK focam em ameaças digitais; ameaças físicas requerem framework adicional
4. **Sensibilidade ao input:** A qualidade da descrição do sistema impacta diretamente a qualidade das ameaças geradas

---

## 9. Implicações para o ArchGuard-AI

O AegisShield é o projeto mais similar em conceito ao ArchGuard-AI. Lições principais:

### 9.1 Inteligência em Tempo Real como Diferencial

Integrar feeds de inteligência (NVD, OTX, ATT&CK TAXII) é o diferencial mais impactante para qualidade de análise. O ArchGuard-AI deve considerar:
- Integração com NVD API para enriquecimento de vulnerabilidades
- AlienVault OTX API para contexto de ameaças ativas
- MITRE ATT&CK TAXII para TTPs atualizados

### 9.2 Foco na Usabilidade (Democratização)

A validação do AegisShield confirma que a **simplicidade de uso é tão importante quanto a precisão técnica**. Para o ArchGuard-AI:
- Interface conversacional acessível a desenvolvedores sem expertise em segurança
- Descrições de ameaças em linguagem clara, não jargão de segurança
- Mitigações acionáveis específicas ao stack tecnológico identificado

### 9.3 Validação Estatística do Output

O paper demonstra como validar o output de um sistema de threat modeling por IA:
- Comparar contra ground truth de especialistas (kappa de concordância)
- Medir alinhamento semântico (ROUGE-L, BLEURT)
- Avaliar mapeamento ATT&CK via precision@k
- Estudar redução de complexidade percebida (user study)

---

## Referências

- **Paper original:** https://arxiv.org/abs/2509.10482
- NVD (NIST National Vulnerability Database): https://nvd.nist.gov/
- AlienVault OTX: https://otx.alienvault.com/
- MITRE ATT&CK TAXII: https://attack.mitre.org/resources/attack-data-and-tools/
- MITRE ATT&CK Navigator: https://mitre-attack.github.io/attack-navigator/
