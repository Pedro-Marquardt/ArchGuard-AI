# Arquitetura e Implementação do Fluxo Base: Diagrama ➔ Mermaid DFD ➔ Relatório STRIDE

Este guia detalha tecnicamente como construir o fluxo lógico do MVP inspirado no funcionamento do STRIDE GPT, unindo o modelo de visão computacional supervisionado (**YOLO**) ao poder de estruturação e análise de uma **LLM** (via API) dentro de uma interface **Streamlit**.

---

## 🛠️ Engenharia do Fluxo Passo a Passo

O pipeline do sistema é dividido em 4 etapas sequenciais. Abaixo está o detalhamento técnico e a lógica de código para implementar cada uma delas.

### Passo 1: Upload e Inferência Visual (YOLO + Fallback Vision AI)
O usuário faz o upload do diagrama de arquitetura (AWS, Azure ou GCP) na interface Streamlit. O arquivo é processado pelo modelo YOLO. Se a confiança média das detecções ficar abaixo de um threshold configurável pelo usuário, o sistema automaticamente aciona um fallback via GPT-4o Vision para garantir cobertura.

```python
import streamlit as st
import cv2
import numpy as np
import base64
import json
from ultralytics import YOLO
from openai import OpenAI

# 1. Inicializa o modelo YOLO e cliente OpenAI
@st.cache_resource
def load_yolo_model():
    return YOLO("pesos_treinados_yolo.pt")

model = load_yolo_model()
client = OpenAI(api_key=st.secrets["OPENAI_API_KEY"])

STRIDE_CLASSES = ['actor', 'gateway', 'compute', 'database', 'storage', 'message_bus']

st.title("🛡️ StrideVision AI - MVP")
uploaded_file = st.file_uploader("Suba o diagrama de arquitetura (AWS / Azure / GCP)", type=["png", "jpg", "jpeg"])

# Threshold configurável pelo usuário na sidebar
confidence_threshold = st.sidebar.slider(
    "Confiança mínima YOLO (fallback abaixo disso)",
    min_value=0.10, max_value=0.95, value=0.40, step=0.05
)


def fallback_vision_analysis(image_bytes: bytes) -> list:
    """Fallback: GPT-4o Vision detecta componentes quando YOLO tem baixa confiança."""
    b64_image = base64.b64encode(image_bytes).decode("utf-8")

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": f"""Analise este diagrama de arquitetura cloud/software.
Identifique TODOS os componentes visíveis e classifique cada um em uma destas categorias: {STRIDE_CLASSES}

Regras de classificação:
- actor: usuários, clients, identidade (IAM, Cognito, AD)
- gateway: load balancers, API gateways, CDN, VPC, firewalls, DNS
- compute: VMs, containers, Lambda, App Service, Cloud Run
- database: RDS, DynamoDB, SQL, CosmosDB, BigQuery
- storage: S3, EBS, Blob Storage, discos, backup
- message_bus: SQS, SNS, EventBridge, Service Bus, Pub/Sub

Retorne APENAS uma lista JSON. Exemplo: ["compute", "database", "gateway"]
Sem explicações, sem markdown."""
                },
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/png;base64,{b64_image}"}
                }
            ]
        }],
        temperature=0.1,
        max_tokens=300
    )

    try:
        return json.loads(response.choices[0].message.content.strip())
    except json.JSONDecodeError:
        content = response.choices[0].message.content
        return [c.strip().strip('"') for c in content.strip("[]").split(",")]


if uploaded_file is not None:
    file_bytes = np.asarray(bytearray(uploaded_file.read()), dtype=np.uint8)
    opencv_image = cv2.imdecode(file_bytes, 1)

    # Inferência YOLO
    results = model(opencv_image)

    # Extrai classes + confianças
    detected_classes = []
    confidences = []
    names = model.names
    for r in results:
        for c, conf in zip(r.boxes.cls, r.boxes.conf):
            detected_classes.append(names[int(c)])
            confidences.append(float(conf))

    avg_confidence = sum(confidences) / len(confidences) if confidences else 0.0

    # Decisão: YOLO confiável ou fallback Vision AI
    if avg_confidence >= confidence_threshold and len(detected_classes) > 0:
        unique_components = list(set(detected_classes))
        st.success(f"✅ YOLO (confiança média: {avg_confidence:.2f}): {unique_components}")
    else:
        st.warning(
            f"⚠️ YOLO com confiança baixa ({avg_confidence:.2f} < {confidence_threshold}). "
            "Acionando fallback GPT-4o Vision..."
        )
        uploaded_file.seek(0)
        raw_bytes = uploaded_file.read()
        detected_classes = fallback_vision_analysis(raw_bytes)
        unique_components = list(set(detected_classes))
        st.success(f"🤖 Vision AI (fallback): {unique_components}")
```

### Passo 2: Tradução e Geração do Código Mermaid.js
Com a lista de componentes detectados pelo YOLO em mãos, fazemos a primeira chamada à API da LLM. O objetivo aqui é puramente estrutural: pedir para a LLM entender como esses componentes se conectam logicamente em uma arquitetura padrão e cuspir uma string válida de Mermaid.js.

```python
from openai import OpenAI

# Inicializa o cliente da LLM (Ex: OpenAI, Groq, local via LM Studio)
client = OpenAI(api_key=st.secrets["OPENAI_API_KEY"])

def generate_mermaid_code(components_list):
    prompt = f"""
    Você é um Arquiteto de Software Especialista. Recebi uma lista de componentes de infraestrutura detectados por um modelo de visão computacional: {components_list}.
    
    Gere o código de um fluxograma usando a sintaxe exata do Mermaid.js (graph LR) que represente um fluxo lógico e padrão de dados entre esses elementos.
    
    Regras estritas:
    1. Retorne APENAS o código Mermaid estruturado, sem explicações, sem markdown de bloco de código (```mermaid). Comece direto com 'graph LR'.
    2. Use labels genéricos para os nós baseados na lista fornecida.
    """
    
    response = client.chat.completions.create(
        model="gpt-4o-mini", # Ou qualquer modelo de sua escolha
        messages=[{"role": "user", "content": prompt}],
        temperature=0.2
    )
    return response.choices[0].message.content.strip()

# Executa a geração do fluxo se houver componentes detectados
if uploaded_file is not None and unique_components:
    mermaid_code = generate_mermaid_code(unique_components)
    st.subheader("Data Flow Diagram (DFD) Gerado")
    
    # Exibe o código gerado para auditoria
    st.code(mermaid_code, language="text")
```

Nota de Implementação: Para exibir o gráfico visualizado diretamente no Streamlit, utilize a biblioteca da comunidade executando: pip install streamlit-mermaid. No código, basta renderizar usando st_mermaid(mermaid_code).

### Passo 3: Engine STRIDE (A Análise de Segurança)
Agora passamos o código do diagrama Mermaid gerado (que descreve o fluxo exato e as conexões da arquitetura) para a LLM principal juntamente com o framework de regras de segurança. A IA utilizará o DFD para analisar as fronteiras de confiança (trust boundaries).


```python
def generate_stride_report(mermaid_diagram):
    prompt = f"""
    Você é um Engenheiro de Segurança de Sistemas Specialist na metodologia STRIDE e OWASP.
    Analise o seguinte Diagrama de Fluxo de Dados (DFD) estruturado em Mermaid:
    
    {mermaid_diagram}
    
    Gere um relatório completo de Modelagem de Ameaças focado no framework STRIDE. 
    Para cada conexão ou componente crítico do fluxo:
    1. Identifique a categoria STRIDE aplicável (Spoofing, Tampering, etc.).
    2. Explique a vulnerabilidade e o cenário de risco.
    3. Sugira contramedidas agnósticas (mencionando serviços equivalentes para AWS e Azure, ex: AWS WAF / Azure WAF).
    
    Formate o resultado final estritamente em uma tabela Markdown bem organizada.
    """
    
    response = client.chat.completions.create(
        model="gpt-4o", # Modelo mais robusto para raciocínio analítico
        messages=[{"role": "user", "content": prompt}],
        temperature=0.3
    )
    return response.choices[0].message.content

if uploaded_file is not None and 'mermaid_code' in locals():
    with st.spinner("Analisando vulnerabilidades com Engine STRIDE..."):
        stride_report = generate_stride_report(mermaid_code)
        st.subheader("Relatório de Modelagem de Ameaças (STRIDE)")
        st.markdown(stride_report)
```

### Passo 4: Geração Automática de Testes Gherkin (BDD)

Para amarrar o projeto com engenharia de software de ponta, adicionamos uma última função na pipeline que lê a string do relatório STRIDE e gera cenários de teste automatizados em formato Gherkin (Cucumber).


```python
def generate_gherkin_tests(stride_report):
    prompt = f"""
    Com base no relatório de ameaças STRIDE abaixo, gere cenários de teste Cucumber/Gherkin (BDD) para validar que as contramedidas e mitigações sugeridas estão funcionando corretamente no sistema.
    
    Relatório:
    {stride_report}
    
    Forneça pelo menos 3 cenários críticos estruturados em: Cenário, Dado que, Quando, Entao. Retorne em formato Markdown de bloco de código.
    """
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.5
    )
    return response.choices[0].message.content

if uploaded_file is not None and 'stride_report' in locals():
    gherkin_tests = generate_gherkin_tests(stride_report)
    st.subheader("Massa de Testes Automatizados (BDD/Gherkin)")
    st.markdown(gherkin_tests)

```

### Passo 5: Mapeamento de Custo Estimado de Mitigação:

#### Abordagem 1: Implementação via RAG (Retrieval-Augmented Generation)

Nesta abordagem, você faz o download prévio (scraping manual ou via script uma única vez) das tabelas de preços principais de segurança da AWS e Azure (ex: preços do AWS WAF, Azure Front Door, KMS, Key Vault) e armazena em um banco de dados vetorial local (como o ChromaDB).

```
graph LR
    A[Componente Detectado] --> B[ChromaDB Local]
    B -->|Busca por Similaridade Vetorial| C[Contexto de Preços Encontrado]
    C --> D[LLM Prompt]
    D --> E[Relatório STRIDE Precificado]
```    


#### Abordagem 2: Implementação via Crawler Agent / MCP Agent

Nesta abordagem, o seu sistema não guarda dados de preço. Quando o YOLO detecta um componente, um Agente Inteligente (usando o protocolo MCP - Model Context Protocol ou bibliotecas como CrewAI / LangGraph) recebe a tarefa de abrir o navegador de forma autônoma, pesquisar no Google, ler o site oficial da AWS/Azure, extrair o preço corrente e retornar para a LLM principal.


```
graph LR
    A[Componente Detectado] --> B[MCP Agent com Ferramenta de Busca]
    B -->|Acessa Web em Tempo Real| C[Google / Sites Oficiais de Cloud]
    C -->|Web Scraping Dinâmico| B
    B -->|Dados Brutos Extraídos| D[LLM Principal]
```

Como Implementar o Agente com Ferramenta de Busca:
Você pode criar uma ferramenta (Tool) utilizando a API do Tavily ou Serper para que o agente faça buscas na web sem precisar escrever seletores CSS complexos na mão.

```python

from langchain_community.tools.tavily_search import TavilySearchResults
from openai import OpenAI

client = OpenAI()
web_search_tool = TavilySearchResults(max_results=2)

def agent_fetch_live_pricing(component, cloud):
    query = f"site:aws.amazon.com/pt/ ou site:azure.microsoft.com/pt-br/ preço atualizado serviço segurança {component} {cloud} pricing"
    
    # O agente executa a busca na web em tempo real
    search_results = web_search_tool.invoke({"query": query})
    
    # Passa o retorno bruto da web para uma LLM rápida resumir o dado financeiro
    summary_prompt = f"Com base nos resultados de busca abaixo, extraia estritamente o modelo de preço e valores do serviço de segurança para {component} na nuvem {cloud}:\n\n{search_results}"
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": summary_prompt}],
        temperature=0.0
    )
    return response.choices[0].message.content
```


### 💾 Recursos de Exportação (Entregáveis do Hackathon)

No final do script do Streamlit, adicione botões para que o usuário possa fazer o download dos artefatos gerados, garantindo pontos preciosos com a banca pela usabilidade do MVP:

```python
if uploaded_file is not None and 'stride_report' in locals():
    st.divider()
    st.subheader("💾 Exportar Artefatos")
    
    # Download do Relatório STRIDE
    st.download_button(
        label="Baixar Relatório STRIDE (.md)",
        data=stride_report,
        file_name="relatorio_stride_threat_model.md",
        mime="text/markdown"
    )
    
    # Download dos Testes Gherkin
    st.download_button(
        label="Baixar Suíte de Testes BDD (.feature)",
        data=gherkin_tests,
        file_name="seguranca_features.feature",
        mime="text/plain"
    )
```