# Previsão de Direção do Preço da PETR4 

Projeto de estudo que investiga se é possível prever se a PETR4 vai fechar em
alta ou em baixa daqui a 5 dias úteis, usando indicadores técnicos e variáveis
macro (Ibovespa, petróleo Brent, câmbio USD/BRL).

## Pergunta de negócio

O fechamento da PETR4 em 5 dias úteis será maior do que o fechamento de hoje?
(classificação binária: **Sobe** / **Cai**)

## Resultado principal

Nenhum modelo testado supera de forma confiável um baseline ingênuo (sempre
prever a classe majoritária):

| Modelo | CV médio (5 folds, TimeSeriesSplit) |
|---|---|
| Baseline (classe majoritária) | 56,16% (+/- 2,17%) |
| Random Forest | 55,46% (+/- 2,51%) |
| XGBoost | 53,95% (+/- 3,15%) |
| Decision Tree | 49,14% (+/- 3,87%) |

A causa raiz é uma quebra de regime: o período de treino (2017-2024) teve uma
tendência de alta persistente na PETR4, enquanto o período de teste
(2024-2026) foi mais equilibrado e menos volátil. Isso explica por que até
modelos "otimizados" via GridSearchCV melhoram no CV mas pioram no holdout
final. Os detalhes completos dessa investigação estão no notebook, na seção
8.2 e na conclusão.

## Estrutura do notebook

1. Configuração e constantes
2. Coleta de dados (preço, petróleo, câmbio)
3. Engenharia de features
4. Definição do target
5. Análise exploratória
6. Baseline
7. Modelos e validação (TimeSeriesSplit)
8. Otimização de hiperparâmetros
9. Importância das features
10. Backtest da estratégia
11. Conclusões

## Dados

Os dados são baixados via [`yfinance`]:

- **PETR4.SA**: preço do ativo (OHLCV)
- **^BVSP**: Ibovespa
- **BZ=F**: petróleo Brent
- **BRL=X**: câmbio USD/BRL

Todas as séries de contexto macro são deslocadas em um dia (`shift(1)`) para
evitar vazamento de dados: no momento da previsão, o fechamento de "hoje"
dessas séries ainda não estaria disponível.

## Features

- **Tendência**: médias móveis (5, 20, 50, 200 dias) e razões entre elas
- **Volatilidade**: desvio padrão do retorno, ATR normalizado
- **Momentum**: RSI, MACD, momentum de 5 dias
- **Volume e candle**: volume relativo, corpo e sombra do candle
- **Bollinger**: posição do preço relativa às bandas
- **Contexto macro**: retorno do Ibovespa, Brent e USD/BRL
- **Sazonalidade**: dia da semana

## Modelos

- Decision Tree
- Random Forest
- XGBoost
- Baseline (`DummyClassifier`, classe majoritária)

Validação com `TimeSeriesSplit` (5 folds) e otimização de hiperparâmetros com
`GridSearchCV`, sempre respeitando a ordem temporal dos dados.

## Como rodar

```bash
pip install pandas numpy matplotlib yfinance scikit-learn xgboost
jupyter notebook main.ipynb
```

O notebook baixa os dados direto do Yahoo Finance ao ser executado, não
precisa de nenhum arquivo de dados local.

## Limitações

- O backtest de estratégia e a análise de importância de features ainda
  precisam ser conferidos com os dados mais recentes.
- A comparação de holdout usa uma única janela de teste; uma validação mais
  robusta usaria múltiplas janelas (walk-forward).
- O backtest ignora custos de transação e slippage.
