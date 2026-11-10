# Global Solution 2026.2 - CitizenTech: Previsão de lotação no Metrô de São Paulo

## Integrantes do grupo

- Pietro Vitor Pezzente | RM 557283
- Eric Darakjian | RM 557082
- Kauã Soares Guimarães | RM 559044
- Enzo Mikael Sanches Baptista Paes Fontes | RM 558887

---

## Ficha da CitizenTech

| Campo | Resposta |
|---|---|
| Nome da CitizenTech | Vagão Livre |
| Cidade e bairro | São Paulo, estações das Linhas 1-Azul, 2-Verde, 3-Vermelha e 15-Prata do Metrô |
| Cidadão beneficiado | Quem depende do metrô no dia a dia: trabalhadores, estudantes, idosos e pessoas com mobilidade reduzida |
| Problema, em uma frase | O passageiro não sabe com antecedência se a estação vai estar mais cheia ou mais vazia que o normal |
| Como funciona | Um modelo de Machine Learning prevê as entradas de cada estação no dia e transforma a previsão em um nível simples: tranquilo, normal ou cheio |
| O que cada parte faz (ML) | Treina um Random Forest com dados reais do Metrô SP e entrega a previsão de lotação por estação e dia |
| Link do repositório | https://github.com/ericdarak/GlobalSolution-ML |
| Indicador de impacto | Erro médio da previsão (MAE) e % de acerto do nível de lotação (89,2% em 2026) |
| ODS conectados | ODS 11, ODS 9 e ODS 10 |

---

## Parte de Machine Learning

- **Pergunta:** dá para prever quantas pessoas vão entrar em cada estação do Metrô SP em um dia, usando o calendário e o histórico recente da estação?
- **Dados:** entradas diárias de passageiros por estação, Metrô SP, de 01/2020 a 09/2026 (dados reais e públicos). Fonte e detalhes em [`ml/dados/README.md`](ml/dados/README.md).
- **Modelo:** Random Forest Regressor, com 12 combinações de `n_estimators`, `max_depth` e `min_samples_leaf` testadas por validação cruzada temporal só no treino.
- **Divisão:** treino de 2022 a 2025, teste de janeiro a setembro de 2026.
- **Resultado no teste:** MAE de 1,44 mil passageiros por estação por dia (erro médio de 6,2%) e R² de 0,988, contra MAE de 3,13 mil repetindo a semana anterior.

O relatório completo está em [`RELATORIO_ML.md`](RELATORIO_ML.md).

---

## Estrutura

```text
GlobalSolution_ML/
├── README.md
├── RELATORIO_ML.md
└── ml/
    ├── previsao_lotacao_metro.ipynb   # notebook executado
    ├── requirements.txt
    └── dados/
        ├── README.md                  # fonte, link e período dos dados
        └── entradas_diarias_estacoes_metro_sp.csv
```

---

## Como executar

```bash
cd ml
pip install -r requirements.txt
jupyter notebook previsao_lotacao_metro.ipynb
```

Depois, execute as células em ordem. O notebook roda em menos de 1 minuto em um computador comum.
