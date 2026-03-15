# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A systematic, data-driven trading platform for biotech equities. Combines three independent alpha signals (clinical catalyst probability, M&A acquirability, pharma pipeline intelligence) into a composite score, then feeds into event-driven portfolio construction and backtesting.

**Status:** Early development. Architecture is designed; source files are not yet written.

## Tech Stack

- Python 3.11+
- pandas, numpy, scipy for analysis
- SQLite (dev) / PostgreSQL (prod) for storage
- Streamlit for the live dashboard
- Custom event-driven backtest engine (no third-party backtesting framework)

## Development Setup

```bash
pip install -r requirements.txt
```

Data API keys required (set as environment variables):
- `POLYGON_API_KEY` — historical and real-time market data (paid)
- `SEC_EDGAR` — free, no key needed
- `CLINICALTRIALS_API` — free, no key needed

## Architecture: Five-Layer Pipeline

Data flows top-to-bottom through these layers:

| Layer | Directory | Purpose |
|---|---|---|
| 0 | `universe/` | Build the investable biotech universe (screener + data pipeline) |
| 1 | `signals/` | Three independent scoring modules (see below) |
| 2 | `alpha_model/` | Combine signals into a composite score (`combiner.py`) |
| 3 | `portfolio/` | Translate alpha scores into position sizes (`construction.py`) |
| 4 | `backtest/` | Event-driven backtest engine (events, not price bars) |
| 5 | `execution/` | Catalyst calendar, trade alerts, live P&L dashboard |

### Layer 1 — Signal Modules (`signals/`)

Each module takes a ticker and returns a numeric signal:

- `catalyst_dd.py` — Bayesian probability-of-success (PoS) for upcoming clinical/regulatory catalysts
- `ma_analysis.py` — Acquirability score (strategic fit, pipeline complementarity, valuation vs. precedents)
- `pharma_intel.py` — Pipeline gap score (maps large-cap pharma unmet needs onto biotech assets)

### Layer 4 — Backtest Design

The backtest engine in `backtest/engine.py` is event-driven around **historical catalyst events** (trial readouts, PDUFA dates, approvals), not price bars. Walk-forward optimisation is used to calibrate signal weights.

## Key Domain Concepts

- **Universe filters**: therapy area, modality (small molecule, mAb, ADC, cell therapy, gene therapy), development stage, cash runway
- **Position sizing**: conviction-tier framework with catalyst-event sizing rules, stop-loss logic, concentration limits by therapy area and modality
- **Regime conditioning**: alpha model supports optional risk-on / risk-off market state overlay in `combiner.py`

## Data Sources

| Source | Use |
|---|---|
| SEC EDGAR | Financials, pipeline data from filings |
| ClinicalTrials.gov | Trial phase, indication, status |
| FDA CDER/CBER | PDUFA dates, approvals, designations |
| yFinance | Price, market cap, shares outstanding |
| Polygon.io | Historical and real-time market data (paid) |
| Evaluate Pharma / Citeline | Enterprise pipeline database (enterprise license) |
