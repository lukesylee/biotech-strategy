# Biotech Alpha Platform

A systematic, data-driven trading strategy for biotech equities. The platform combines proprietary signal generation across three analytical dimensions — clinical catalyst probability, M&A acquirability, and pharma pipeline intelligence — into a composite alpha model with event-driven backtesting and portfolio construction.

---

## Motivation

Generic stock screeners and financial analysis tools are not built for biotech. The drivers of biotech stock returns — clinical trial outcomes, regulatory decisions, pipeline gaps at large-cap acquirers — require domain-specific analytical frameworks. This platform is built from the ground up around those drivers.

---

## Architecture

The platform is organised into five layers:

**Layer 0 — Universe construction**
Defines the investable biotech universe using therapy area, modality (small molecule, mAb, ADC, cell therapy, gene therapy), development stage, and cash runway as filters. Data sourced from SEC EDGAR, ClinicalTrials.gov, and financial APIs.

**Layer 1 — Signal generation**
Three independent scoring modules, each producing a numeric signal per ticker:
- `signals/catalyst_dd.py` — Bayesian probability-of-success (PoS) model for upcoming clinical and regulatory catalysts
- `signals/ma_analysis.py` — Acquirability score based on strategic fit, pipeline complementarity, and valuation vs. precedent transactions
- `signals/pharma_intel.py` — Pipeline gap score mapping large-cap pharma unmet needs onto individual biotech assets

**Layer 2 — Alpha model**
Combines the three signals into a composite score using weighted combination, with optional regime conditioning (risk-on / risk-off market state). Located in `alpha_model/combiner.py`.

**Layer 3 — Portfolio construction**
Translates alpha scores into position sizes using a conviction-tier framework. Includes catalyst-event sizing rules, stop-loss logic, and concentration limits by therapy area and modality. Located in `portfolio/construction.py`.

**Layer 4 — Backtesting**
Event-driven backtest engine built around historical catalyst events rather than price bars. Supports walk-forward optimisation for signal weight calibration. Located in `backtest/`.

**Layer 5 — Execution & monitoring**
Catalyst calendar, threshold-based trade alerts, and live P&L dashboard. Located in `execution/`.

---

## Repository Structure

```
biotech-strategy/
│
├── README.md
├── requirements.txt
│
├── universe/                  # Layer 0: stock screener & data pipeline
│   ├── screener.py
│   └── data/
│
├── signals/                   # Layer 1: scoring functions
│   ├── catalyst_dd.py
│   ├── ma_analysis.py
│   └── pharma_intel.py
│
├── alpha_model/               # Layer 2: signal combination
│   └── combiner.py
│
├── portfolio/                 # Layer 3: position sizing & risk rules
│   └── construction.py
│
├── backtest/                  # Layer 4: event-driven backtest engine
│   ├── engine.py
│   └── events/
│
├── execution/                 # Layer 5: monitoring & alerts
│   └── dashboard.py
│
└── notebooks/                 # Jupyter notebooks for analysis & exploration
```

---

## Data Sources

| Source | Use | Access |
|---|---|---|
| SEC EDGAR | Financials, pipeline data from filings | Free API |
| ClinicalTrials.gov | Trial phase, indication, status | Free API |
| FDA CDER/CBER | PDUFA dates, approvals, designations | Free |
| yFinance | Price, market cap, shares outstanding | Free (Python) |
| Polygon.io | Historical and real-time market data | Paid |
| Evaluate Pharma / Citeline | Enterprise pipeline database | Enterprise |

---

## Tech Stack

- **Language**: Python 3.11+
- **Data storage**: SQLite (development), PostgreSQL (production)
- **Analysis**: pandas, numpy, scipy
- **Backtesting**: custom event-driven engine
- **Visualisation**: Streamlit (dashboard), matplotlib / plotly (analysis)
- **Version control**: Git / GitHub

---

## Development Roadmap

- [x] Project architecture design
- [x] Analytical frameworks: catalyst DD, M&A analysis, pharma intelligence
- [ ] Universe pipeline (Layer 0)
- [ ] Signal scoring functions (Layer 1)
- [ ] Historical catalyst event database
- [ ] Alpha model combiner (Layer 2)
- [ ] Portfolio construction rules (Layer 3)
- [ ] Event-driven backtest engine (Layer 4)
- [ ] Walk-forward optimisation
- [ ] Execution dashboard (Layer 5)

---

## Status

Early development. Analytical frameworks complete. Data pipeline and scoring modules in progress.

---

*This repository is private. All analytical frameworks, scoring models, and strategy logic are proprietary.*
