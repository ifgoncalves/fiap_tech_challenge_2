# FinOps

## 1. Objetivo

As decisões de FinOps do projeto buscaram reduzir processamento desnecessário, evitar infraestrutura adicional e manter a solução adequada ao volume e ao objetivo acadêmico.

Como o processamento foi realizado em Databricks Serverless, o foco ficou no uso eficiente de compute, redução de movimentação de dados e reaproveitamento das informações já tratadas.

---

## 2. Uso de Databricks Serverless

O ambiente Serverless evita a necessidade de manter clusters dedicados em execução continuamente.

Para o cenário deste projeto, em que as execuções são pontuais, essa abordagem é mais adequada do que manter infraestrutura permanente.

---

## 3. Armazenamento e processamento

As tabelas das camadas Bronze, Silver e Gold são persistidas em Delta Lake.

O formato colunar utilizado pelo Delta é adequado para consultas analíticas e evita conversões desnecessárias entre formatos ao longo do pipeline.

A leitura dos dados também é feita diretamente do BigQuery para o Spark, sem criação de arquivos intermediários para transporte.

---

## 4. Otimização das transformações

Tabelas pequenas, como o dicionário e as metas, são utilizadas em Broadcast Join.

Essa estratégia reduz operações de shuffle e movimentação de dados entre os executores.

Além disso, o fluxo Streaming reutiliza a tabela `silver.metas` produzida pelo pipeline Batch, evitando recalcular informações já tratadas.

---

## 5. Processamento Streaming

O Spark Structured Streaming utiliza checkpoints para registrar os eventos já processados.

Com isso, novas execuções processam apenas os registros ainda não consumidos.

Foi utilizado:

    trigger(availableNow=True)

Essa escolha ocorreu devido às restrições do ambiente Databricks Serverless e também evita manter uma consulta Streaming ativa sem necessidade.

Não foi utilizada infraestrutura Kafka, pois o Structured Streaming com Delta e checkpoints foi suficiente para a simulação de eventos proposta no projeto.

---

## 6. Custos e possíveis melhorias

Em uma implantação produtiva, os principais custos estariam relacionados a:

- processamento no Databricks;
- volume consultado no BigQuery;
- armazenamento;
- frequência de execução do Streaming.

As principais decisões adotadas para reduzir consumo foram:

- uso de Serverless;
- armazenamento em Delta;
- processamento incremental com checkpoints;
- Broadcast Join para tabelas pequenas;
- reaproveitamento das tabelas Batch;
- ausência de infraestrutura Kafka dedicada;
- leitura direta do BigQuery.

Em produção, também poderiam ser avaliadas estratégias como particionamento, políticas de retenção e otimizações específicas das tabelas Delta, de acordo com o crescimento do volume.