# Tech Challenge - Pipeline Híbrido para Análise da Alfabetização no Brasil

Projeto desenvolvido para o Tech Challenge da pós-graduação, com foco na construção de uma pipeline híbrida de dados utilizando processamento Batch e Streaming.

A solução trabalha com dados públicos de alfabetização no Brasil, aplicando arquitetura Medalhão nas camadas Bronze, Silver e Gold.

---

## Objetivo

O objetivo do projeto é organizar e transformar os dados de alfabetização para permitir análises sobre:

- desempenho por território e rede de ensino;
- acompanhamento das metas de alfabetização;
- evolução dos indicadores;
- recebimento de novas medições por Streaming;
- qualidade e consistência dos dados.

---

## Arquitetura

O fluxo histórico é processado em Batch:

    Google BigQuery
          |
          v
        Bronze
          |
          v
        Silver
          |
          v
         Gold

Também foi criado um fluxo Streaming para simular a chegada de novas medições:

    Simulador
        |
        v
    Bronze Streaming
        |
        v
    Silver Streaming
        |
        v
    Gold Streaming
        ^
        |
    silver.metas
       Batch

A Gold Streaming utiliza as metas já tratadas pelo pipeline Batch para comparar as novas medições com os valores esperados.

Os eventos Streaming utilizados no projeto são sintéticos e não representam resultados oficiais do INEP.

Mais detalhes estão disponíveis em [docs/arquitetura.md](docs/arquitetura.md).

---

## Principais tabelas

### Silver

- `silver.alunos`
- `silver.resultados_territoriais`
- `silver.distribuicao_niveis`
- `silver.metas`

### Gold

- `gold.acompanhamento_metas`
- `gold.indicadores_rede`
- `gold.resumo_atingimento`
- `gold.acompanhamento_metas_streaming`

---

## Estrutura do repositório

    notebooks/
    ├── batch/
    │   ├── 01_ingestao_bronze
    │   ├── 02_transformacao_silver
    │   └── 03_construcao_gold
    │
    ├── streaming/
    │   ├── 01_simulador_eventos
    │   └── 02_pipeline_streaming
    │
    └── quality/
        └── 01_validacao_qualidade

    docs/
    ├── arquitetura.md
    └── finops.md

---

## Data Quality

Foi criada uma etapa específica para validar as principais tabelas do pipeline.

As regras verificam:

- campos obrigatórios;
- unicidade de chaves;
- intervalos válidos;
- consistência entre indicadores;
- integridade referencial;
- preservação dos eventos entre Silver e Gold Streaming.

Os resultados são armazenados em `quality.resultados_validacao` e classificados como `PASS`, `WARN` ou `FAIL`.

---

## Streaming

O Streaming foi implementado com Spark Structured Streaming e checkpoints.

No ambiente Databricks Serverless foi utilizado:

    trigger(availableNow=True)

Dessa forma, cada execução processa apenas os novos eventos ainda não registrados no checkpoint.

Durante os testes, após o processamento inicial de cinco eventos, um sexto evento foi incluído e processado sem duplicar os registros anteriores.

---

## FinOps

Algumas decisões do projeto também buscaram reduzir processamento e infraestrutura desnecessária, como:

- uso de Databricks Serverless;
- armazenamento em Delta;
- leitura direta do BigQuery;
- Broadcast Join para tabelas pequenas;
- processamento incremental com checkpoints;
- reaproveitamento das metas Batch no Streaming;
- não utilização de infraestrutura Kafka dedicada.

Mais detalhes estão disponíveis em [docs/finops.md](docs/finops.md).

---

## Tecnologias

- Databricks
- Apache Spark
- PySpark
- Spark Structured Streaming
- Delta Lake
- Unity Catalog
- Google BigQuery
- Git
- GitHub

---

## Possíveis aplicações em IA

As tabelas produzidas na camada Gold podem ser utilizadas como base para análises mais avançadas, como:

- identificação de regiões com maior risco de não atingir as metas;
- análise de desigualdades territoriais;
- acompanhamento da evolução dos indicadores;
- apoio à priorização de políticas públicas.

Uma evolução do projeto poderia utilizar essas informações como entrada para modelos preditivos de desempenho e atingimento das metas.

---

## Execução

Os notebooks foram organizados na ordem do pipeline.

Para reproduzir o fluxo principal:

1. executar os notebooks Batch na ordem Bronze, Silver e Gold;
2. executar o simulador de eventos;
3. executar o pipeline Streaming;
4. executar o notebook de Data Quality.

As credenciais de acesso ao Google Cloud não são armazenadas no repositório e devem ser configuradas separadamente no ambiente Databricks.
