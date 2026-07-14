# PETR4 Price Direction Prediction

Study project investigating whether it's possible to predict if PETR4 will
close higher or lower 5 business days from now, using technical indicators
and macro variables (Ibovespa, Brent crude oil, USD/BRL exchange rate).

## Business question

Will PETR4's closing price in 5 business days be higher than today's close?
(binary classification: **Up** / **Down**)

## Main result

No tested model reliably beats a naive baseline (always predicting the
majority class):

| Model | Mean CV (5 folds, TimeSeriesSplit) |
|---|---|
| Baseline (majority class) | 56.16% (+/- 2.17%) |
| Random Forest | 55.46% (+/- 2.51%) |
| XGBoost | 53.95% (+/- 3.15%) |
| Decision Tree | 49.14% (+/- 3.87%) |

The root cause is a regime shift: the training period (2017-2024) had a
persistent uptrend in PETR4, while the test period (2024-2026) was more
balanced and less volatile. This explains why even models "optimized" via
GridSearchCV improve on CV but perform worse on the final holdout. Full
details of this investigation are in the notebook, in section 8.2 and in
the conclusion.

## Notebook structure

1. Setup and constants
2. Data collection (price, oil, exchange rate)
3. Feature engineering
4. Target definition
5. Exploratory analysis
6. Baseline
7. Models and validation (TimeSeriesSplit)
8. Hyperparameter tuning
9. Feature importance
10. Strategy backtest
11. Conclusions
12. Appendix: Results Visualizations

## Data

Data is downloaded via [`yfinance`]:

- **PETR4.SA**: asset price (OHLCV)
- **^BVSP**: Ibovespa
- **BZ=F**: Brent crude oil
- **BRL=X**: USD/BRL exchange rate

All macro context series are shifted by one day (`shift(1)`) to avoid data
leakage: at prediction time, "today's" close for these series would not yet
be available.

## Features

- **Trend**: moving averages (5, 20, 50, 200 days) and ratios between them
- **Volatility**: return standard deviation, normalized ATR
- **Momentum**: RSI, MACD, 5-day momentum
- **Volume and candle**: relative volume, candle body and shadow
- **Bollinger**: price position relative to the bands
- **Macro context**: Ibovespa, Brent, and USD/BRL returns
- **Seasonality**: day of the week

## Models

- Decision Tree
- Random Forest
- XGBoost
- Baseline (`DummyClassifier`, majority class)

Validation with `TimeSeriesSplit` (5 folds) and hyperparameter tuning with
`GridSearchCV`, always respecting the temporal order of the data.

## How to run

```bash
pip install -r requirements.txt
jupyter notebook petr4_direction_prediction.ipynb
```

The notebook downloads the data directly from Yahoo Finance when run; no
local data file is needed.

## Appendix

The notebook closes with an **Appendix: Results Visualizations** section,
added because live access to Yahoo Finance (`finance.yahoo.com`) was
blocked in the environment used to produce it. It contains two charts
built strictly from the numbers already reported in the Conclusions
section (no invented or estimated values):

- **A1. Model comparison: CV mean vs. holdout** — reproduces the model
  comparison table as a chart, contrasting each model's mean
  cross-validated accuracy (with error bars) against the baseline, and
  the optimistic CV score used during tuning against the real holdout
  accuracy (where the overfitting-to-CV problem becomes visible).
- **A2. Regime shift: train (2017-2024) vs. test (2024-2026)** —
  compares mean daily return, daily volatility, and the proportion of
  "Up" 5-day periods between the training and test windows, illustrating
  the regime-shift explanation from the conclusion.

Charts that require the raw OHLCV price series (correlation matrix,
per-class boxplots, confusion matrix / ROC / PR curves, hit/miss overlay
on price, and the strategy backtest) still need to be run in an
environment with data access — their code is already in place in the
main body of the notebook (sections 5, 9.1, 9.2, and 10).

## Limitations

- The appendix charts (see above) plot pre-computed values rather than
  being regenerated from raw data; the remaining charts and the feature
  importance analysis still need to be run against live data.
- The strategy backtest still needs to be checked against the most
  recent data.
- The holdout comparison uses a single test window; a more robust
  validation would use multiple windows (walk-forward).
- The backtest ignores transaction costs and slippage.
