# Cross-Asset Correlation and Lead-Lag Research Lab

A Python/Jupyter quantitative research project for measuring contemporaneous dependence, testing lead-lag structure, discovering candidate equity relationships, evaluating them out of sample and monitoring current signals.

The project is designed to separate four questions that are often conflated:

1. Are two assets related on the same day?
2. Is there statistically detectable delayed structure?
3. Does that delayed structure survive walk-forward testing after costs?
4. Is a historically interesting relationship active now?

The notebook uses Yahoo Finance market data and an interactive `ipywidgets` Strategy Lab.

## Project workflow

The Strategy Lab contains five tabs.

### Research Pair

Runs the full statistical investigation for two selected Yahoo Finance tickers:

- trading-calendar alignment without forward-filling
- daily log returns
- Pearson and Spearman dependence
- rolling correlation
- equal-sample lead-lag scanning
- multiple-testing correction
- moving-block bootstrap
- Augmented Dickey-Fuller stationarity checks
- bidirectional Granger predictive-information tests
- rolling lag-stability analysis

### Backtest Pair

Tests one specified pair with a walk-forward model.

At each rebalance, leader, follower, lag horizon and forward-response beta are re-estimated using only the formation window. A completed leader return is observed before the follower position begins at the next available adjusted open. Transaction costs are applied when exposure changes.

This tab is intended for focused pair research. It is separate from the universe-level discovery test.

### Discover

Builds a current liquid-equity universe through Yahoo Finance screening, downloads historical data, applies a cheap correlation funnel and performs deeper lead-lag analysis on the strongest candidates.

Discover ranks relationships using:

- same-day correlation
- lag correlation
- sign stability
- lag-win share
- corrected lag evidence
- scanner-wide false-discovery-rate evidence
- a transparent Opportunity Score

Same-issuer and obvious share-class pairs can be excluded.

### Scanner Backtest

Tests the discovery process itself rather than selecting a pair with hindsight.

At historical rebalance date $T$, candidate relationships are rebuilt using only trailing observations:

<br>

```math
\Large \mathcal{C}_T = \mathrm{Scanner}(r_t : T-W \le t < T)
```

<br>

The top relationships are selected under the chosen evidence and ranking rule, then traded during the following out-of-sample block.

This is the main strategy-level evidence because the candidate set itself is reconstructed through time.

### Investment Monitor

Combines the current Discover evidence with the latest completed market observations.

The monitor refreshes recent prices for discovered assets, re-estimates current relationship direction, lag sign and forward beta, standardises the latest leader return and classifies relationships into four research tiers.

- **Tier A:** active signal with strong positive scanner-backtest support
- **Tier B:** active signal with limited historical support
- **Tier C:** stable relationship waiting for a qualifying leader shock
- **Tier D:** changed or invalidated current relationship

The monitor also reports the shock-linked model response component:

<br>

```math
\Large \widehat{R}^{F,\mathrm{shock}}_{t,t+h}
=
\hat{\beta}r^L_t.
```

<br>

This is an economic-magnitude diagnostic, not a guaranteed follower return.

## Statistical design

### Log returns

For adjusted price $P_t$:

<br>

```math
\Large r_t
=
\ln(\frac{P_t}{P_{t-1}}).
```

<br>

### Lag convention

For Asset A and Asset B:

<br>

```math
\Large \rho(k)
=
\mathrm{Corr}(r_t^A,r_{t+k}^B).
```

<br>

Positive $k$ means Asset A leads Asset B. Negative $k$ means Asset B leads Asset A. The sign of the lag is separate from the sign of the correlation.

### Equal-sample lag scan

All candidate lags are evaluated on the same central sample. This prevents extreme lags from benefiting from a different observation count.

### Multiple testing

The project uses two distinct corrections:

1. Bonferroni correction across lags within a pair.
2. Benjamini-Hochberg false-discovery-rate correction across deep-stage candidate pairs in the discovery scan.

The scanner-wide FDR is conditional on the screened candidate set rather than a perfect universe-wide correction.

### Moving-block bootstrap

The bootstrap resamples contiguous blocks rather than individual observations so short-run dependence is retained when estimating uncertainty around the selected lag correlation.

### Granger tests

Granger tests are run in both directions. Statistical significance is interpreted as incremental predictive information under the tested autoregressive specification, not structural causality.

## Reference research findings

The original Brent crude futures (`BZ=F`) and Shell (`SHEL.L`) case study showed a strong contemporaneous relationship and much weaker delayed structure.

Reference results included:

| Metric | Result |
|---|---:|
| Pearson daily log-return correlation | 0.481 |
| Spearman correlation | 0.455 |
| Strongest equal-sample non-zero lag | Brent leads by 6 observations |
| Equal-sample lag correlation | about -0.099 |
| 95% moving-block bootstrap interval | about [-0.175, -0.017] |
| Fixed +6 lag strongest in rolling windows | about 13% |
| Lag magnitude exceeding same-day correlation in rolling windows | 0% |

The important conclusion is that statistical detectability of a lag is not the same as a stable fixed market delay. Same-day dependence dominates the relationship.

## Reference scanner-backtest snapshot

A later 200-security Yahoo Finance screen retained 187 securities with sufficient history. Using the frozen primary research specification:

- 504-observation formation window
- 63-observation rebalance interval
- top 3 relationships
- Opportunity Score ranking
- scanner FDR gate <= 10%
- absolute leader shock threshold >= 1.5
- 10 bps one-way transaction cost
- next-open follower execution

the walk-forward scanner produced the following reference result:

| Metric | Reference result |
|---|---:|
| Net total return | 13.98% |
| Gross total return | 20.86% |
| CAGR | 2.87% |
| Annualised volatility | 5.23% |
| Sharpe ratio | 0.57 |
| Maximum drawdown | -6.46% |
| Trades | 52 |
| Win rate | 55.8% |
| Active rebalance periods | 6 of 19 |
| Unique relationships selected | 7 |
| Approximate break-even one-way cost | 35.6 bps |

The aggregate result is concentrated rather than broad. `MFG -> SMFG` contributed most of the positive historical result, with 13 trades and approximately +14.67% net scanner contribution. The project therefore does not present seven independent alpha sources.

These figures are a research snapshot, not a promise that a fresh Yahoo Finance universe will reproduce the same result.

## Investment-monitor example

After refreshing market data through 2026-08-24, the monitor identified two Tier B relationships:

- `V -> MA`: Mastercard downside bias, one-observation horizon, leader z-score about +2.11, model response component about -0.29%
- `DUK -> SO`: Southern Company downside bias, four-observation horizon, leader z-score about +1.62, model response component about -0.30%

Neither relationship had established scanner-backtest support in that snapshot, so neither qualified for Tier A.

`MFG -> SMFG` remained Tier C because its historical support was positive but the latest leader shock was too small.

This distinction is intentional: current statistical timing and historical trading support are separate pieces of evidence.

## Why the project is conservative

The notebook intentionally avoids several common research shortcuts:

- no forward-filling across mismatched trading calendars
- no unequal lag sample sizes
- no uncorrected multiple-lag claims
- no causal interpretation of Granger tests
- no same-interval follower execution after observing the leader close
- no zero-cost strategy reporting
- no fixed leader/follower assumption through the entire backtest
- no forced scanner position when no relationship passes the selected gate
- no promotion of a current signal to Tier A without historical scanner support

## Important limitations

The results should be interpreted with several constraints in mind.

### Current-universe survivorship bias

The scanner uses a universe screened from Yahoo Finance today. Historical walk-forward selection is valid conditional on that universe, but the security list is not a point-in-time historical constituent database. Delisted and failed securities may therefore be absent.

### Daily market-session timing

Daily bars cannot fully resolve differences in market hours, time zones, ADR trading, futures settlement conventions or information arrival during overlapping sessions.

### Yahoo Finance

Yahoo Finance is appropriate for research and portfolio demonstration, but it is not institutional market data.

### Sparse strategy evidence

The strongest scanner specification is supported by a modest number of trades and active periods. Performance is concentrated in a small number of relationships and recent regimes.

### Short implementation

A downside follower bias is a statistical direction, not an assumption that the security can or should be shorted. Real implementation would require borrow availability, financing costs, position sizing and portfolio risk controls.

### Statistical significance

Small p-values do not guarantee economic significance, stability or profitability. Many current models have low forward $R^2$, which is why the monitor reports both evidence and estimated response magnitude.

## Repository structure

```text
.
├── cross_asset_correlation_lead_lag.ipynb
├── README.md
├── requirements.txt
├── .gitignore
└── docs/
    └── FORMULA_GUIDE.md
```

## Installation

Create a Python environment and install:

```bash
pip install -r requirements.txt
```

Then open:

```text
cross_asset_correlation_lead_lag.ipynb
```

in JupyterLab, Jupyter Notebook or VS Code.

Run the notebook from top to bottom. The network-heavy Discover and Scanner Backtest actions are button-driven inside the Strategy Lab.

## Main dependencies

- pandas
- numpy
- matplotlib
- yfinance
- scipy
- statsmodels
- ipywidgets
- jupyterlab

## Reproducibility and security

The repository does not require API credentials for Yahoo Finance.

Local environment files are ignored through `.gitignore`:

```text
.env
.env.*
```

An `.env.example` file may be committed when it contains placeholders only. Real credentials should remain outside version control.

## Research use

This repository is an educational quantitative-research project. It is not investment advice.
