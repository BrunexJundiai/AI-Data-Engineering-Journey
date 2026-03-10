# Fase 1: Motor RAG Local para Auditoria ESG e Risco Climático 🌍🏦

## 📝 Descrição do Módulo
Foco na construção de uma arquitetura de *Retrieval-Augmented Generation* (RAG) para extração de dados densos em relatórios regulatórios de sustentabilidade. O projeto substitui buscas simples por palavras-chave por uma compreensão semântica profunda, permitindo cruzar métricas financeiras (ex: Preço Interno de Carbono) com metas climáticas (Escopos 1, 2 e 3).

## 🛠️ Stack Tecnológica (V2.0 - Arquitetura Desacoplada)
* **LLM (Motor Cognitivo):** `llama-3.3-70b-versatile` (via Groq API para inferência em milissegundos).
* **Framework Orquestrador:** LangChain com arquitetura moderna **LCEL** (LangChain Expression Language).
* **Embeddings (Vetorização local):** `intfloat/multilingual-e5-small` (HuggingFace).
* **Extração Estrutural (ETL):** `PyMuPDF` (Fitz) com ingestão em loop para **Múltiplos Documentos**.
* **Banco de Dados Vetorial:** ChromaDB.
* **Aceleração de Hardware:** CUDA (NVIDIA GPU) para processamento paralelo de tensores.
* **Interface (Front-end):** Streamlit com Gestão de Estado (Session State) para Interface de Chat interativo e renderização de metadados em JSON.
* **Paradigma de Software:** Orientação a Objetos (POO) com Separação de Preocupações (Backend/Frontend).

---

## ⚙️ Engenharia de Dados e Software: A Evolução Definitiva

Relatórios ESG do setor financeiro são "pesadelos estruturais" para IAs: possuem layouts em múltiplas colunas, tabelas quebradas entre páginas e uma infinidade de notas de rodapé. Esta versão final resolveu os gargalos reescrevendo o pipeline de ingestão e a arquitetura de software:

1. **Separação de Preocupações (POO):** O projeto migrou de um script monolítico para uma arquitetura profissional dividida em duas camadas: o Cérebro (`bbe_rag_engine.py` como classe Backend autônoma) e a Interface (`bbe_app.py` focado apenas em UI e histórico de chat).
2. **Arquitetura LCEL e "Stuff" Manual:** Abandono de cadeias opacas (Legacy `RetrievalQA`) em favor da *LangChain Expression Language*. Os blocos de contexto recuperados agora são injetados de forma transparente no prompt, permitindo cruzamento de dados de alta precisão pelo Llama 3.
3. **Extração Estrutural de Tabelas (PyMuPDF):** Substituição de leitores de PDF ingênuos por um extrator que respeita o fluxo de leitura humano. Isso impediu que a IA misturasse os números do Escopo 1 com o Escopo 2 em tabelas complexas.
4. **Sanitização e Data Enrichment (Injeção de Metadados):** Implementação de uma camada de limpeza via *Regex* antes da vetorização. O motor "carimba" a origem e o número exato da página fisicamente no início de cada bloco de texto, obrigando o LLM a fundamentar sua resposta com a fonte.
5. **Otimização Extrema de Hardware (2GB VRAM):** Para viabilizar a vetorização massiva em hardware local restrito, a carga foi transferida da CPU para a GPU (`device: cuda`), operando com lotes reduzidos (`batch_size: 8`) e janela de contexto ampliada (`k=15`) para não perder o raciocínio macro.
6. **Zero-Shot Constraints e Teste de Estresse:** Prompt de engenharia estrito para a persona de Auditor de Compliance. Aprovado em testes de estresse para limites temporais e alucinações. Se a informação não está no chunk, a IA trava e emite o alerta: *"A informação não consta nos documentos analisados"*.
7. **Ingestão Multi-Documento:** O pipeline temporário (`tempfile`) agora processa *N* relatórios simultaneamente em lote, preparando o terreno para cruzamentos temporais (ex: Relatório 2024 + Relatório 2025).

---

## 📸 Fluxo de Uso e Evidências Visuais

Abaixo, as evidências de performance e a rastreabilidade do produto em ação.

### 1. Aceleração de Infraestrutura (Gargalo de Latência)
A ingestão de dados não estruturados massivos (276 páginas) exigiu a migração do processamento matemático da CPU para o paralelismo da GPU.

* **Antes (Gargalo em CPU):** Alta carga no processador principal e latência extrema na criação dos vetores.
  ![Gargalo CPU](img/alta_latencia_modelo_CPU.png)

* **Depois (Aceleração em GPU):** Placa de vídeo assumindo a carga, processando o documento em segundos.
  ![Otimização GPU](img/baixa_latencia_modelo_GPU.png)

### 2. O Pipeline RAG V2 em Ação

* **Ingestão Inteligente:** A prova da eficácia do *Noise Cleaning*. O sistema processou centenas de páginas e gerou 641 blocos limpos, sem estourar a memória.
  ![Processamento Limpo](img/v2_app_esg_img1.png)

* **Precisão e Citação de Fontes:** A IA cruza a política de crédito com o risco climático e entrega a resposta citando rigorosamente a `[Página 127]` do documento original.
  ![Resposta com Página](img/v2_app_esg_rsp_chunk.png)

* **Resolução de Consultas Complexas (Tabelas e Cálculos):** A prova de fogo da V2. O LLM extrai as métricas de Escopo 1 e Escopo 2 de colunas distintas da tabela, realiza a soma matemática exata em toneladas de CO2 equivalente (tCO2e) e fornece a rastreabilidade. Zero alucinação em jargões financeiros densos.
  ![Consulta Complexa](img/exemplo_resposta.png)

* **Rastreabilidade (Compliance):** Abertura do JSON nativo do banco vetorial na interface, comprovando aos auditores humanos o chunk exato que a IA consumiu para gerar o *insight*.
  ![Auditoria JSON](img/v2_app_esg_chunk_detail.png)

---

## 🚀 Roadmap (Próximos Passos: Fase 2)

A infraestrutura base está consolidada. Os próximos passos focam em resolver as limitações intrínsecas da recuperação vetorial densa e na ingestão de dados estruturados complexos:

* **Busca Híbrida (BM25 + Dense Vectors):** Combinação da busca semântica atual com motores clássicos de palavras-chave. Identificamos em testes de estresse que vetores densos diluem termos rigorosos de compliance (ex: "*expressamente proibido*"). A busca híbrida garantirá a recuperação exata de políticas restritivas.
* **Agentic RAG para Dados Tabulares (CSVs ESG):** Migração do tratamento de planilhas e reportes fiscais (bases brutas de 100+ linhas) do paradigma de *Chunking/ChromaDB* para **Agentes Autônomos (Pandas/SQL Agents)**, permitindo execução de código e matemática exata sobre matrizes de risco climático e representatividade demográfica.
* **Validação Temporal Cruzada:** Ingestão simultânea do Relatório ESG 2025 para auditar automaticamente o cumprimento das metas assumidas no ano anterior.