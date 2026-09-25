# VOO-Replica-2018-2025

<img width="4169" height="2384" alt="Replica_VOO_Audit" src="https://github.com/user-attachments/assets/d71b3369-c805-48a4-a4aa-abee44e6ccee" />

## Overview
This repository contains a quantitative Python model designed to replicate the performance and underlying mechanics of the Vanguard S&P 500 ETF (VOO) over an 8-year time horizon (2018–2025). 

Rather than simply tracking the index, this model reconstructs it from the ground up using historical daily market capitalizations and adjusted close prices. It serves as a framework to analyze index concentration, bucket efficiency, and the exact drivers of performance in a market-cap-weighted portfolio.

## Data Sources
A robust replication requires point-in-time data to avoid survivorship bias. Data for this model was aggregated from two primary sources:
* **Historical Index Constituents:** Point-in-time S&P 500 index memberships were sourced from an open-source GitHub repository fja05680. This ensures the model only "trades" companies that were actually in the S&P 500 on any given historical date.
* **Financial & Fundamental Data:** All daily adjusted close prices and daily market capitalizations for the constituent tickers were sourced via the **[EODHD (End of Day Historical Data) API](https://eodhd.com/)**. 

## Build Process & Architecture
The data pipeline and replication logic were built sequentially in Python:
1. **Constituent Mapping:** Aggregating historical S&P 500 memberships to establish the exact cross-section of 500 stocks active on every trading day.
2. **API Data Ingestion:** Automating calls to the EODHD API to pull end-of-day pricing and market cap data for all current and historical tickers.
3. **Matrix Assembly & Cleansing:** Constructing localized, optimized master matrices (`master_prices_adj.parquet` and `master_mcaps.parquet`). This step handles missing data and forward-fills where appropriate, utilizing custom suppression logic to drop delisted or untradable assets.
4. **Dynamic Weighting & Replication:** Recreating the passive mechanics of the ETF. The replica engine (Steps 5.1 / 5.2) holds shares and rebalances to market-cap weights at the close on quarterly dates and whenever index membership changes, so additions and deletions trade on their effective date. The analytics (Steps 7.2 / 7.3) weight each day's return by prior-day market caps (`t-1`) to prevent look-ahead bias.

## Performance Metrics
To validate the replica against the actual VOO ETF, the model calculates advanced performance metrics:
* **Tracking Error:** Analyzes the annualized standard deviation of the daily return differences between the Replica and VOO, isolating drag caused by rebalancing lag and constituent mismatches.
* **Omega Ratio:** Measures the probability-weighted ratio of gains versus losses for each market cap bucket, utilizing a highly specific 5% annualized threshold. 

## Running It
* **API key:** add your EODHD key in Colab Secrets (key icon in the sidebar) as `EODHD_API_KEY`, or set it as an environment variable. Without a key, the download steps fail but cached parquets in Drive still work.
* **Order:** run the cells top to bottom. Step 7.1 reuses the `df_prices` name, so re-run Step 5.1 before re-running Step 6.3.

## Known Limitations
The model is a close approximation, not an exact replica. Known gaps:

* **Hardcoded drag:** Step 5.1 deducts a fitted 0.6% a year (`HARDCODE_DRAG`) so the replica lands close to VOO. It's a calibration plug that absorbs the gaps below, not a modeled cost, and it compounds to roughly 5% over 8 years. Read the alpha figures with that in mind.
* **Survivorship bias:** 16 former constituents (e.g. UTX, BHGE, TMK, LLL) are missing for 20+ trading days because EODHD no longer carries their data (Step 4.2). The replica doesn't hold them on those days.
* **Proxy prices:** a few acquired companies are priced with the acquirer's history (CA uses AVGO, SCG uses D), so their returns are wrong while they were in the index.
* **Estimated share counts:** the Step 3.4 patches are approximations (sourced from Gemini), each applied as one constant across 2017–2025, overriding EODHD even for tickers still listed. PCG, for example, stays at 520M shares after emerging from bankruptcy with ~2B.
* **Adjusted-close market caps:** caps are built from dividend-adjusted prices, which slightly understate early-period weights for high-yield stocks.
* **Point-in-time data:** share counts are dated at the balance sheet period end rather than the filing date (a small look-ahead), and float factors are hand estimates rather than S&P's published float adjustments.
* **S&P 100 approximation:** Step 5.2 holds the top 100 S&P 500 names by market cap; the real S&P 100 is committee selected.
* **Unvalidated fixes:** the float, share-alignment and rebalancing fixes were tested end to end on synthetic data only, not re-run on real data.
* **Stored outputs:** the outputs saved in the notebook come from the 01.30.2026 run, before those fixes. They have not been regenerated.

## Key Visualizations

<img width="4168" height="2384" alt="Market_Portfolio_Daily_Rebalanced_Pegs" src="https://github.com/user-attachments/assets/92836ca6-d18c-4638-b6dc-f7f59a8c989d" />


<img width="4168" height="2370" alt="Bucket_Efficiency" src="https://github.com/user-attachments/assets/0f8b45f0-df59-4449-85f6-9fb11083f9b3" />


## Tech Stack
* **Language:** Python
* **Data Sourcing:** EODHD API
* **Libraries:** `pandas`, `numpy`, `matplotlib`
* **File Formats:** Jupyter Notebook (`.ipynb`), Parquet
