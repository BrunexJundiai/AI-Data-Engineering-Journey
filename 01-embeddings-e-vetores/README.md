# 🌍 Hub de Inteligência Estratégica ESG: RAG Multimodal e Comparativo Executivo (V4.0)

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=FastAPI&logoColor=white)](https://fastapi.tiangolo.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=Streamlit&logoColor=white)](https://streamlit.io/)
[![Qdrant](https://img.shields.io/badge/Qdrant-1F1F1F?style=for-the-badge&logo=Qdrant&logoColor=EF4D7B)](https://qdrant.tech/)
[![Groq](https://img.shields.io/badge/Groq_Cloud-F56522?style=for-the-badge&logo=groq&logoColor=white)](https://groq.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI_Vision-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com/)

## 📝 About (Sobre o Projeto)
O **Hub de Inteligência ESG** é uma arquitetura de dados *Enterprise* desenhada para realizar a **Auditoria Analítica Multimodal** de relatórios de sustentabilidade. O sistema ingere, cruza e audita dados não estruturados (PDFs regulatórios), estruturados (Planilhas Excel) e visuais (Gráficos e Tabelas Rasterizadas) das maiores instituições financeiras do Brasil (**Itaú, Santander, Bradesco e Banco do Brasil**).

O grande salto desta *Release* é a implementação de um Backend Desacoplado (FastAPI), Busca Híbrida (Semântica + Lexical) e Roteamento Multi-LLM, permitindo que o sistema atue como um consultor executivo automatizado com tolerância zero à alucinação de dados financeiros.

---

## 🚀 Release Notes (V4.0 - Enterprise Architecture)
A evolução da Fase 3 para a V4.0 consolidou a plataforma com padrões de mercado de alto nível:
* **Desacoplamento de Arquitetura:** Separação estrita entre o Frontend (Streamlit) e o Backend de IA (FastAPI).
* **Ingestão Omnichannel Assíncrona:** Capacidade de processar PDFs densos e planilhas de indicadores corporativos simultaneamente.
* **Inteligência Multimodal (Roteamento):** Uso do `gpt-4o-mini` exclusivo para ler e descrever fluxogramas e tabelas embutidas como imagens nos PDFs.
* **Persistência de Estado (State Persistence):** Cache em JSON para manter o histórico do Data Lake visualmente ativo no Frontend após recarregamentos (F5).
* **Tratamento de Rate Limits:** *Exponential backoff* robusto para evitar quedas no pipeline durante bloqueios de API (Erros 429).

---

## 🛠️ Tecnologias Utilizadas

### Orquestração & Backend
* **FastAPI / Uvicorn:** Servidor assíncrono para endpoints de IA.
* **LangChain (LCEL):** Orquestração do pipeline RAG.

### Cérebro Multi-LLM (Roteamento de Modelos)
* **Groq (`llama-3.3-70b-versatile`):** Motor textual ultra-rápido para síntese do relatório e cruzamento de dados (Temperature 0.0).
* **OpenAI (`gpt-4o-mini`):** Especialista em Visão Computacional para leitura de gráficos financeiros complexos.

### Banco de Dados Vetorial & Embeddings
* **Qdrant (Local):** Motor com Quantização Int8 e suporte a múltiplos vetores por ponto (*Payload Filtering*).
* **Busca Densa (Semântica):** `intfloat/multilingual-e5-small` (Processado em CPU ou GPU/CUDA).
* **Busca Esparsa (Lexical):** Algoritmo `BM25` via biblioteca `fastembed` (Processado em CPU/ONNX).
* **Reciprocal Rank Fusion (RRF):** Algoritmo de fusão matemática para balancear os *Match Scores* textuais e lexicais.

### Ingestão Estrutural (ETL)
* **PyMuPDF (`fitz`):** Extração de texto e recorte de imagens de PDFs.
* **Pandas:** Serialização rica e higienização de planilhas `.xlsx`.

---

## ⚙️ Decisões de Arquitetura (ADR) & Solução de Desafios

### 1. Busca Híbrida vs. "Colisão de Siglas"
Identificamos que vetores puramente semânticos confundiam siglas como **SAC** (*Serviço de Atendimento*) com **SAC** (*Socioambiental e Climático*). A implementação da busca lexical **BM25** garante que termos técnicos regulatórios (ex: ICAAP, PRSAC) sejam recuperados com prioridade cirúrgica.

### 2. Otimização de Custos em FinOps (Visão Computacional)
*Decisão Arquitetural:* A Visão Computacional (OpenAI) foi homologada com sucesso, extraindo cifras exatas de gráficos rasterizados. Contudo, constatamos que a maioria dos relatórios ESG utiliza imagens apenas como elementos decorativos (fotos institucionais), enquanto os gráficos reais são desenhados em formato vetorial. 
**Ação:** O pipeline foi programado para rejeitar "lixo visual". A VLM é acionada estritamente quando gráficos rasterizados reais são detectados, reduzindo os custos de API em mais de 90% sem perda de precisão analítica.

### 3. Cross-Format Reasoning (PDF x Excel)
O motor de Busca Híbrida RRF cruza narrativas textuais dos PDFs com linhas exatas de planilhas Excel (lidando com colunas `Unnamed`). A IA responde comparando a "intenção" escrita no relatório com o "número real" reportado na planilha do mesmo ano.

---

## 📸 Fluxo de Uso e Evidências Visuais

### 1. Data Lake e Pipeline de Ingestão (Frontend)
Upload massivo de documentos (PDFs e Excels) com etiquetagem de metadados dinâmicos e persistência visual do acervo.
![Tela Inicial e Ingestão](img/tela_inicial1.png)
<p align="center">
  <img src="img/menu_esquerda.png" width="30%">
  <img src="img/menu_esquerda2.png" width="30%">
  <img src="img/menu_esquerda3.png" width="30%">
</p>

### 2. Auditoria Cruzada (Stress Test: Itaú vs Bradesco)
A IA cruza dados de diferentes relatórios e anos, aplicando filtros rigorosos. Resposta direta, sem alucinação, demonstrando falta de dados quando necessário e extraindo métricas exatas de diversidade e governança.
![Exemplo Comparativo 1](img/resposta1.png)
![Exemplo Comparativo 2](img/resposta1_2.png)
![Exemplo Comparativo 3](img/resposta1_3.png)

### 3. Governança e Rastreabilidade com Zero Alucinação
Identificação clara de hierarquias complexas (Comitês e Conselhos de Administração) suportada pelo *Match Score* transparente da Busca Híbrida.
![Exemplo Governança](img/resposta3.png)
![Rastreabilidade Governança](img/resposta3_2.png)

---

## 🚀 Como Executar Localmente

### 1. Pré-Requisitos
* Python 3.10+
* Conta ativa na Groq Cloud e OpenAI (para as API Keys).

### 2. Instalação
Clone o repositório e instale as dependências:
```bash
git clone [https://github.com/BrunexJundiai/AI-Data-Engineering-Journey.git](https://github.com/BrunexJundiai/AI-Data-Engineering-Journey.git)
cd projeto_esg_fase3_v4
pip install -r requirements.txt