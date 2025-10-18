# **📊 Chatbot RAG para Análise de Documentos (PDF)**

## **📌 Visão Geral do Projeto**

Este projeto implementa um sistema de Chatbot de **Geração Aumentada por Recuperação (Retrieval-Augmented Generation \- RAG)** que utiliza a IA Generativa (via Gemini API) para responder a perguntas baseadas *exclusivamente* no conteúdo de documentos carregados (como PDFs de artigos científicos para um TCC).

O principal objetivo é criar uma ferramenta de busca inteligente que transcende a simples pesquisa por palavras-chave, permitindo ao usuário fazer perguntas contextuais e receber respostas fundamentadas no conhecimento proprietário dos seus documentos.

## **🛠️ Stack Tecnológico (Conceitual)**

| Componente | Ferramenta Sugerida | Função no Projeto |
| :---- | :---- | :---- |
| **Linguagem** | Python | Ambiente de desenvolvimento principal. |
| **LLM (AI)** | Google Gemini API (gemini-2.5-flash) | Geração de respostas coerentes e Embeddings. |
| **Processamento de PDF** | pypdf ou PyPDF2 | Leitura e extração de texto dos documentos. |
| **Vector Store (DB Vetorial)** | ChromaDB (local) ou Pinecone/Weaviate (nuvem) | Armazenamento e busca eficiente de vetores (embeddings). |
| **Framework RAG** | LangChain ou LlamaIndex | Orquestração das etapas de processamento e busca. |

## **⚙️ Processo de Implementação: O Fluxo RAG**

O sistema RAG é dividido em duas fases principais: **Indexação** (preparação dos dados) e **Consulta** (geração da resposta).

### **Fase 1: Indexação (Preparação do Conhecimento)**

1. **Carga do Documento:** Os PDFs (ou outros documentos na pasta inputs/) são lidos e o texto é extraído.  
2. **Chunking (Divisão em Partes):** O texto longo é dividido em fragmentos menores e gerenciáveis (chunks), geralmente com 500 a 1000 tokens e uma sobreposição (overlap) para manter o contexto entre os chunks.  
3. **Criação de Embeddings:** Cada chunk de texto é enviado ao modelo do Gemini para ser convertido em um vetor numérico de alta dimensão (embedding).  
   * *Exemplo:* chunk\_i $\\rightarrow$ embedding\_i (um array de centenas de números).  
4. **Indexação Vetorial:** Os embeddings são armazenados no Banco de Dados Vetorial, juntamente com o texto original do chunk correspondente. Este banco de dados está pronto para a busca semântica.

### **Fase 2: Consulta (Busca e Geração)**

1. **Pergunta do Usuário:** O usuário envia uma pergunta ("Qual é a correlação entre microsserviços e Scrum?").  
2. **Embeddings da Pergunta:** A pergunta do usuário é convertida no seu próprio embedding vetorial.  
3. **Busca Vetorial (Retrieval):** O DB Vetorial compara o embedding da pergunta com todos os embeddings armazenados (dos chunks) para encontrar os $k$ (e.g., 3 a 5\) chunks de texto mais semanticamente similares.  
4. **Aumento do Prompt (Augmentation):** É criado um novo prompt para o LLM do Gemini contendo:  
   * Uma **Instrução de Sistema** (Ex: "Você é um assistente de TCC. Responda a pergunta do usuário apenas com base nos documentos fornecidos.").  
   * O **Contexto Recuperado** (Os $k$ chunks de texto mais relevantes).  
   * A **Pergunta Original do Usuário**.  
5. **Geração da Resposta (Generation):** O Gemini processa este prompt aumentado e gera uma resposta coerente, *fundamentada unicamente no contexto fornecido pelos documentos*.

## **💡 Insights e Possibilidades de Melhoria**

### **Insights Principais**

1. **Personalização do Conhecimento:** A abordagem RAG supera as limitações de conhecimento do modelo base. O Chatbot não depende do seu treinamento geral, mas sim da informação específica nos seus PDFs, tornando-o um especialista no seu domínio (por exemplo, na sua bibliografia do TCC).  
2. **Mitigação de Alucinações:** Ao forçar o LLM a responder "apenas com base nos documentos fornecidos" (usando o prompt aumentado), reduzimos drasticamente a chance de ele "alucinar" ou inventar fatos.  
3. **Rastreabilidade (Grounding):** É possível e altamente recomendado que, junto com a resposta, o sistema retorne as fontes (os próprios chunks ou o nome do PDF de onde a informação foi retirada). Isso confere credibilidade acadêmica ao resultado.  
4. **Eficiência para Grandes Volumes:** Para um TCC com dezenas ou centenas de artigos, o RAG é a única forma eficiente de correlacionar ideias e extrair informações específicas sem ter que reler manualmente todos os documentos.

### **Possibilidades de Expansão**

* **Multimodalidade:** Utilizar modelos mais avançados do Gemini para processar gráficos, tabelas e imagens presentes nos PDFs, não apenas o texto.  
* **Sumarização Dinâmica:** Adicionar uma funcionalidade para pedir ao chatbot que gere um resumo conciso de todos os artigos recuperados em um tópico específico.  
* **Análise de Sentimento/Tonalidade:** Avaliar a tonalidade ou o consenso de diferentes autores sobre um determinado tema nos documentos, o que é útil para a seção de Revisão Bibliográfica.  
* **Chat Persistente:** Implementar um histórico de conversas que o LLM possa usar como contexto adicional para as perguntas seguintes.
