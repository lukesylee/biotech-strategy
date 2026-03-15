# Biotech Screener — Data Pipeline

## Setup (one-time)
```bash
pip install requests pandas yfinance openpyxl tqdm
```

## Run Order

### Step 1 — Pull full SIC 2836 universe + financial data
```bash
python build_universe.py
```
**What it does:**
- Pulls all SIC 2836 companies from SEC EDGAR (company_tickers_exchange.json)
- Filters to NASDAQ / NYSE / NYSE American listings
- Enriches each ticker with yFinance: market cap, cash, burn rate, 52W high/low
- Filters out companies below $50M market cap
- Writes → `Biotech_Stock_Screener_FULL.xlsx` (financial columns populated)
- Writes → `biotech_universe_raw.csv` (backup)

**Runtime:** ~10–20 minutes depending on universe size (~500–700 companies × 0.5s each)

---

### Step 2 — Enrich with pipeline / clinical trial data
```bash
python build_pipeline.py
```
**What it does:**
- Reads `biotech_universe_raw.csv`
- Queries ClinicalTrials.gov API for each company's trials
- Extracts: Lead Asset, Dev Stage, Therapy Area, Indication, Partners
- Classifies Therapy Area from indication text (keyword matching)
- Writes → `pipeline_db.csv` (one row per trial)
- Writes → `biotech_universe_enriched.csv`
- Updates `Biotech_Stock_Screener_FULL.xlsx` 🧬 Pipeline DB sheet

**Runtime:** ~5–10 minutes (~500 companies × 0.2s each)

---

## Refresh Schedule (recommended)

| Data Type          | Refresh Cadence | Script             |
|--------------------|-----------------|---------------------|
| Market cap / price | Weekly          | build_universe.py   |
| Cash / burn rate   | Quarterly       | build_universe.py   |
| Pipeline stage     | Monthly         | build_pipeline.py   |
| Catalyst dates     | Monthly (manual)| Update 📅 Catalyst Calendar tab manually |

---

## Output Files

| File                                  | Description                        |
|---------------------------------------|------------------------------------|
| `Biotech_Stock_Screener_FULL.xlsx`    | Main screener (all tabs populated) |
| `biotech_universe_raw.csv`            | Raw ticker + financial data        |
| `pipeline_db.csv`                     | All clinical trials per company    |
| `biotech_universe_enriched.csv`       | Universe with pipeline fields added|

---

## Known Limitations

1. **Modality classification** from ClinicalTrials.gov is approximate:
   - BIOLOGICAL interventions → labelled "Monoclonal Ab (mAb)" (covers all biologics)
   - GENETIC interventions → "Gene Therapy / Gene Editing"
   - DRUG interventions → "Small Molecule" (includes some peptides)
   - Correct modality for each company should be verified manually for precision screening

2. **Therapy area** is keyword-matched from indication text. Review "Other" category — 
   some companies may be miscategorised.

3. **Lead asset** = highest-phase programme in ClinicalTrials.gov. 
   Some companies have multiple Phase 3 programmes — add these manually to Pipeline DB.

4. **Revenue / 10-K data** (Revenue TTM, revenue growth) not pulled by yFinance reliably 
   enough for commercial-stage companies. Add manually from SEC EDGAR 10-K for 
   revenue-generating companies.

5. **Cash burn from yFinance** uses annual operating cash flow ÷ 4 as a quarterly proxy. 
   For precision, update from the actual quarterly cash flow statement in 10-Q.

---

## Data Sources

| Source                  | URL                                               | Access  |
|-------------------------|---------------------------------------------------|---------|
| SEC EDGAR Company List  | https://www.sec.gov/files/company_tickers_exchange.json | Free |
| SEC EDGAR Submissions   | https://data.sec.gov/submissions/CIK{CIK}.json   | Free    |
| ClinicalTrials.gov API  | https://clinicaltrials.gov/api/v2/studies        | Free    |
| yFinance                | Python library (Yahoo Finance)                   | Free    |
| FDA PDUFA Calendar      | https://www.fda.gov/patients/learn-about-drug-and-device-approvals/pdufa-performance-report | Free (manual) |
