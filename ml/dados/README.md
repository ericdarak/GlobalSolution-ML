# Dados utilizados

## Arquivo

`entradas_diarias_estacoes_metro_sp.csv`

## Fonte

- **Origem dos dados:** Companhia do Metropolitano de São Paulo (Metrô SP), Portal da Transparência, seção Demanda: https://transparencia.metrosp.com.br/dataset/demanda
- **Onde baixamos:** o portal do Metrô estava fora do ar (página de manutenção) quando fizemos o trabalho, em novembro de 2026. Por isso baixamos o mesmo dado já consolidado em CSV pelo projeto aberto `metrosp`, que junta os relatórios mensais publicados pelo Metrô:
  - Repositório: https://github.com/viniciusoike/metrosp
  - Arquivo original: `data-raw/outputs/processed/metro_sp_station_daily_current.csv`
  - Link direto: https://raw.githubusercontent.com/viniciusoike/metrosp/main/data-raw/outputs/processed/metro_sp_station_daily_current.csv
- **Data do download:** 09/11/2026.
- O arquivo foi apenas renomeado. Nenhum valor foi alterado.

## Período e cobertura

- De **01/01/2020 a 30/09/2026**, um registro por dia para cada estação.
- Linhas operadas pelo Metrô: **1-Azul, 2-Verde, 3-Vermelha e 15-Prata** (62 estações, 160.646 linhas).
- As Linhas 4-Amarela e 5-Lilás são operadas por concessionárias e não estão neste arquivo.

## Colunas

| Coluna | Descrição |
|---|---|
| `date` | data (AAAA-MM-DD) |
| `year` | ano |
| `line_number` | número da linha (1, 2, 3 ou 15) |
| `station_code` | código curto da estação |
| `station_name` | nome da estação |
| `passengers` | entradas de passageiros na estação no dia, em **milhares** |

## Observações

- São dados **reais e públicos**, agregados por estação e dia. Não há nenhum dado pessoal.
- Estações de transferência (ex.: Sé, nas Linhas 1 e 3) aparecem uma vez para cada linha.
- 2020 e 2021 foram afetados pela pandemia. No notebook usamos 2022 em diante para treino e teste.
- Há 4 registros com 0 entradas (estação fechada no dia), que foram removidos no notebook.
