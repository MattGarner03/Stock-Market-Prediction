# Quantitative Index Constituent Signals

A **systematic mean‑reversion overlay** for index investing.
Instead of buying the whole benchmark every month, we buy the *cheap* stocks inside the index and trim the *rich* ones — automatically, with auditable code and live signals.

---

## 1  Executive Summary

* **Objective**  Generate long‑only (or 130/30) position sizes for every member of a major equity index, sized by how far each name has drifted from its long‑term trend and short‑term momentum.
* **Use‑case**  Enhance a benchmark‑tracking portfolio or a passive DCA programme with price‑sensitive execution points.
* **Indices covered**  S\&P 500, Russell 3000, STOXX 600 (easily extendable).
* **Outputs**  Daily‑refreshed Google Sheet with trade instructions, plus per‑ticker diagnostics, back‑tests and performance attribution.

> **Live dashboard** (updated by CI after the US close)
> [https://docs.google.com/spreadsheets/d/10HWVyIMgBojLZaUFelmt9oTCO4Z6Oc\_cAVCD-3jb\_ss/edit?usp=sharing](https://docs.google.com/spreadsheets/d/10HWVyIMgBojLZaUFelmt9oTCO4Z6Oc_cAVCD-3jb_ss/edit?usp=sharing)

---

## 2  Signal Framework (high‑level)

1. **Fetch & preprocess** 40 years of adjusted close data via `yfinance` (backup: FMP API).
2. **Decompose** the log‑price into *Trend*, *Seasonal* and *Residual* (additive model, 252‑day period).
3. **Trend momentum**  One‑year rolling slope on the trend component.
4. **Residual Z‑score**  1‑year rolling Z of the residual component.
5. **Position sizing**

   * *Buy* incrementally when Z < −2; *Sell/Trim* when Z > +2.
   * Lot size ∝ |Z| × trend‑strength multiplier (weak ↦ 1×, strong ↦ 2×).
   * Hard cap at ±3 “units” per name to keep the book diversified.
6. **Back‑test** versus buy‑and‑hold and store full PnL, Sharpe, and drawdowns.
7. **Ornstein–Uhlenbeck reversion module** (optional) for short‑cycle mean‑reversion with a momentum veto.

---

## 3  Repository Layout

```
├── Data/                       # static inputs (ticker lists, creds)
│   └── S&P500 Stocks.csv
├── Quantitative Investment/
│   ├──   notebooks/            # research & production notebooks
│   ├──   QI Output/            # autogen spreadsheets & PNGs
│   │      └── plots/
│   └──   Reversion/            # OU‑based signals (sub‑folder)
└── README.md
```

---

## 4  Quick Start

> Tested on Python 3.11 / Ubuntu 22.04

```bash
# 1. Clone
$ git clone https://github.com/your‑org/Stock‑Market‑Prediction.git
$ cd Stock‑Market‑Prediction

# 2. Create env
$ python -m venv .venv && source .venv/bin/activate
$ pip install -r requirements.txt  # or see header of each notebook

# 3. Run signals
$ jupyter lab  # open Quantitative Investment/Run_Signals.ipynb
# …or …
$ python Quantitative\ Investment/run_signals.py  # headless refresh
```

### API keys & secrets

Place the following inside **Data/** (never commit these):

* `google_credentials.json` – service‑account creds with edit rights to the Sheet.
* `API_KEY` – Financial Modelling Prep key (only required for the OU module).

---

## 5  Understanding the Output

| Column                         | Meaning                                                        |
| ------------------------------ | -------------------------------------------------------------- |
| **Trend Status**               | *Upward* / *Downward* / *No momentum* (sign of last 1‑y slope) |
| **Residual Status**            | Oversold (≤−2), Neutral, Overbought (≥+2)                      |
| **Strategy Annual Return**     | CAGR of the dynamic‑position strategy                          |
| **Buy‑and‑Hold Annual Return** | CAGR if you had simply held the stock                          |
| **Last Trade Signal**          | `Buy` / `Sell` with date & size                                |

Diagnostics plots (`QI Output/plots/…png`) show the decomposition, slope and Z‑score timelines for audit.

---

## 6  Trade Workflow (PM view)

1. **06:30 ET**  CI pulls fresh prices → recomputes signals.
2. **06:45 ET**  Sheet updates; you review *Buys* and *Sells* larger than 0.2 units.
3. **09:00 ET**  Execution desk slices the orders via VWAP or adds to the ALGOs/DMA list.
4. **T+1**  Positions feed back into performance and risk reports.

---

## 7  Performance Caveats

* Signals are **un‑smoothed**; expect turnover of ≈ 120 % p.a. on the S\&P 500 universe.
* Transaction costs, slippage and borrow fees are *not* modelled in the notebooks – these must be applied downstream.
* Some ADRs and dual‑listed names exhibit stale prints; data cleaning is ongoing.

---

## 8  Extending / Customising

* **Universe**  Replace `Data/*.csv` with your own constituents; notebooks detect columns automatically.
* **Risk caps**  Edit `max_position` and `get_trend_strength()` to fit your risk budget.
* **Execution style**  The Sheet can publish *parent* notional only, leaving the OMS to slice.

---

## 9  License & Disclaimer

MIT License.
For research and educational use. No warranty of performance is expressed or implied; past results do not guarantee future returns. Use of the signals in live trading is at your own risk.
