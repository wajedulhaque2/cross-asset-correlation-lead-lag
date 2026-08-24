# Cross-Asset Correlation & Lead–Lag Explorer

A reusable Python research notebook for testing contemporaneous correlation, rolling dependence, lead–lag structure and bidirectional predictive information between two Yahoo Finance tickers.

The default case studies **Brent crude futures (`BZ=F`) vs Shell plc (`SHEL.L`)**, but the notebook is ticker-agnostic: edit the controls or use the interactive research panel to compare stocks, commodities, ETFs, indices or other supported instruments.

## What the project does

The notebook builds the analysis in layers rather than jumping from a correlation coefficient to a causal story:

1. downloads adjusted market data and audits trading-calendar mismatches;
2. aligns both assets on common observed dates without forward-filling;
3. converts prices to daily log returns;
4. measures Pearson and Spearman contemporaneous correlation;
5. tracks 20-, 60- and 120-observation rolling correlations and historical percentiles;
6. scans lead–lag correlations over a symmetric lag range using the **same observation count at every lag**;
7. applies Bonferroni family-wise multiple-testing correction;
8. estimates selected-lag uncertainty with a moving-block bootstrap and block-size sensitivity;
9. checks return stationarity with the Augmented Dickey–Fuller test;
10. tests Granger predictive information in both directions with corrected p-values;
11. repeats the lag search through rolling windows to measure stability;
12. generates a compact evidence table and a dynamic conclusion.

## Interactive research panel

After running the notebook through the research-engine cells, an `ipywidgets` panel lets you enter two tickers and a date range and generate the compact research dashboard from the same methodology.

The manual controls remain the authoritative configuration for a full **Run All**:

```python
ASSET_A = "BZ=F"
ASSET_B = "SHEL.L"

START_DATE = "2015-01-01"
END_DATE = "2026-08-24"

ROLLING_WINDOWS = [20, 60, 120]
MAX_LAG = 20

BOOTSTRAP_SAMPLES = 1000
BOOTSTRAP_BLOCK_SIZE = 20
BOOTSTRAP_BLOCK_SIZES = [10, 20, 40]

GRANGER_MAX_LAG = 10

ROLLING_LAG_WINDOW = 252
ROLLING_LAG_STEP = 21
```

Example Yahoo Finance tickers include `AAPL`, `MSFT`, `SHEL.L`, `RR.L`, `BZ=F`, `CL=F`, `GC=F`, `SPY` and `^FTSE`.

## Default Brent–Shell findings

The supplied 2015–August 2026 run produced the following reference results:

| Metric | Result |
|---|---:|
| Common aligned price observations | 2,875 |
| Daily log-return observations | 2,874 |
| Pearson return correlation | 0.481 |
| Spearman return correlation | 0.455 |
| Latest 120-observation correlation | 0.713 |
| Latest 120-observation historical percentile | 98.8% |
| Strongest equal-sample non-zero lag | +6 observations |
| Lagged correlation | -0.099 |
| 95% moving-block bootstrap interval | [-0.175, -0.017] |
| +6 strongest in equal-sample rolling windows | 12.8% |
| Lag magnitude exceeds same-day magnitude in rolling windows | 0.0% |

The principal result is **not** that Brent mechanically leads Shell by six days. The data support a much stronger contemporaneous relationship, plus weaker delayed structure that is statistically detectable in the full sample but not stable enough to describe as a fixed market lag.

Bonferroni-corrected Granger tests were strongly asymmetric: `BZ=F → SHEL.L` specifications with maximum lags 6–10 survived correction, while only the Shell-to-Brent lag-7 specification survived and only narrowly.

## Default-case charts

### Rolling correlation

![Rolling correlation](assets/default_rolling_correlation.png)

### Equal-sample lead–lag scan

![Equal-sample lead-lag scan](assets/default_equal_sample_lead_lag.png)

### Moving-block bootstrap

![Moving-block bootstrap](assets/default_bootstrap.png)

### Bidirectional Granger tests

![Bidirectional Granger tests](assets/default_granger.png)

### Rolling lag stability

![Rolling lag stability](assets/default_rolling_lag_stability.png)

## Methodology notes

For aligned daily log returns $r_t^A$ and $r_t^B$, the lag convention is

$$
\rho(k)=\operatorname{Corr}(r_t^A,r_{t+k}^B).
$$

Positive $k$ means Asset A leads Asset B; negative $k$ means Asset B leads Asset A. The sign of the correlation is separate from the sign of the lag.

Every candidate lag is evaluated on the same central sample of $n-2M$ observations, where $M$ is the maximum searched lag. This prevents extreme lags from winning simply because smaller samples make their estimated correlations noisier.

The moving-block bootstrap preserves short contiguous runs of the selected lagged pair, providing a dependence-aware uncertainty estimate. Its interval is conditional on the selected lag, while Bonferroni correction separately addresses the search across multiple candidate lags.

Granger testing compares an autoregression of the target asset with an unrestricted model that also includes lagged values of the candidate predictor. A significant test means incremental predictive information under that specification; it does **not** establish economic causality.

## Installation

Clone the repository and install the dependencies:

```bash
pip install -r requirements.txt
```

Then open:

```text
cross_asset_correlation_lead_lag.ipynb
```

in JupyterLab, Jupyter Notebook or VS Code and run the notebook from top to bottom.

## Repository structure

```text
.
├── cross_asset_correlation_lead_lag.ipynb
├── README.md
├── requirements.txt
└── assets/
    ├── default_normalised_prices.png
    ├── default_rolling_correlation.png
    ├── default_equal_sample_lead_lag.png
    ├── default_bootstrap.png
    ├── default_granger.png
    └── default_rolling_lag_stability.png
```

## Limitations

Daily timing cannot fully resolve differences in market hours, settlement conventions or time zones. Yahoo Finance is suitable for research and portfolio demonstration but is not institutional market data. Continuous futures can embed contract-roll mechanics. Correlation and Granger predictive information are not causal identification. Results depend on the sample, frequency, lag range and rolling-window design, and a historical relationship may not persist out of sample.

The notebook intentionally reports these limitations alongside the statistics so that a small p-value is never presented as sufficient evidence of a stable or tradable effect.

## Research use

This repository is an educational and quantitative-research project. It is not investment advice.
