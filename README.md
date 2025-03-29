# Fourth Coffee - Análise de Avaliações com Azure AI Search

## Visão Geral
Este projeto implementa uma solução de mineração de conhecimento para extrair insights das avaliações de clientes da Fourth Coffee, uma rede nacional de cafés. Utilizando Azure AI Search, Cognitive Services e Azure Blob Storage, criamos um sistema que processa avaliações textuais e imagens, aplica enriquecimento de IA (como detecção de sentimentos, extração de frases-chave e reconhecimento de imagens) e armazena os resultados em um Knowledge Store para análise posterior.

## 📌 Passo a Passo da Implementação

### 1. Criação dos Recursos no Azure
Foram provisionados os seguintes recursos:

- **Azure AI Search** (`fourth-coffee-search`)
  - Função: Indexar e consultar os dados das avaliações.
  - Configuração: Camada Básica, região East US 2.

- **Serviços de IA do Azure** (`fourth-coffee-ai`)
  - Função: Aplicar habilidades cognitivas (NLP, OCR, detecção de sentimentos).
  - Configuração: Tipo Standard S0, mesma região do Azure AI Search.

- **Conta de Armazenamento** (`fourthcoffeestorage`)
  - Função: Armazenar avaliações brutas e resultados enriquecidos.
  - Configuração: Contêiner público `coffee-reviews` para upload dos arquivos e `knowledge-store` (privado) para os dados processados.

### 2. Upload das Avaliações
Os arquivos `review-1.docx` a `review-9.docx` foram carregados no Blob Storage (contêiner `coffee-reviews`).

As avaliações incluíam:
- Texto descritivo sobre a experiência no café.
- Imagens do ambiente (em algumas avaliações).

### 3. Configuração do Azure AI Search

#### Pipeline de Indexação
- **Fonte de Dados** (`coffee-customer-data`):
  - Conectada ao contêiner `coffee-reviews`.
  - Extração de conteúdo + metadados (OCR habilitado para imagens).

- **Skillset de IA** (`coffee-skillset`):
  - Habilidades aplicadas:

    | Habilidade               | Saída          | Exemplo de Uso                              |
    |--------------------------|----------------|---------------------------------------------|
    | Extração de localizações | locations      | Identificar cidades (Chicago, Seattle, LA)  |
    | Detecção de sentimentos  | sentiment      | Classificar avaliações como "positivo/negativo" |
    | Extração de frases-chave | keyphrases     | "Wi-Fi excelente", "ambiente aconchegante"  |
    | Reconhecimento de imagens | imageTags, imageCaption | Tags: "café", "mesa"; Legendas: "Pessoas trabalhando em um café" |

- **Knowledge Store** (`knowledge-store`):
  - Armazena dados enriquecidos em tabelas e blobs para análise posterior.

- **Índice** (`coffee-index`):
  - Campos filtráveis: content, locations, sentiment, keyphrases, etc.
  - Chave: metadata_storage_path.

- **Indexador** (`coffee-indexer`):
  - Executado uma vez para processar os dados.

## 4. Consultas e Insights
Utilizamos o Search Explorer para extrair informações:

### 📊 Principais Descobertas
- **Distribuição Geográfica**:
  ```json
  { "search": "locations:'Seattle'", "count": true }  // 4 avaliações
  { "search": "locations:'Chicago'", "count": true }  // 3 avaliações
  { "search": "locations:'Los Angeles'", "count": true }  // 2 avaliações
