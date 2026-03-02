# Fase 1: Motor RAG Local para Auditoria ESG e Risco Climático 🌍🏦

## 📝 Descrição do Módulo
Foco na construção de uma arquitetura de *Retrieval-Augmented Generation* (RAG) para extração de dados densos em relatórios regulatórios de sustentabilidade. O projeto substitui buscas simples por palavras-chave por uma compreensão semântica profunda, permitindo cruzar métricas financeiras (ex: Preço Interno de Carbono) com metas climáticas (Escopos 1, 2 e 3).

## 🛠️ Tecnologias e Ferramentas
* **Linguagem:** Python
* **LLM (Raciocínio):** `llama-3.3-70b-versatile` (via Groq API para latência ultrabaixa).
* **Embeddings (Vetorização):** `intfloat/multilingual-e5-small` (HuggingFace).
* **Banco de Dados Vetorial:** ChromaDB (Persistente local).
* **Processamento:** CUDA (NVIDIA GPU) para processamento paralelo em tensores.
* **Interface:** Streamlit.

## 🏗️ Projeto Prático / Laboratório
* **O Desafio:** Criar um Auditor de IA capaz de processar e interpretar um Relatório ESG bancário de 275 páginas localmente, extraindo métricas exatas sem alucinação, enfrentando o gargalo de uma infraestrutura local com limite de 2GB de VRAM na GPU.
* **A Solução:** Um pipeline de ingestão otimizado que fatia o documento respeitando os limites de *tokens* do modelo de embeddings, utiliza processamento em lotes (`batch_size`) na placa de vídeo e orquestra a recuperação de múltiplos contextos simultâneos para o LLM.
* **Resultado:** Redução drástica na latência de ingestão (migração CPU para GPU) e respostas com 100% de precisão na localização de valores financeiros e normativas de Risco Climático.

---

## ⚙️ Log de Engenharia e Decisões de Arquitetura

O desenvolvimento desta Fase 1 exigiu refatorações profundas para equilibrar a inteligência do modelo com as restrições de hardware.

**1. Evolução do Modelo de Embeddings e Gestão de VRAM:**
* A arquitetura iniciou com o modelo `MiniLM`, migrando para o `BAAI/bge-m3` devido à sua superioridade multilíngue e capacidade de ler textos densos.
* **Problema:** O processamento via CPU gerava uma latência impraticável. A migração para GPU (CUDA) esbarrou no limite físico de 2GB de VRAM da máquina (`CUDA Out of memory`).
* **Solução:** *Downgrade* estratégico para o modelo `intfloat/multilingual-e5-small`. Ele consome pouca VRAM, é altamente veloz e foi desenhado especificamente para tarefas de RAG.

**2. Engenharia de Chunking e Limites de Tokens:**
* Para relatórios ESG, tabelas de emissões e notas explicativas precisam ficar no mesmo bloco semântico. 
* **Ajuste Fino:** Com a adoção do modelo `e5-small`, o tamanho do bloco (`chunk_size`) precisou ser reduzido para **1000 caracteres** com `overlap` de **200**. Isso foi necessário para respeitar o limite máximo estrito de 512 *tokens* do modelo, evitando o truncamento silencioso de dados críticos pelo motor do HuggingFace.

**3. Compensação na Janela de Recuperação (Retriever):**
* Ao reduzir o tamanho do *chunk* pela metade, o LLM perderia contexto.
* **Solução:** O parâmetro de busca (`search_kwargs={"k": 15}`) foi elevado. O sistema agora recupera os 15 blocos mais relevantes, garantindo que o `Llama-3.3-70B` receba cerca de 15.000 caracteres de contexto para cruzar dados antes de formular a resposta.

**4. Otimização de Processamento em Lote:**
* Implementação de `model_kwargs={'device': 'cuda'}` e `encode_kwargs={'normalize_embeddings': True, 'batch_size': 8}`. Lotes menores (8) garantiram fluidez na GPU sem estourar a memória durante a vetorização massiva do PDF. Correção de erros de bloqueio de banco de dados (`Code: 13`) expurgando a pasta raiz do ChromaDB para realinhar as novas dimensões vetoriais.

**5. Engenharia de Prompt (Zero-Shot Constraint):**
* Alteração da persona da IA genérica para **Auditor de IA em Compliance ESG e Risco Climático**. Adição de travas condicionais forçando rigor metrológico (toneladas de CO2e, R$) e blindagem contra alucinações de dados não constantes no texto recuperado.

---

## 🚀 Próximos Passos (Roadmap do Projeto)

A aplicação continuará escalando para se tornar uma solução completa de Governança de Dados Corporativos, englobando:

* **Análise Comparativa Temporal:** Ingestão do Relatório ESG de 2025 (assim que publicado) para criar *prompts* de confrontamento e evolução de metas climáticas (2024 vs 2025).
* **Escalabilidade e Persistência Avançada:** Migração ou otimização do banco vetorial para lidar com o acúmulo de múltiplos anos de relatórios.
* **Busca Híbrida e Filtragem Avançada:** Combinação de busca semântica (vetores) com busca lexical (BM25) e metadados estruturados (ex: ano, diretoria, escopo de emissão).
* **RAG Multimodal:** Implementação de inteligência para leitura de gráficos e infográficos complexos diretamente das imagens dos PDFs.
* **Fine-Tuning:** Treinamento especializado do modelo de *embeddings* com vocabulário proprietário do setor bancário brasileiro.
* **Agentes e Deploy:** Construção de memória transacional para o agente e *deploy* da aplicação em ambiente de produção (Cloud).
