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
4. **Dynamic Weighting & Replication:** Recreating the passive mechanics of the ETF. The model dynamically recalculates portfolio weights using lagged market caps (`t-1`) to completely prevent look-ahead bias, computing daily portfolio returns as the sum product of component weights and daily percent changes.

## Performance Metrics
To validate the replica against the actual VOO ETF, the model calculates advanced performance metrics:
* **Tracking Error:** Analyzes the annualized standard deviation of the daily return differences between the Replica and VOO, isolating drag caused by rebalancing lag and constituent mismatches.
* **Omega Ratio:** Measures the probability-weighted ratio of gains versus losses for each market cap bucket, utilizing a highly specific 5% annualized threshold. 

## Key Visualizations
The model outputs production-grade visualizations to dissect index mechanics:
* **Efficiency vs. Performance Profile (Bucket Efficiency):** A comprehensive dual-axis chart comparing the total return of 20 distinct 5% market-cap segments against their Omega Ratios. This chart accurately maps the mega-cap concentration by overlaying the exact number of underlying stocks required to fill each segment, contrasting the top-heavy drivers against the long tail of the index.

<img width="4168" height="2384" alt="Market_Portfolio_Daily_Rebalanced_Pegs" src="https://github.com/user-attachments/assets/92836ca6-d18c-4638-b6dc-f7f59a8c989d" />


## Tech Stack
* **Language:** Python
* **Data Sourcing:** EODHD API
* **Libraries:** `pandas`, `numpy`, `matplotlib`
* **File Formats:** Jupyter Notebook (`.ipynb`), Parquet
