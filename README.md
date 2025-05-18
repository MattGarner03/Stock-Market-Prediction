# Quantitative Index Constituent Signals

A **systematic mean‑reversion overlay** for index investing.
Instead of Dollar Cost Averaging into the whole index, I buy the relative value inside the index at a stock elvel and trim the relatively expensive ones with rebalancing each day after US close.


---

## 1  Executive Summary

* **Objective**  Generate long‑only (or 130/30) position sizes for every member of a major equity index, sized by how far each name has drifted from its long‑term trend and short‑term momentum.
* **Use‑case**  Enhance a benchmark‑tracking portfolio or a passive DCA programme with price‑sensitive execution points.
* **Indices covered**  S\&P 500, Russell 3000, STOXX 600 (easily extendable).
* **Outputs**  Daily‑refreshed Google Sheet with trade instructions, plus per‑ticker diagnostics, back‑tests and performance attribution.

> **Live Google Sheet** (updated by CI after the US close)
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

* Note: Strategy Sharpe Ratio and Strategy Annual Return Calculations are at the moment misleading as they account for buying incramentally with increasing size as Z falls from -2 --> -3 --> -4 and vice versa on the sells. Work to be done to improve the logistics
* The other known issue is the obvious one of survivorship bias. This notebook aims to DCA efficiently into the S&P at a single stock level with the ability to go short the relatively expensive names and long the relatively cheap names, however once a name falls out of the s&p it falls out of the strategy so the simulation overstates profitability by ignoring names falling out of the index. Strategy decay attributed to this factor is a clear caveat to running this without any discretion in the decision making process.

Diagnostics plots (`QI Output/plots/…png`) show the decomposition, slope and Z‑score timelines for debugging.

---

## 6  Execution Workflow 

1. **21:00**  CI pulls fresh prices → recomputes signals.
2. **21:05**  Sheet updates; you review *Buys* and *Sells* larger than 0.2 units.
3. **21:10**  Limit Orders Placed at something reasonable like yesterdays VWAP and limit sells placed at the reversion price target if they exhibit reversion behaviour.
Alternatively you could run the signals at 8:45-8:50 pm UK time and then buy/ sell aggressively into the close once the signals are returned.

---

## 7  Performance Caveats

* Signals are **un‑smoothed**; expect turnover of ≈ 120 % p.a. on the S\&P 500 universe.
* Transaction costs, slippage and borrow fees are *not* modelled in the notebooks – these must be applied downstream plus further portfolio optimiser must be used to prevent sector concentration. Work has been done to incorporate the information needed for industry/ sector level mapping. Right now the best POA is scheduling ChatGPT to run on the google sheet at 10pm and run the portfolio optimiser with a strict prompt to maintain sector neutrality. 
* Some ADRs and dual‑listed names exhibit stale prints; data cleaning is ongoing, for example ASML here is the main killer as I want to trade the US listing but the signal runs with EU listing close. Future work to be done to remove the EU listing and run the signal on the US listing

