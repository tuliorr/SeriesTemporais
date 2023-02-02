# Previsão de Série Temporais

## Intro

* Esse repositório objetiva analisar um conjunto de dados e realizar a previsão de séries temporais

## Conjunto de dados

* Utilizou-se o conjunto de dados de séries temporais de clima registrado pelo Instituto Max Planck de Biogeoquímica. Este conjunto de dados contém 14 features diferentes, como temperatura do ar, pressão atmosférica e umidade. Estes dados foram coletados a cada 10 minutos, a partir de 2003. Contudo, serão utilizados apenas os dados coletados entre 2009 e 2016

## Objetivo

* Realizar previsões sobre a variável 'T(degC)' com no mínimo dois passos a frente

## Tratamento inicial

* Inicialmente, foram removidas as linhas duplicadas e o conjunto de dados foi segmentado em medidas por hora ao invés de a cada 10 minutos

## Análise exploratória

* Criou-se uma série temporal com a variável a ser prevista 'T(degC)' e foram plotados o histograma da série e a sua variação dos valores ao longo do tempo para verificar o seu comportamento ao longo das observações

![hist](/images/hist.png)

![plot](/images/plot.png)

* Analisando preliminarmente os gráficos acima, a série aparenta ter distribuição normal e ser estacionária

* A série foi redimensionada para medidas diárias utilizando a média das medidas por hora de cada dia com o objetivo de analisar a sazonalidade nas observações

![plot_daily](/images/plot_daily.png)

* Verifica-se no gráfico acima que a série possui uma tendência de alta seguida por uma tendência de baixa no período de um ano

* Análise da decomposição sazonal da série de medidas diárias (média das medidas por hora)

![decompose](/images/decompose.png)

**Interpretação:**

**Tendência - Trend**

* A tendência representa a direção geral da série ao longo de um período de tempo. Dessa forma, observa-se que a série tem uma tendência de alta no período de analisado, mas se encontra na faixa entre 7.5 e 10.0.

**Sazonalidade - Seasonal**

* A sazonalidade retrata um padrão distinto e repetitivo em intervalos de tempo regulares resultante de vários fatores sazonais. Nesse contexto, nota-se que a série possui um padrão de alta seguido por uma baixa no intervalo anual.

**Residual - Resid**

* O residual corresponde ao componente irregular presente nas flutuações da série após a remoção dos componentes anteriores.

* Aplicação do teste Dickey Fuller e KPSS para verificar se a série é estacionária

**Teste de Dickey Fuller e KPSS**

* Para determinar se a série é estacionária, isto é, se a sua média e variância são constantes durante o tempo, foram utilizados os testes de (Augmented) Dickey Fuller e KPSS.

* O teste **Augmented Dickey-Fuller (ADF)** é conhecido por teste de raiz unitária, a qual é a causa para a não estacionariedade:

```Hipótese nula (H0): A série temporal possui raiz unitária e não é estacionária```

```Hipótese alternativa (H1): A série temporal não possui raiz unitária e é estacionária```

* Foi especificado o nível de significância em 5% e a hipótese nula é rejeitada (série estacionária) se o valor-P < 0.05

        Teste Estatistico Dickey Fuller      -3.5847
        Valor-P                               0.0061
        Lags Usados                          18.0000
        Número de observações usadas       2904.0000
        Valores Críticos (1%)                -3.4326
        Valores Críticos (5%)                -2.8625
        Valores Críticos (10%)               -2.5673

        Resultado: A série temporal é estacionária

* O teste **KPSS** objetiva reduzir a incerteza do teste ADF:

```Hipótese nula (H0): A série temporal é estacionária (oposto ao ADF)```

```Hipótese alternativa (H1): A série temporal não é estacionária```

* Foi especificado o nível de significância em 5% e a hipótese alternativa é rejeitada (série estacionária) se o valor-P > 0.05

        Teste Statistico KPSS       0.2011
        Valor-P                     0.1000
        Lags utilizados            31.0000
        Valores Críticos (10%)      0.3470
        Valores Críticos (5%)       0.4630
        Valores Críticos (2.5%)     0.5740
        Valores Críticos (1%)       0.7390

        Resultado: A série temporal é estacionária

* Os testes estatísticos Dickey Fuller e KPSS indicaram que a série é estacionária, isto é:

```Dickey Fuller (H1): p_value > 0.05```

```KPSS (H0): p_value < 0.05```

## Modelagem

Para realizar a previsão de valores da série temporal foram utilizadas as Redes Neurais Artificiais recorrentes (RNN), em específico as Redes de Memória de Curto Prazo Longo ou **LSTM** (Long Short-Term Memory), por se tratar de um algoritmo adequado para previsão de séries temporais e ser capaz de aprender dependências de longo prazo.

Nesse contexto, antes de elaborar o modelo propriamente dito, se faz necessário fazer a transformação dos dados, os quais servirão como entrada no modelo.

A transformação dos dados será dividida em 3 etapas, quais sejam:

* Transformar as séries temporais num problema de aprendizagem supervisionado, isto é, será criada a entrada (X) e a saída (y) (ou label) da série.

* Definição dos conjuntos de treinamento, validação e teste.

Inicialmente, foi desenvolvida uma função para obter o X (input) e o y (output). Essa função considerou uma janela consecutiva de 5 observações, as quais foram utilizadas para prever o y com um intervalo de 24 horas (OFFSET) após a última observação da "janela", conforme exemplificado abaixo:
     
        X = [[1],
             [2],
             [3],
             [4],
             [5]],
            
            [[2],
             [3],
             [4],
             [5],
             [6]],
             
             .
             .
             .

        y = [[5 + 24],
             [6 + 24]],
             
             .
             .
             .

Como a série foi caracterizada anteriormente como estacionária, não foi necessário escalar os valores da série. Segmentação da série em treino, validação e teste com 70%, 20% e 10% de todas as observações, respectivamente

Utilizou-se o modelo sequencial da biblioteca Keras, o qual permite inserir camadas de uma rede neural artificial em série, onde o output da primeira camada serve como input da segunda, e assim por diante. Como mencionado anteriormente, a arquitetura de rede neural utilizada foi a LSTM com 32 neurônios, seguida de uma camada com 8 neurônios utilizando a função de ativação ReLU (linear retificada) e por último a camada de saída.


        Model: "sequential"
        _________________________________________________________________
        Layer (type)                Output Shape              Param #   
        =================================================================
        lstm (LSTM)                 (None, 32)                4352      
                                                                        
        dense (Dense)               (None, 8)                 264       
                                                                        
        dense_1 (Dense)             (None, 1)                 9         
                                                                        
        =================================================================
        Total params: 4,625
        Trainable params: 4,625
        Non-trainable params: 0
        _________________________________________________________________

Os hiperparâmetros utilizados no modelo foram os seguintes:

* Perda = Mean Squared Error (MSE)

* Otimizador = Adam

* Métrica = Mean Absolut Error (MAE)

## Resultados

Nessa seção, o modelo é importado e são realizadas as predições utilizando o ```X_test```.Como foi utilizado o parâmetro "offset" na transformação nos dados, o resultado da predição precisa ser "deslocado para trás" na mesma medida do offset (-24) para as suas observações coincidirem com as respectivas observações reais presentes no ```y_test```.

![preditoXreal](/images/preditoXreal.png)

Pode-se notar visualmente que a predição realizada pelo modelo ficou próxima ao valor real e que o modelo conseguiu identificar os momentos de alta e de baixa.

        Root Mean Squared Error: 0.9608540980720118
        Coeficiente de Determinação: 0.9848603680356965

As métricas utilizadas para avaliação do modelo foram o RMSE e o Coeficiente de Determinação (R²). A Raiz do Erro Quadrático Médio (RMSE) representa a diferença entre os valores preditos e observados do modelo, ou seja, quanto menor essa diferença, menor será o erro. Já o R² indica o quão próximo as medidas reais estão do modelo, isto é, quanto maior o seu valor, melhor o modelo está ajustado aos valores reais.

Analisando o RMSE obtido pelo modelo, pode-se concluir que o erro manteve-se dentro de um limite aceitável, visto que as predições podem estar diferentes 0.96°C para mais ou para menos. Além disso, o modelo conseguiu identificar o momentos de alta e de baixa que podem ser observados no Gráfico predito x real. Ressalta-se que em determinados problemas, é mais importante saber o "quando" do que o "quanto", isto é, o momento do pico ou do vale ao invés da quantidade para atingir tais momentos.

Já o R² do modelo desenvolvido atingiu um valor de 0.98 e provavelmente está sobreajustado aos dados (overfitting), isto é, o modelo tende a aumentar o erro com dados novos que sejam muito diferentes dos dados utilizados para treinamento. Entretanto, o sobreajuste do modelo desenvolvido pode não significar uma ineficiência, visto que a série temporal é estacionária e não possui grande variação de valores no período observado. 

Adicionalmente, o modelo atingiu uma eficácia adequada utilizando dados de teste para predição, os quais eram desconhecidos na etapa de treinamento do modelo.