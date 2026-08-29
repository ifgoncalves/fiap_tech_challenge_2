# Arquitetura da Solução

## 1. Visão Geral

O projeto foi desenvolvido com o objetivo de criar uma pipeline híbrida para análise dos indicadores de alfabetização no Brasil.

A solução combina processamento Batch, utilizado para os dados históricos, com processamento Streaming, utilizado para simular a chegada de novas medições de desempenho.

A arquitetura segue o modelo Medalhão, com separação entre as camadas Bronze, Silver e Gold. Essa organização facilita o tratamento dos dados, a rastreabilidade das transformações e a disponibilização das informações para análise.

O processamento foi implementado no Databricks utilizando PySpark, Spark Structured Streaming e tabelas Delta.

---

## 2. Fonte de Dados

Os dados históricos utilizados no projeto são provenientes do conjunto `br_inep_avaliacao_alfabetizacao`, disponibilizado pela Base dos Dados no Google BigQuery.

Foram utilizadas as seguintes tabelas:

- `alunos`
- `municipio`
- `uf`
- `meta_alfabetizacao_municipio`
- `meta_alfabetizacao_uf`
- `meta_alfabetizacao_brasil`
- `dicionario`

A ingestão é realizada diretamente do BigQuery para o Spark, sem necessidade de geração intermediária de arquivos CSV.

Essa abordagem reduz etapas adicionais de armazenamento e mantém o processo de ingestão mais simples.

---

## 3. Arquitetura Batch

O fluxo Batch é responsável pelo processamento dos dados históricos.

A ingestão começa com a leitura das tabelas do BigQuery e gravação na camada Bronze. A partir dessa camada são executados os tratamentos necessários para construção da Silver e, posteriormente, das tabelas analíticas da Gold.

O fluxo pode ser resumido da seguinte forma:

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

### 3.1 Bronze

A camada Bronze mantém os dados próximos ao formato original da fonte.

Nesta etapa são adicionados apenas metadados técnicos utilizados para rastreabilidade do processo, como origem dos dados e momento da ingestão.

As tabelas são persistidas em formato Delta e funcionam como ponto de entrada dos dados históricos dentro do Databricks.

As sete tabelas provenientes do BigQuery são armazenadas individualmente na camada Bronze, preservando a estrutura original da fonte.

---

### 3.2 Silver

Na Silver são aplicadas as principais regras de tratamento, padronização e organização dos dados.

Antes da definição das transformações foram realizadas análises de duplicidade e valores ausentes para entender as características das tabelas e evitar tratamentos que alterassem o significado original dos dados.

Entre os principais tratamentos realizados estão:

- análise de duplicidades;
- análise de valores ausentes;
- padronização de campos;
- enriquecimento de códigos utilizando a tabela de dicionário;
- consolidação dos resultados territoriais;
- transformação das distribuições de níveis de proficiência;
- normalização das metas de alfabetização.

A partir dessas transformações foram criadas as seguintes tabelas:

- `silver.alunos`
- `silver.resultados_territoriais`
- `silver.distribuicao_niveis`
- `silver.metas`

Valores ausentes que fazem parte da própria estrutura da base foram preservados.

Por exemplo, foram identificados registros de alunos sem informações de proficiência quando a prova não havia sido preenchida. Nesses casos, os valores não foram substituídos por zero ou outro valor artificial, pois isso poderia alterar a interpretação do dado.

A tabela `silver.metas` também reorganiza as metas disponíveis na fonte, transformando as colunas referentes aos diferentes anos em uma estrutura única baseada em `ano_meta`. Isso facilita o relacionamento das metas com os resultados observados.

---

### 3.3 Gold

A camada Gold concentra as informações já preparadas para consumo analítico.

Foram criadas três tabelas principais.

#### `gold.acompanhamento_metas`

Permite comparar a taxa de alfabetização observada com a meta definida para cada território.

Entre os indicadores calculados estão:

- taxa de alfabetização;
- meta de alfabetização;
- gap em pontos percentuais;
- percentual de atingimento da meta;
- flag de atingimento;
- status da meta.

A tabela facilita a identificação dos territórios que atingiram ou não os objetivos definidos para cada período.

#### `gold.indicadores_rede`

Consolida os principais indicadores de alfabetização por ano e rede de ensino.

Entre os indicadores calculados estão:

- total de alunos;
- alunos presentes;
- provas preenchidas;
- taxa de presença;
- taxa de preenchimento;
- taxa de preenchimento entre alunos presentes;
- alunos alfabetizados;
- taxa de alfabetização entre avaliados;
- taxa de alfabetização ponderada;
- média de proficiência.

Essa tabela fornece uma visão resumida do desempenho das diferentes redes de ensino.

#### `gold.resumo_atingimento`

Apresenta uma visão consolidada do cumprimento das metas por ano e nível geográfico.

Entre as informações disponibilizadas estão:

- total de regiões;
- regiões avaliáveis;
- regiões que atingiram a meta;
- regiões que não atingiram a meta;
- regiões sem resultado;
- regiões sem meta;
- percentual de regiões que atingiram a meta;
- gap médio;
- melhor gap;
- pior gap.

Essa tabela foi construída para facilitar análises mais agregadas sobre o cumprimento das metas de alfabetização.