# Backtesting.py

## Overview

  A **hybrid trend-following + mean-reversion strategy** backtested on Bitcoin (BTC-USD)
  using [`backtesting.py`](https://kernc.github.io/backtesting.py/).

  The strategy enters on two distinct signals — trend confirmation or a Bollinger Band
  mean-reversion bounce — and uses an **ATR-based adaptive stop-loss** to size risk
  dynamically to current market volatility rather than a fixed percentage.

  ---


  ### Entry — either condition triggers

  | Condition | Type | Rule |
  |---|---|---|
  | Trend following | Momentum | SMA(10) > SMA(50) — short-term trend above long-term |
  | Mean reversion | Contrarian | Price touches or breaches lower Bollinger Band (±3% buffer) |

  ### Exit — either condition triggers

  | Condition | Rule |
  |---|---|
  | Trend reversal | SMA(10) < SMA(50) |
  | Upper band touch | Price reaches upper Bollinger Band (±3% buffer) |
  ---
  Parameters

  sma_fast  = 10    # short-term trend MA
  sma_slow  = 50    # long-term trend MA
  bb_len    = 20    # Bollinger Band window
  bb_std    = 2     # Bollinger Band width (standard deviations)
  atr_len   = 14    # ATR lookback
  atr_mult  = 2     # stop-loss = 2 × ATR below entry

  ---
  Backtest Setup

  ┌──────────────────┬───────────────────────────────────┐
  │     Property     │               Value               │
  ├──────────────────┼───────────────────────────────────┤
  │ Asset            │ BTC (Bitcoin)                     │
  ├──────────────────┼───────────────────────────────────┤
  │ Period           │ 2024-07-31 → 2025-12-31           │
  ├──────────────────┼───────────────────────────────────┤
  │ Starting Capital │ $10,000                           │
  ├──────────────────┼───────────────────────────────────┤
  │ Commission       │ 0.2% per trade                    │
  ├──────────────────┼───────────────────────────────────┤
  │ Engine           │ backtesting.py event-driven, OHLC │
  └──────────────────┴───────────────────────────────────┘

  ---
  Results

  Metric                        Value
  ──────────────────────────────────────────
  Return                       +77.20%
  Buy & Hold Return            +43.44%
  CAGR                          32.09% / yr
  Sharpe Ratio                   0.956
  Sortino Ratio                  2.513
  Calmar Ratio                   2.434
  Alpha                        +56.99%
  Max Drawdown                 -20.44%
  Exposure Time                 62.18%
  # Trades                          25
  Win Rate                       56.0%
  Avg Trade Return               +2.32%
  Profit Factor                  3.242
  Expectancy                    +2.68%
  SQN                            1.333
  Kelly Criterion                0.329

  ---
  Limitations

  - Short backtest window
  - Single bull market regime
  - No parameter optimization
  - Crypto-specific dynamics (BTC trades 24/7..)
  ---
  Tech Stack

  Python 3.10+
  yfinance        market data
  pandas / numpy  data manipulation
  pandas-ta       technical indicators (Bollinger Bands, ATR)
  backtesting.py  event-driven backtesting engine

  ---
  Author

  Simone Buzzo 
  LinkedIn · GitHub
