
INTRODUÇÃO

Primeiramente foram instaladas as bibliotecas duckdb pyarrow e pyspark. A seguir, foi criada a estrutura do projeto em camadas: raw, bronze, silver e gold.O arquivo com os dados foi colocado na pasta raw.

Na etapa 1 de ingestão se verificou que o arquivo de dados continha 30.000 linhas (incluindo os rótulos) e 12 colunas, sendo estas:

'transaction_id' (valores inteiros);

'customer_id' (valores inteiros);

'amount' (decimal);

'transaction_type' (objeto);

'channel' (objeto);

'merchant_category' (objeto);

'timestamp' (objeto);

'status' (objeto);

'risk_score' (valores inteiros);

'segment' (objeto);

'credit_score' (valores inteiros);

'is_fraud' (booleano).

No diagnóstico inicial, foi constatado que não havia valores nulos nem duplicados. Esta verificação foi feita porque valores nulos necessitariam de uma decisão de tratamento, seja preenchimento pela média ou exclusão da linha. Por sua vez, valores duplicados precisam ser identificados para não gerar erro nas estatísticas ao contar uma única ocorrência duas vezes. Assim, as estatísticas foram:
### Tabela 1


|  | amount | risk_score | credit_score |
| --- | --- | --- | --- |
| Quantidade | 30000,000000 | 30000,000000 | 30000,000000 |
| Média | 173,594783 | 50,106509 | 638,627033 |
| Desvio Padrão | 382,578220 | 28,858136 | 135,341620 |
| Mínimo | 5,000000 | 0,000000 | 300,000000 |
| 25% | 30,987500 | 24,927500 | 545,000000 |
| 50% | 74,595000 | 50,165000 | 640,000000 |
| 75% | 179,230000 | 75,392500 | 735,000000 |
| Máximo | 19714,690000 | 100,000000 | 900,000000 |

Na etapa 2 de criação da raw, foi criado uma cópia do data frame original, a fim de se manter os dados originais intactos, caso fosse preciso recorrer aos mesmos novamente.  A cópia foi nomeada como df e nela os nomes das colunas foram padronizados com remoção dos espaços inicial e final e conversão de todas as letras em minúsculas: isso facilita a chamada do data frame, de seus rótulos e variáveis.

As colunas de identificação foram forçadas a números, assim como as colunas numéricas (valores inteiros), a fim de se evitar erros de leitura. A coluna timestamp foi forçada a datas, enquanto as colunas categóricas tiveram seus dados padronizados com remoção de espaços extras e transformação das letras em minúsculas, no intuito de alinhar as categorias para que pudessem ser contadas corretamente. Por sua vez, a coluna booleana teve seus valores padronizados em True e False, também para a contagem correta. Este dataframe foi salvo como parquet, por ser um tipo de arquivo projetado para armazenar dados de forma eficiente para análise. Seu conceito de armazenar os dados por coluna facilita o armazenamento em bancos de dados como por exemplo o SQL.

A etapa 3 consistiu no particionamento de acordo com a distribuição temporal em anos (todos foram do ano 2025) e meses (12 meses). Caso houvesse mais anos, a quebra dos eventos em meses permitiria a identificação de alguma sazonalidade. No entanto, os dados mostraram apenas o ano de 2025.

BRONZE

A etapa 4 foi feito o processo bronze, que consiste na limpeza dos dados. Nela não foram encontrados valores nulos nem duplicidade nas colunas de identificação, como dito anteriormente. Foram ainda buscados valores negativos na coluna amount, pois isso caracterizaria um erro de sistema, o que não foi encontrado. Também não foram obtidos valores inválidos para risk_score nem credit_score, haja vista que valores inválidos nestas colunas demonstrariam erro de cálculo ou de registro no sistema. A coluna timestamp também foi varrida em busca de data inválida, pois isso geraria erro de classificação por período, o que não foi achado. Portanto, ao final da etapa bronze foi obtido o mesmo número de linhas do início da etapa, sem nenhum descarte.

======================================================================

RESULTADO DA BRONZE

======================================================================

Linhas antes:       30,000

Linhas depois:      30,000

Linhas descartadas: 0

Já a distribuição nas categorias ficou:

### Tabela 2
| transaction_type | quantidade |
| --- | --- |
| compra | 16.607 |
| transferencia | 5.893 |
| saque | 4.501 |
| pagamento | 2.999 |


### Tabela 3
| channel | quantidade |
| --- | --- |
| app | 12.021 |
| web | 9.004 |
| pos | 5.968 |
| atm | 3.007 |


### Tabela 4
| merchant_category | quantidade |
| --- | --- |
| varejo | 9.047 |
| alimentacao | 5.976 |
| servicos | 4.576 |
| eletronico | 4.440 |
| viagem | 3.010 |
| saude | 2.951 |


### Tabela 5
| status | quantidade |
| --- | --- |
| approved | 26.948 |
| declined | 3.052 |

### Tabela 6
| segment | quantidade |
| --- | --- |
| premium | 16.440 |
| standard | 10.645 |
| high-risk | 2.915 |

Para finalizar a fase bronze, foi salvo o data frame em parquet particionado no timestamp entre ano e mês no diretório correspondente.

SILVER

Partiu-se então para a etapa 5, fase silver, na qual se optou por manter channel e merchant_category. Por outro lado, foram feitas classificações temporais por período do dia e fim de semana.

Foram contadas as ocorrências de acordo com faixas de valor, para se ter uma ideia do volume de eventos por faixa de valor. A partir daí, percebe-se que os menores valores têm maior frequência.

### Tabela 7
| faixa_valor | quantidade |
| --- | --- |
| Até R$ 100 | 17.691 |
| R$ 100 a R$ 500 | 10.215 |
| R$ 500 a R$ 1.000 | 1.428 |
| R$ 1.000 a R$ 5.000 | 640 |
| Acima de R$ 5.000 | 26 |

Foram ainda classificados os gastos de acordo com o período do dia constante na timestamp, mostrando que a distribuição dos pagamentos ocorre de forma muito semelhante ao longo do dia.

### Tabela 8
| periodo_dia | quantidade |
| --- | --- |
| madrugada | 7.553 |
| manhã | 7.514 |
| tarde | 7.467 |
| noite | 7.466 |

Também foram acrescentadas 2 colunas de classificação: dia_semana, indicado o dia da semana em que o gasto ocorreu e fim_de_semana, classificando o gasto como True, se ocorrido no final de semana e False, se ocorrido de segunda à sexta. Esta classificação mostra que os fins de semana mantém a média de transações da semana, sendo aproximadamente 4.285 pagamentos por dia.

### Tabela 9
| fim_de_semana | quantidade |
| --- | --- |
| False | 21.453 |
| True | 8.547 |

Assim o resultado salvo no diretório silver manteve o número de linhas, mas acrescentou número de colunas:

======================================================================

RESULTADO DA SILVER

======================================================================

Linhas na Bronze: 30,000

Linhas na Silver: 30,000

Colunas na Silver: 18

Colunas adicionais criadas:

• faixa_valor

• periodo_dia

• dia_semana

• fim_de_semana

GOLD

Na etapa 6, fase gold, foram levantadas apenas as ocorrências de fraude por canal, a fim de se verificar que canal apresenta a maior taxa de risco, maior valor fraudado e se esses dados são coerentes com o maior risck_score_medio

======================================================================

GOLD 1 — FRAUDE POR CANAL

======================================================================

Seguindo a mesma linha, foram levantadas as fraudes por categoria, contando valor, soma fraudada, taxa de fraude e risk_score_medio:

### Tabela 10
|  | channel | total_transacoes | transacoes_fraudulentas | valor_total | valor_fraudado | risk_score_medio | taxa_fraude |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | app | 12.021 | 455 | 2.007.907,06 | 63.909,91 | 49,822697 | 0,03785 |
| 1 | atm | 3.007 | 54 | 538.079,57 | 12.395,30 | 49,502441 | 0,017958 |
| 3 | web | 9.004 | 157 | 1.573.259,20 | 31.579,22 | 50,364439 | 0,017437 |
| 2 | pos | 5.968 | 85 | 1.088.597,67 | 11.483,50 | 50,593396 | 0,014243 |

======================================================================

GOLD 2 — FRAUDE POR CATEGORIA

======================================================================

Foram contadas ainda as fraudes por segmento e suas taxas de fraude, demonstrando que o segmento high-risk de fato possui a maior taxa:

### Tabela 11
|  | merchant_category | total_transacoes | transacoes_fraudulentas | valor_total | valor_fraudado | risk_score_medio | taxa_fraude |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 5 | viagem | 3.010 | 160 | 545.957,31 | 25.966,40 | 50,824847 | 0,053156 |
| 2 | saude | 2.951 | 73 | 528.445,88 | 11.141,53 | 50,023450 | 0,024737 |
| 0 | alimentacao | 5.976 | 138 | 1.055.465,65 | 21.017,00 | 49,685634 | 0,023092 |
| 4 | varejo | 9.047 | 203 | 1.524.470,13 | 34.870,54 | 50,312766 | 0,022438 |
| 3 | servicos | 4.576 | 90 | 831.119,26 | 14.197,18 | 49,922489 | 0,019668 |
| 1 | eletronico | 4.440 | 87 | 722.385,27 | 12.175,28 | 50,010595 | 0,019595 |

======================================================================

GOLD 3 — FRAUDE POR SEGMENTO

======================================================================
### Tabela 12
|  | segment | total_transacoes | transacoes_fraudulentas | valor_fraudado | taxa_fraude |
| --- | --- | --- | --- | --- | --- |
| 0 | high-risk | 2.915 | 282 | 42.156,04 | 0,096741 |
| 2 | standard | 10.645 | 312 | 50.874,90 | 0,02931 |
| 1 | premium | 16.440 | 157 | 26.336,99 | 0,00955 |

Com os dados organizados, pôde-se realizar a evolução temporal, no intuito de perceber o comportamento das fraudes ao longo do ano:

======================================================================

GOLD 4 — FRAUDE POR PERÍODO

======================================================================

### Tabela 13
|  | ano | mes | total_transacoes | transacoes_fraudulentas | valor_total | valor_fraudado | taxa_fraude |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 2025 | 1 | 2.625 | 68 | 467.809,10 | 14.740,00 | 0,025905 |
| 1 | 2025 | 2 | 2.317 | 62 | 420.165,83 | 8.706,60 | 0,026759 |
| 2 | 2025 | 3 | 2.511 | 60 | 439.726,15 | 8.842,40 | 0,023895 |
| 3 | 2025 | 4 | 2.458 | 60 | 410.684,31 | 8.257,61 | 0,02441 |
| 4 | 2025 | 5 | 2.499 | 58 | 412.912,65 | 9.554,79 | 0,023209 |
| 5 | 2025 | 6 | 2.525 | 66 | 426.197,25 | 8.441,12 | 0,026139 |
| 6 | 2025 | 7 | 2.530 | 62 | 470.474,91 | 10.847,87 | 0,024506 |
| 7 | 2025 | 8 | 2.472 | 67 | 414.286,69 | 11.654,65 | 0,027104 |
| 8 | 2025 | 9 | 2.451 | 51 | 423.809,56 | 5.882,48 | 0,020808 |
| 9 | 2025 | 10 | 2.585 | 62 | 445.911,36 | 9.441,86 | 0,023985 |
| 10 | 2025 | 11 | 2.451 | 63 | 461.487,15 | 13.879,74 | 0,025704 |
| 11 | 2025 | 12 | 2.576 | 72 | 414.378,54 | 9.118,81 | 0,02795 |

Todas as tabelas da fase gold foram salvas como parquet no diretório gold.

DUCKDB

Para consultar e transformar os dados armazenados em parquet, foi utilizado o pacote DuckDB.

Inicialmente as tabelas gold foram disponibilizadas no DuckDB:



======================================================================

SERVING — DUCKDB

======================================================================

### Tabela 14
|  | name |
| --- | --- |
| 0 | gold_fraude_canal |
| 1 | gold_fraude_categoria |
| 2 | gold_fraude_periodo |
| 3 | gold_fraude_segmento |

Em seguida, foram feitas as consultas por canal e categoria:

### Tabela 15
|  | channel | total_transacoes | transacoes_fraudulentas | taxa_fraude | valor_total | valor_fraudado | risk_score_medio |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | app | 12.021 | 455 | 0,037850 | 2.007.907,06 | 63.909,91 | 49,822697 |
| 1 | atm | 3.007 | 54 | 0,017958 | 538.079,57 | 12.395,30 | 49,502441 |
| 2 | web | 9.004 | 157 | 0,017437 | 1.573.259,20 | 31.579,22 | 50,364439 |
| 3 | pos | 5.968 | 85 | 0,014243 | 1.088.597,67 | 11.483,50 | 50,593396 |

### Tabela 16
|  | merchant_category | total_transacoes | transacoes_fraudulentas | taxa_fraude | valor_total | valor_fraudado | risk_score_medio |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | viagem | 3.010 | 160 | 0,053156 | 545.957,31 | 25.966,40 | 50,824847 |
| 1 | saude | 2.951 | 73 | 0,024737 | 528.445,88 | 11.141,53 | 50,023450 |
| 2 | alimentacao | 5.976 | 138 | 0,023092 | 1.055.465,65 | 21.017,00 | 49,685634 |
| 3 | varejo | 9.047 | 203 | 0,022438 | 1.524.470,13 | 34.870,54 | 50,312766 |
| 4 | servicos | 4.576 | 90 | 0,019668 | 831.119,26 | 14.197,18 | 49,922489 |
| 5 | eletronico | 4.440 | 87 | 0,019595 | 722.385,27 | 12.175,28 | 50,010595 |

Por último, foi encerrada a conexão com o DuckDB.

Resumidamente, este foi o resultado do pipeline demonstrou que não houve perda de dados:

======================================================================

RESUMO FINAL — PIPELINE TECHPAY

======================================================================

1. RAW:     30,000 linhas

2. BRONZE:  30,000 linhas

3. SILVER:  30,000 linhas

4. Gold - canal:      4 registros

5. Gold - categoria:  6 registros

6. Gold - segmento:   3 registros

7. Gold - período:    12 registros

Linhas descartadas na Bronze: 0

Pipeline executado com sucesso.

E estes foram os arquivos produzidos:

======================================================================

ARQUIVOS PRODUZIDOS

======================================================================

data/

techpay.duckdb

raw/

avaliacao_transactions.csv

transactions_raw.parquet

silver/

transactions_silver.parquet

gold/

fraude_por_canal.parquet

fraude_por_categoria.parquet

fraude_por_segmento.parquet

fraude_por_periodo.parquet

bronze/

ano=2025/

mes=4/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=7/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=6/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=8/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=5/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=1/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=2/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=3/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=9/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=10/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=11/

631ef81cae124791b58842dfab0aeecf-0.parquet

mes=12/

631ef81cae124791b58842dfab0aeecf-0.parquet

