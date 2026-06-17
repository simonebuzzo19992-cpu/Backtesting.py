# Backtesting.py
Backtesting.py is a powerful Python library I want to explore and study. Here my experiments :)

## Overview

  This notebook implements and backtests a **rule-based multi-indicator strategy**
  on NVIDIA Corp. (NVDA) using [`backtesting.py`](https://kernc.github.io/backtesting.py/).

  The strategy combines **trend, momentum, and volatility filters** to define
  precise entry and exit conditions, then evaluates performance across the full
  available price history (1999–2026) with realistic commission modeling.

  ---

  ## Strategy Logic

  ### Entry — all three conditions must hold simultaneously

  | Condition | Rule |
  |-----------|------|
  | Trend filter | SMA(10) > SMA(50) — price is in an uptrend |
  | Momentum trigger | MACD line crosses above Signal line |
  | Overbought guard | RSI(14) < 70 — not entering at extended levels |

  ### Exit — any one condition is sufficient

  | Condition | Rule |
  |-----------|------|
  | Momentum reversal | Signal line crosses above MACD line |
  | Overbought exit | RSI(14) > 75 |
  | Upper band touch | Price >= Bollinger Upper Band (20, 2σ) |
  ### Rationale

  The entry logic requires **trend confirmation before momentum entry** — a deliberate
  design choice to reduce false signals in choppy markets. The asymmetric exit logic
  (any one condition sufficient) prioritizes capital preservation over maximizing
  individual trade duration.

  ---

  ## Parameters

  ```python
  sma_fast    = 10    # fast trend MA
  sma_slow    = 50    # slow trend MA
  rsi_len     = 14    # RSI lookback
  rsi_max_buy = 70    # no entry if overbought
  rsi_exit    = 75    # exit if RSI spikes
  bb_len      = 20    # Bollinger Band window
  bb_std      = 2     # Bollinger Band width
  macd_fast   = 12    # MACD fast EMA
  macd_slow   = 26    # MACD slow EMA
  macd_sig    = 9     # MACD signal EMA

  ---
  Backtest Setup

  ┌──────────────────┬─────────────────────────────────────┐
  │     Property     │                Value                │
  ├──────────────────┼─────────────────────────────────────┤
  │ Asset            │ NVDA (NVIDIA Corporation)           │
  ├──────────────────┼─────────────────────────────────────┤
  │ Period           │ 1999-01-22 → 2026-06-16 (~27 years) │
  ├──────────────────┼─────────────────────────────────────┤
  │ Starting Capital │ $10,000                             │
  ├──────────────────┼─────────────────────────────────────┤
  │ Commission       │ 0.2% per trade (round-trip)         │
  ├──────────────────┼─────────────────────────────────────┤
  │ Engine           │ backtesting.py — event-driven, OHLC │
  └──────────────────┴─────────────────────────────────────┘

  ---
  Results

  Metric                        Value
  ─────────────────────────────────────────────
  Return                        +17.28%
  Buy & Hold Return         +546,922.04%
  CAGR                           0.40% / yr
  Sharpe Ratio                   0.052
  Sortino Ratio                  0.080
  Max Drawdown                  -36.27%
  Exposure Time                   7.40%
  # Trades                          112
  Win Rate                         50.0%
  Avg Trade Return                +0.14%
  Profit Factor                    1.155
  Commissions Paid              $6,514
  SQN                              0.208

  ---
  Interpretation

  The strategy beats randomness — but barely

  A Profit Factor of 1.155 and positive expectancy (+0.32% per trade) confirm
  the entry/exit logic has genuine predictive content: winners are larger than
  losers on average. An SQN of 0.208, while low, is above the 0.0 threshold that
  distinguishes a positive-expectancy system from noise.

  It fails against buy & hold — and the reason is instructive

  The core problem is exposure time: 7.4%. The strategy is in the market for
  fewer than 8 trading days in 100, sitting in cash the rest of the time. For an
  asset like NVDA — which compounded at extraordinary rates over this period —
  missing the major bull runs is catastrophic. This is a well-documented failure
  mode of discretionary-style technical strategies applied to secular growth stocks.

  Commission drag is severe

  Over 112 trades, commissions consumed $6,514 on a $10,000 portfolio — more
  than three times the net profit of $1,728. At 0.2% per side, each round-trip
  costs 0.4%, which the average trade (+0.14%) cannot consistently overcome.
  This underlines a critical real-world constraint: any viable strategy must
  produce an average trade return that comfortably exceeds its transaction costs.

  What this does not mean

  Poor performance against NVDA buy & hold is not evidence that the strategy
  logic is broken. Applied to mean-reverting or lower-trending assets, the same
  signal structure may behave very differently. This backtest is an asset-specific
  evaluation, not a verdict on the approach.

  ---
  Known Limitations

  - No parameter optimization — default values are used throughout; no grid
  search or walk-forward optimization is applied. This is intentional for
  Episode 1 of the backtesting thread: establishing an unoptimized baseline
  avoids curve-fitting and gives a more honest starting point.
  - No position sizing — the strategy goes all-in on every signal (self.buy()
  without size). Kelly Criterion (0.025) and fractional sizing are on the roadmap.
  - Single-asset evaluation — results are specific to NVDA's historical
  distribution. Cross-asset validation is required before drawing general conclusions.
  - Survivorship bias — NVDA is among the best-performing large-cap equities
  in history. This dataset is not representative of the average stock.

  ---
  Roadmap

  ┌─────────────────────────────┬───────────────────────────────────────────────┬───────────────┐
  │            Step             │                     Topic                     │    Status     │
  ├─────────────────────────────┼───────────────────────────────────────────────┼───────────────┤
  │ Baseline strategy           │ Multi-indicator entry/exit, fixed sizing      │ This notebook │
  ├─────────────────────────────┼───────────────────────────────────────────────┼───────────────┤
  │ Optimization                │ bt.optimize() grid search + walk-forward      │ Coming soon   │
  ├─────────────────────────────┼───────────────────────────────────────────────┼───────────────┤
  │ Position sizing             │ Kelly / fractional / volatility-scaled sizing │ Planned       │
  ├─────────────────────────────┼───────────────────────────────────────────────┼───────────────┤
  │ Multi-asset validation      │ Test on SPY, QQQ, mean-reverting assets       │ Planned       │
  ├─────────────────────────────┼───────────────────────────────────────────────┼───────────────┤
  │ Integration with ML signals │ Replace rule-based entry with Episode 1 model │ Planned       │
  └─────────────────────────────┴───────────────────────────────────────────────┴───────────────┘

  ---
  Tech Stack

  └─────────────────────────────┴───────────────────────────────────────────────┴───────────────┘

  ---
  Tech Stack

  Python 3.10+
  yfinance          market data
  pandas / numpy    data manipulation
  pandas-ta         technical indicators
  backtesting.py    event-driven backtesting engine

  ---
  Setup

  git clone https://github.com/<your-username>/<repo-name>.git
  cd <repo-name>
  pip install -r requirements.txt
  jupyter notebook Backtesting_py.ipynb

  ---
  Author

  Simone Buzzo — Data Scientist
  LinkedIn · GitHub

  ---
