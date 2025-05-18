# Stock Market Prediction

This repository contains Jupyter notebooks, data, and output files used for quantitative investment research. It focuses on analyzing stock market data (including S&P 500, Russell 3000, and STOXX 600 constituents) and generating trading signals. The project relies heavily on Python notebooks that fetch data from **yfinance** and other sources to perform time-series analysis and build trading strategies.

## Repository structure

- **Data/** – CSV files with ticker lists, spreadsheet credentials, and miscellaneous data.
- **Quantative Investment/** – Core notebooks and their generated output. The `QI Output` subdirectory stores result spreadsheets and thousands of diagnostic plots produced by the notebooks.

## Viewing signals

A live Google Sheet showing current trading signals is available at the following link:
<https://docs.google.com/spreadsheets/d/10HWVyIMgBojLZaUFelmt9oTCO4Z6Oc_cAVCD-3jb_ss/edit?usp=sharing>

The sheet is updated by the notebooks and reflects the most recent signal computations.

## Getting started

1. Clone this repository.
2. Open one of the notebooks in `Quantative Investment/` using Jupyter or VS Code.
3. Run the notebook cells to fetch data and compute signals. Some notebooks expect API keys or credentials (e.g., for Google Sheets) which are stored separately in the `Data` folder.

The generated results will appear in `Quantative Investment/QI Output/`.

## Notes

- The repository contains many large PNG plots illustrating decomposition and trend analysis for each ticker.
- Data sources include free APIs and may be subject to availability or rate limits.
- The notebooks are intended for research and educational purposes.
