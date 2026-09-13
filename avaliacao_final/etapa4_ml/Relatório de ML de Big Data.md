**1- Modelo de Aprendizado de Máquina para Detecção de Fraudes**

Foi desenvolvido um modelo de aprendizado de máquina para classificação de transações fraudulentas da TechPay. O objetivo foi utilizar características das transações e dos clientes para estimar a probabilidade de uma operação ser fraudulenta, apoiando a identificação de transações de maior risco.

O modelo utilizado foi a Regressão Logística, uma técnica de classificação supervisionada adequada para problemas binários, neste caso, distinguindo entre transações fraudulentas e não fraudulentas.

## **1.1-** **Preparação dos dados**

A primeira etapa aborda a utilização dos dados tratados na camada “Silver” do “pipeline” de “Big Data”. Foram selecionadas variáveis relevantes para a classificação, incluindo:

* Valor da transação (amount);  
* Pontuação de risco (risk\_score);  
* Pontuação de crédito (credit\_score);  
* Segmento do cliente (segment);  
* Indicador de fraude (is\_fraud), utilizado como variável-alvo.

A variável indicador de fraude foi convertida para o formato numérico necessário ao treinamento do modelo. A variável categórica segmento do cliente passou por indexação e codificação One-Hot, permitindo sua utilização pelo modelo.

### **1.2- Construção do modelo**

As variáveis foram organizadas em um vetor de características por meio do “VectorAssembler”. Em seguida, foi construída uma “pipeline” do “PySpark”, integrando as etapas de transformação dos dados e treinamento da regressão logística.

O conjunto de dados, composto por 30.000 registros, foi dividido em: 24.032 registros (aproximadamente 80%) para treinamento e 5.968 registros (aproximadamente 20%) para teste. O modelo foi treinado com os dados de treinamento e posteriormente avaliado sobre o conjunto de teste, que não participou do processo de ajuste.

### **1.3- Avaliação dos resultados**

Os resultados apresentam bom desempenho geral do classificador. A acurácia de 97,50% indica que a maior parte das transações do conjunto de teste foi classificada corretamente, como pode-se observar na tabela abaixo. 

| Métrica | Resultado |
| :---: | :---: |
| AUC | 0,7333 |
| Acurácia | 97,50% |
| Precisão | 95,07% |
| Recall | 97,50% |
| F1-score | 96,27% |

A precisão de 95,07% demonstra que, entre as transações classificadas pelo modelo como fraudulentas, uma proporção elevada realmente correspondia à classe de fraude. O recall de 97,50% é relevante para o problema de detecção de fraudes. O resultado indica que o modelo conseguiu identificar a grande maioria das fraudes presentes no conjunto de teste. Em um sistema antifraude, essa característica é importante porque deixar de identificar uma fraude pode representar prejuízo financeiro para a empresa.

O F1-score de 96,27%, por combinar precisão e recall, reforça que o modelo apresentou um bom equilíbrio entre identificar fraudes e evitar classificações incorretas. No entanto, a AUC de 0,73 indica uma capacidade de discriminação moderada. Portanto, embora as métricas obtidas no ponto de classificação sejam elevadas, o modelo ainda possui espaço para melhorar sua capacidade de separar, de forma consistente, transações fraudulentas das legítimas em diferentes níveis de probabilidade.

	