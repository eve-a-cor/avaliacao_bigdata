**TechPay --- Projeto Big Data, BI e Machine Learning**

Projeto desenvolvido para a aplicação prática dos conceitos de Big Data,
transformação de dados, análise exploratória, Business Intelligence e
Machine Learning no contexto fictício da fintech **TechPay**. Rota sem
admin utilizada.

**Objetivo**

O projeto tem como objetivo construir um fluxo completo de dados para
apoiar a equipe de risco da TechPay na identificação e análise de
transações fraudulentas.

O projeto contempla:

-   organização e armazenamento dos dados;

-   processamento e transformação em camadas;

-   análise exploratória;

-   construção de indicadores de fraude;

-   dashboard para análise de risco;

-   aplicação de modelo de Machine Learning para classificação de
    transações.

**Estrutura do projeto**

bigdata-curso-eve-marlyo-valdemir/

│

├── README.md

├── .gitignore

│

├── dia1_fundamentos/

│ ├── lab01_hdfs/

│ │ └── comandos.sh

│ └── lab02_sqoop/

│ └── import.sh

│

├── dia2_transformacao/

│ ├── lab03_hive_tabelas/

│ │ └── create_tables.sql

│ ├── lab04_particoes/

│ │ └── particionamento.sql

│ ├── lab05_bronze/

│ │ └── bronze.sql

│ ├── lab06_silver/

│ │ └── silver.sql

│ ├── lab07_gold/

│ │ └── gold.sql

│ └── lab08_eda/

│ └── eda_queries.sql

│

├── dia3_insights_bi/

│ ├── lab09_export/

│ │ └── export.sh

│ ├── lab10_spark/

│ │ └── spark_queries.py

│ ├── lab11_dashboard/

│ │ └── dashboard.py

│ └── lab12_ml_preview/

│ └── modelo_fraude.py

│

└── avaliacao_final/

├── etapa1_arquitetura/

│ ├── diagrama.jpg

│ └── arquitetura justificativa

├── etapa2_bigdata/

│ ├── data

│ └── raw

│ └── bronze

│ └── silver

│ └── gold

│ └── techpay.duckdb

│ ├── projeto_big_data_pipeline.py

│ └── explicacao.md

├── etapa3_analise/

│ ├── predições_fraude.parquet

│ └──
.part-00000-0ebbca92-7d69-4c36-90ae-b3ede97ed890-c000.snappy.parquet.crc

│ └── .\_SUCCESS.crc

│ ├── Projeto_Big_Data_Dashboard.py

│ ├── dashboard_tecpay.html

│ └── analise_fraude_por_faixa_valor.csv

│ └── ideia_bi.md

└── etapa4_ml/

├── projeto_big_data_ml.py

└── Relatório de ML de Big Data.md

**Etapas do projeto**

**Etapa 1 --- Arquitetura**

Definição da arquitetura de dados da TechPay, contemplando as fontes de
dados e a organização do armazenamento e processamento.

Foi utilizada a **Rota B --- organização local**, com separação dos
dados nas camadas:

-   Raw;

-   Bronze;

-   Silver;

-   Gold.

**Etapa 2 --- Pipeline Big Data**

Construção do pipeline de dados, incluindo:

1.  ingestão;

2.  definição dos tipos dos dados;

3.  particionamento;

4.  tratamento de duplicidades;

5.  correção de tipos;

6.  tratamento de valores inválidos;

7.  criação de variáveis derivadas;

8.  disponibilização dos dados para análise.

**Etapa 3 --- Análise e BI**

Desenvolvimento de análises sobre as transações da TechPay e construção
de um dashboard para acompanhamento de risco e fraude.

O dashboard apresenta indicadores como:

-   total de transações;

-   total de fraudes;

-   taxa de fraude;

-   valor fraudado;

-   evolução temporal;

-   taxa de fraude por canal;

-   taxa de fraude por categoria de estabelecimento;

-   análise por faixa de valor.

**Etapa 4 --- Machine Learning**

Desenvolvimento de um modelo de classificação para identificar
transações potencialmente fraudulentas.

O processo contempla:

-   preparação dos dados;

-   seleção das variáveis;

-   transformação das variáveis categóricas;

-   divisão entre treino e teste;

-   treinamento do modelo;

-   geração das previsões;

-   avaliação do desempenho.

**Tecnologias utilizadas**

-   Python

-   SQL

-   Apache Spark / PySpark

-   Pandas

-   Plotly

-   Scikit-learn

-   Jupyter / Google Colab

-   GitHub

**Organização dos dados**

Os dados são organizados seguindo uma estrutura de camadas:

**Raw → Bronze → Silver → Gold**

Essa organização permite separar os dados brutos dos dados tratados e
preparados para análises e modelos de Machine Learning.

**Resultado**

O projeto busca demonstrar, de forma integrada, o fluxo de dados de uma
fintech, desde a ingestão e tratamento das transações até a geração de
informações para tomada de decisão e aplicação de Machine Learning para
detecção de fraude.

**Observação**

Os dados utilizados no projeto são destinados ao desenvolvimento
acadêmico e/ou são dados sintéticos, não representando operações
financeiras reais.
