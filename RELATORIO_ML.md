# Relatório de Machine Learning: Previsão de lotação nas estações do Metrô de São Paulo

**Global Solution 2026.2 - CitizenTech | Inteligência Artificial & Machine Learning**

Integrantes: Pietro Vitor Pezzente (RM 557283), Eric Darakjian (RM 557082), Kauã Soares Guimarães (RM 559044), Enzo Mikael Sanches Baptista Paes Fontes (RM 558887).

Notebook: [`ml/previsao_lotacao_metro.ipynb`](ml/previsao_lotacao_metro.ipynb)

---

## 1. Problema

Quem depende do metrô em São Paulo não sabe com antecedência se a estação vai estar mais cheia ou mais vazia que o normal. Feriados, emendas, férias escolares e o dia da semana mudam muito o movimento, e hoje o passageiro só descobre isso quando chega na plataforma.

**Cidadão beneficiado:** quem usa as Linhas 1-Azul, 2-Verde, 3-Vermelha e 15-Prata no dia a dia (trabalhadores, estudantes, idosos e pessoas com mobilidade reduzida, que sofrem mais com plataformas lotadas).

**Pergunta de ML:** *dá para prever quantas pessoas vão entrar em cada estação do Metrô SP em um dia, usando o calendário e o histórico recente da própria estação?*

É um problema de **regressão**: o alvo é o número de entradas (em mil passageiros) por estação e por dia. Depois, a previsão é transformada em um nível simples para o cidadão: **tranquilo, normal ou cheio**.

## 2. Dados reais e públicos

- **Fonte:** Companhia do Metropolitano de São Paulo, Portal da Transparência, seção Demanda (https://transparencia.metrosp.com.br/dataset/demanda).
- Como o portal estava em manutenção quando fizemos o trabalho, baixamos o mesmo dado já consolidado em CSV pelo projeto aberto `metrosp` (https://github.com/viniciusoike/metrosp), que junta os relatórios mensais do Metrô.
- **Período:** 01/01/2020 a 30/09/2026, um registro por estação por dia (160.646 linhas, 62 estações).
- **Variável principal:** `passengers`, entradas de passageiros no dia, em milhares.
- Não há dados pessoais, só totais por estação. Detalhes em [`ml/dados/README.md`](ml/dados/README.md).

## 3. Escolha do modelo

Usamos o **Random Forest Regressor** (scikit-learn), porque:

- o alvo é numérico (regressão);
- o efeito das variáveis não é linear (um feriado derruba muito mais o movimento de uma estação de escritórios do que o de um terminal rodoviário), e as árvores captam essas interações sem precisar criá-las à mão;
- não precisa normalizar os dados;
- a média de muitas árvores reduz o overfitting de uma árvore sozinha;
- mostra a importância das variáveis, o que ajuda a explicar o resultado.

Como comparação, usamos uma **linha de base** simples: "hoje vai ser igual ao mesmo dia da semana passada".

## 4. Preparação e divisão treino/teste

1. **Limpeza:** sem nulos e sem duplicados. Removemos 2020 e 2021 (pandemia, demanda fora do normal), os 4 dias com 0 entradas (estação fechada) e linhas sem 4 semanas de histórico.
2. **Variáveis criadas:**
   - calendário: `dia_semana`, `mes`, `dia_mes`, `ano`;
   - `feriado`, `vespera_feriado`, `emenda` (dia útil entre feriado e fim de semana, ou Quarta-feira de Cinzas) e `feriado_semana_passada`. Os feriados foram listados à mão (nacionais, 25/01, 9 de julho, 20/11 e Carnaval), com as datas móveis calculadas pela Páscoa;
   - identificação: `linha` e `estacao_id`;
   - histórico da estação: `lag_7`, `lag_14` e `media_4_semanas` (média do mesmo dia da semana nas 4 semanas anteriores). Elas usam só dados do passado, que já estariam disponíveis no dia da previsão.
3. **Divisão por tempo** (não aleatória, para o modelo não "ver o futuro"):
   - **Treino:** 2022 a 2025, 95.307 linhas;
   - **Teste:** janeiro a setembro de 2026, 18.018 linhas (cerca de 16%).

## 5. Treino com diferentes hiperparâmetros

Testamos **12 combinações** de 3 hiperparâmetros com `GridSearchCV`, usando **validação cruzada só no treino** com `TimeSeriesSplit` (4 divisões, sempre treinando no passado e validando no período seguinte). Métrica de escolha: **MAE**.

| Hiperparâmetro | Valores |
|---|---|
| `n_estimators` | 50, 150 |
| `max_depth` | 8, 16, sem limite |
| `min_samples_leaf` | 1, 10 |

Principais resultados da validação cruzada (MAE em mil passageiros):

| n_estimators | max_depth | min_samples_leaf | MAE treino | MAE validação |
|---|---|---|---|---|
| **150** | **sem limite** | **1** | **0,48** | **1,70** |
| 150 | 16 | 1 | 0,82 | 1,71 |
| 150 | sem limite | 10 | 1,26 | 1,88 |
| 150 | 8 | 1 | 1,62 | 2,04 |
| 50 | 8 | 10 | 1,69 | 2,11 |

- `max_depth=8` deu **underfitting** (erro alto em treino e validação).
- Árvores profundas com `min_samples_leaf=1` tiveram o menor erro de validação, mas com uma diferença grande para o treino (sinal de **overfitting**).
- `min_samples_leaf=10` reduziu essa diferença pela metade, mas aumentou o erro de validação.
- 150 árvores foram só um pouco melhores que 50 e levaram quase o triplo do tempo.

**Melhor combinação:** `n_estimators=150`, `max_depth=None`, `min_samples_leaf=1`.

## 6. Resultados em treino e teste

| Métrica | Linha de base (teste) | Random Forest (treino) | Random Forest (teste) |
|---|---|---|---|
| MAE (mil passageiros) | 3,13 | 0,47 | **1,44** |
| RMSE (mil passageiros) | 8,29 | 1,08 | **3,16** |
| R² | 0,915 | 0,998 | **0,988** |
| Erro % médio | 14,3% | 2,5% | **6,2%** |

- **Overfitting:** o erro no treino é cerca de 3 vezes menor que no teste, então o modelo decora parte do treino. Mesmo assim o erro no teste (1,44) ficou parecido com o erro da validação cruzada (1,70) e o R² continua perto de 1 em um período que o modelo nunca viu. O overfitting existe, mas não prejudica a previsão em dados novos.
- O Random Forest **reduziu o erro pela metade** em relação a repetir a semana passada.
- **Onde erra mais:** em feriados o erro médio é 14,6%, contra 5,9% em dias normais. Os piores dias foram o começo de janeiro (recesso de fim de ano) e alguns dias atípicos que o calendário não explica (eventos, greves ou falhas).
- **Variáveis mais importantes:** `media_4_semanas`, `lag_7` e `lag_14` (o "tamanho" normal de cada estação) e depois `feriado`.
- **Nível de lotação:** comparando a previsão com a média da estação em dias úteis (tranquilo abaixo de 80%, normal de 80% a 110%, cheio acima de 110%), o modelo acertou o nível real em **89,2%** dos casos de 2026. Os erros acontecem quase sempre perto dos limites entre as faixas.

Os gráficos de treino x validação, real x previsto, séries de 2026 por estação, importância das variáveis e matriz de confusão dos níveis estão no notebook.

## 7. Conclusão e leitura para o cidadão

Sim, é possível prever o movimento diário de cada estação do Metrô SP com dados públicos: o modelo erra em média cerca de 1,4 mil entradas por estação por dia (6%), e acerta se o dia vai ser tranquilo, normal ou cheio em quase 9 de cada 10 casos.

Para o cidadão, isso vira um aviso simples no aplicativo da nossa CitizenTech, por exemplo: *"Amanhã a estação República deve estar mais cheia que o normal"* ou *"Na sexta de emenda a Sé deve estar mais vazia"*. Com isso a pessoa pode sair mais cedo, escolher outra estação de embarque ou planejar melhor a viagem. Pessoas idosas e com mobilidade reduzida, que mais sofrem com lotação, são as mais beneficiadas.

**Limitações:** os dados são totais por dia, então o modelo não sabe o horário de pico; ele também não conhece eventos (shows, jogos), greves e falhas, que geram os maiores erros. Próximos passos: incluir dados de eventos e de clima (INMET) e, se o Metrô publicar, dados por faixa horária.

**ODS relacionados:** ODS 11 (cidades e comunidades sustentáveis), ODS 9 (indústria, inovação e infraestrutura) e ODS 10 (redução das desigualdades).
