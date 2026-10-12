# Projeto_LOL
Esse projeto consiste em previsões de vitórias. 


Projeto Final EBAC — Previsão de Vitória no League of Legends
1. Visão geral
Este projeto de Ciência de Dados tem como objetivo estimar a probabilidade de vitória do time azul em uma partida de League of Legends, utilizando indicadores registrados após os primeiros 10 minutos de jogo.
A proposta é transformar dados de desempenho em informações que possam apoiar uma equipe de análise competitiva na compreensão das vantagens e dos fatores associados ao resultado final. A previsão é uma estimativa estatística, não uma garantia de vitória.
2. Problema de negócio
Uma equipe de análise de desempenho deseja identificar, ainda no início de uma partida, quais sinais estão associados a uma maior probabilidade de vitória. Para investigar essa questão, foram analisados indicadores como diferença de ouro, diferença de experiência, abates, assistências e objetivos conquistados.
Pergunta central: até que ponto os dados dos primeiros 10 minutos ajudam a prever se o time azul vencerá a partida?
3. Fonte dos dados
O projeto utiliza a base League of Legends Diamond Ranked Games (10 min), disponibilizada publicamente no Kaggle:
- Acessar o dataset no Kaggle
- Dimensão original: 9.879 partidas e 40 colunas.
- Unidade de análise: cada linha representa uma partida.
- Variável-alvo: blueWins, em que 0 significa derrota do time azul e 1 significa vitória do time azul.
4. Ferramentas e bibliotecas
O notebook utiliza Python e bibliotecas para diferentes etapas do processo:
- Pandas e NumPy: manipulação e análise dos dados.
- Matplotlib e Seaborn: construção de gráficos e exploração visual.
- Scikit-learn: separação entre treino e teste, validação cruzada, otimização dos hiperparâmetros e métricas de avaliação.
- XGBoost: treinamento do modelo XGBoost.
5. Etapas do projeto
Etapa 1 — Carregamento dos dados
A base foi carregada em um DataFrame do Pandas. Em seguida, foram verificadas suas dimensões e visualizadas as primeiras linhas para entender a estrutura dos registros e as variáveis disponíveis.
Etapa 2 — Verificação da qualidade dos dados
Foram analisados os tipos das colunas, a existência de valores ausentes e a presença de linhas duplicadas.
Resultado observado: não foram encontrados valores ausentes nem linhas duplicadas. As variáveis já estavam em formatos numéricos adequados às análises e aos modelos utilizados, portanto não foi necessário realizar imputação ou conversões adicionais.
Etapa 3 — Estatística descritiva
Foi utilizado describe() para observar medidas como média, desvio-padrão, valores mínimos e máximos. Essa análise ajudou a compreender a escala e a dispersão dos indicadores e a identificar valores que mereciam investigação.
Etapa 4 — Análise da variável-alvo
A distribuição de blueWins foi analisada por meio de contagens, percentuais e gráfico de barras.
Resultado	Partidas	Percentual
Derrota do time azul (0)	4.949	50,1%
Vitória do time azul (1)	4.930	49,9%


As classes estão praticamente equilibradas. Por isso, não foi necessário aplicar técnicas de balanceamento como SMOTE nesta etapa.
Etapa 5 — Comparação dos indicadores por resultado
Foram comparadas as médias de alguns indicadores entre partidas em que o time azul perdeu e partidas em que venceu. As variáveis observadas foram:
- blueGoldDiff: diferença de ouro entre os times;
- blueExperienceDiff: diferença de experiência;
- blueKills: abates do time azul;
- blueAssists: assistências do time azul;
- blueTowersDestroyed: torres destruídas pelo time azul.
A análise mostrou diferenças nas médias entre os grupos. Por exemplo, a diferença média de ouro foi negativa nas derrotas do time azul e positiva nas vitórias, indicando associação entre vantagem acumulada e resultado.
Etapa 6 — Distribuições e valores extremos
Foi construído um histograma de blueGoldDiff separado por resultado para comparar a distribuição da diferença de ouro. Também foram utilizados boxplots para observar valores extremos em blueGoldDiff, blueExperienceDiff e blueKills.
Os valores extremos foram mantidos, pois uma vantagem muito alta pode representar uma situação real de jogo, e não necessariamente um erro de registro. A decisão foi não remover observações apenas por serem extremas.
Etapa 7 — Análise de correlação
Foi calculada a correlação entre as variáveis numéricas e blueWins, além de uma matriz de correlação para visualizar relações entre os indicadores.
Entre as maiores correlações absolutas com o alvo estavam:
Variável	Correlação com blueWins
redGoldDiff	-0,5111
blueGoldDiff	0,5111
blueExperienceDiff	0,4896
redExperienceDiff	-0,4896


A análise também evidenciou que algumas variáveis dos times azul e vermelho são espelhos matemáticos. Por exemplo, a diferença de ouro do time azul corresponde à diferença oposta do time vermelho. Essa redundância foi considerada na preparação dos dados.
Etapa 8 — Preparação para a modelagem
Foram removidas as seguintes colunas:
- gameId: identificador da partida, sem utilidade direta como indicador de desempenho;
- redGoldDiff e redExperienceDiff: variáveis espelhadas das diferenças do time azul;
- redKills e redDeaths: informações equivalentes às mortes e aos abates do time azul, conforme a estrutura da base.
Após a remoção, o DataFrame ficou com 9.879 linhas e 35 colunas, incluindo a variável-alvo. Em seguida, os dados foram separados em:
- X: 34 variáveis preditoras;
- y: variável-alvo blueWins.
Etapa 9 — Separação entre treino e teste
Os dados foram divididos em 80% para treinamento e 20% para teste, usando train_test_split, random_state=42 e stratify=y.
- Treinamento: 7.903 partidas.
- Teste: 1.976 partidas.
O parâmetro stratify=y mantém a proporção das classes nos dois conjuntos. O teste foi reservado para avaliar os modelos em observações que não participaram do ajuste.
Etapa 10 — Decisão sobre padronização
Não foi aplicada a padronização com StandardScaler. A decisão foi intencional: a Árvore de Decisão e o XGBoost são modelos baseados em árvores e realizam divisões por valores de corte, não dependendo da escala das variáveis da mesma forma que algoritmos baseados em distância.
Etapa 11 — Modelo 1: Árvore de Decisão
A Árvore de Decisão foi escolhida como primeiro modelo por ser relativamente interpretável e servir como referência para a comparação.
Foi utilizada validação cruzada estratificada com cinco divisões (StratifiedKFold). O GridSearchCV testou diferentes valores de:
- max_depth: profundidade máxima da árvore;
- min_samples_leaf: quantidade mínima de observações em uma folha.
Melhores parâmetros encontrados: max_depth=2 e min_samples_leaf=1.
Acurácia média na validação cruzada: 0,7224.
Etapa 12 — Modelo 2: XGBoost
O XGBoost foi utilizado como segundo modelo. Ele combina várias árvores em um processo de boosting, buscando melhorar o desempenho preditivo.
A otimização por GridSearchCV, com a mesma validação cruzada de cinco divisões, avaliou:
- n_estimators: número de árvores;
- max_depth: profundidade máxima das árvores;
- learning_rate: taxa de aprendizagem.
Melhores parâmetros encontrados: learning_rate=0.05, max_depth=2 e n_estimators=100.
Acurácia média na validação cruzada: 0,7347.
Etapa 13 — Avaliação dos modelos
Os dois modelos foram avaliados no conjunto de teste com acurácia, precisão, recall, F1-score e ROC AUC.
Métrica	Árvore de Decisão	XGBoost
Acurácia	0,7206	0,7262
Precisão	0,7343	0,7254
Recall	0,6897	0,7262
F1-score	0,7113	0,7258
ROC AUC	0,7809	0,8064


Interpretação das métricas:
- Acurácia: proporção total de previsões corretas.
- Precisão: entre as previsões de vitória do time azul, quantas estavam corretas.
- Recall: entre as vitórias reais do time azul, quantas foram identificadas pelo modelo.
- F1-score: média harmônica entre precisão e recall.
- ROC AUC: capacidade de distinguir as classes considerando diferentes limiares de decisão.
O XGBoost apresentou a maior acurácia e o maior ROC AUC no teste, além de recall e F1-score superiores. A Árvore de Decisão apresentou precisão ligeiramente maior.
Etapa 14 — Matrizes de confusão
Foram construídas matrizes de confusão para observar os acertos e os erros de cada modelo. Elas permitem distinguir:
- Verdadeiro negativo: derrota azul prevista corretamente;
- Falso positivo: vitória azul prevista, mas o time azul perdeu;
- Falso negativo: derrota azul prevista, mas o time azul venceu;
- Verdadeiro positivo: vitória azul prevista corretamente.
Essa análise complementa as métricas agregadas e ajuda a entender quais tipos de erro os modelos cometem.
Etapa 15 — Curvas ROC
As curvas ROC foram utilizadas para comparar a capacidade dos modelos de separar vitórias e derrotas a partir das probabilidades previstas. O XGBoost obteve ROC AUC de 0,8064, contra 0,7809 da Árvore de Decisão, apresentando melhor capacidade de discriminação neste teste.
Etapa 16 — Importância das variáveis
Foram comparadas as importâncias das variáveis nos dois modelos. No XGBoost, blueGoldDiff foi a variável de maior importância (0,3675), seguida de blueExperienceDiff (0,2412). Indicadores como dragões, nível médio, abates e experiência total também contribuíram.
Na Árvore de Decisão, blueGoldDiff concentrou a importância do modelo ajustado. Isso mostra que a árvore final fez suas decisões principalmente com base nessa variável; não significa que os demais indicadores sejam irrelevantes em todos os modelos.
Etapa 17 — Correlação versus importância
A correlação e a importância do modelo foram comparadas para evitar interpretações equivocadas. A correlação mede a associação linear entre uma variável e o alvo, enquanto a importância representa a contribuição da variável para as decisões do modelo, conforme o critério utilizado pelo algoritmo.
Uma variável pode ter correlação moderada e ainda contribuir por meio de interações ou relações não lineares.
Etapa 18 — Data storytelling e recomendação de negócio
O notebook seleciona o modelo com maior ROC AUC e apresenta seus resultados para apoiar a interpretação do problema. Com base na avaliação realizada, o XGBoost foi o modelo com maior ROC AUC.
Como aplicação prática, uma equipe de análise competitiva poderia usar as probabilidades estimadas para:
- acompanhar indicadores associados à vantagem;
- apoiar análises de partidas e sessões de treinamento;
- identificar padrões em partidas vencidas e perdidas;
- desenvolver indicadores de acompanhamento de desempenho.
A previsão deve ser tratada como apoio à análise, e não como certeza de que uma equipe vencerá.
6. Principais resultados
- A base contém 9.879 partidas, sem valores ausentes ou duplicatas identificados nas verificações realizadas.
- A variável-alvo está equilibrada: 50,1% de derrotas e 49,9% de vitórias do time azul.
- blueGoldDiff e blueExperienceDiff se destacaram na análise exploratória e na interpretação dos modelos.
- Foram comparados dois algoritmos: Árvore de Decisão e XGBoost.
- No conjunto de teste, o XGBoost obteve 72,62% de acurácia e ROC AUC de 0,8064.
- Os resultados indicam que os indicadores dos primeiros 10 minutos carregam informação útil para estimar o resultado, mas não eliminam a incerteza da partida.
7. Limitações e cuidados
- O modelo estima o resultado a partir das variáveis presentes na base; seu desempenho pode mudar em outras temporadas, versões do jogo ou níveis competitivos.
- Associação e importância preditiva não demonstram, por si só, causalidade.
- Indicadores como diferença de ouro e experiência já resumem vantagens acumuladas aos 10 minutos. Portanto, o projeto prevê o resultado final a partir do estado observado da partida, e não apenas de características anteriores ao início do jogo.
- Os resultados de teste representam a divisão utilizada neste notebook e não garantem o mesmo desempenho em dados futuros.
- Antes de uma implantação real, seria importante testar o modelo em partidas mais recentes e monitorar sua performance ao longo do tempo.
