# Formula and Methodology Guide

This guide summarises the core equations, inference rules and trading conventions used by `cross_asset_correlation_lead_lag.ipynb`.

## 1. Daily log returns

For adjusted price $P_t$:

<br>

```math
r_t
=
\ln(\frac{P_t}{P_{t-1}}).
```

<br>

Log returns are additive across adjacent intervals and are used throughout the statistical relationship analysis.

## 2. Same-day dependence

Pearson correlation measures linear dependence:

<br>

```math
\rho_{A,B}
=
\frac{
\mathrm{Cov}(r^A,r^B)
}{
\sigma_A\sigma_B
}.
```

<br>

Spearman correlation is also reported as a rank-based robustness measure.

A strong same-day correlation does not imply a tradable delay.

## 3. Rolling correlation

For rolling window $W$:

<br>

```math
\rho_t^{(W)}
=
\mathrm{Corr}
(
r^A_{t-W+1:t},
r^B_{t-W+1:t}
).
```

<br>

Current rolling dependence is also compared with its historical percentile.

## 4. Lead-lag convention

The lag statistic is

<br>

```math
\rho(k)
=
\mathrm{Corr}(r_t^A,r_{t+k}^B).
```

<br>

Interpretation:

- $k>0$: Asset A leads Asset B
- $k<0$: Asset B leads Asset A
- $k=0$: same-day relationship

The sign of $k$ identifies timing direction. The sign of $\rho(k)$ identifies whether the delayed association is same-direction or inverse.

## 5. Equal-sample lag scan

If the maximum searched absolute lag is $M$, every lag is evaluated using the same central sample rather than allowing observation count to vary with $k$.

Conceptually:

<br>

```math
n_{\mathrm{effective}}
=
n-2M.
```

<br>

This prevents extreme lags from winning because of noisier estimates from smaller samples.

## 6. Within-pair multiple testing

If $m$ non-zero lags are tested, the Bonferroni-adjusted p-value is

<br>

```math
p_i^{\mathrm{Bonf}}
=
\min(1,mp_i).
```

<br>

This controls family-wise error across the lag search for one pair.

## 7. Moving-block bootstrap

Contiguous return blocks are sampled with replacement until a bootstrap series of the required length is created.

The lagged correlation is recalculated for each bootstrap draw:

<br>

```math
\hat{\rho}^{*(b)},
\qquad
b=1,\ldots,B.
```

<br>

The empirical percentile interval is then used as a dependence-aware uncertainty estimate.

Block-size sensitivity is reported because the result can depend on the assumed dependence horizon.

## 8. Stationarity

The Augmented Dickey-Fuller test is used before interpreting autoregressive Granger specifications.

The null hypothesis is that the series contains a unit root.

The notebook applies the test to returns rather than price levels.

## 9. Granger predictive information

For target $Y_t$, the restricted model uses lagged values of $Y$:

<br>

```math
Y_t
=
\alpha
+
\sum_{i=1}^{p}\phi_iY_{t-i}
+
\varepsilon_t.
```

<br>

The unrestricted model also includes lagged values of $X$:

<br>

```math
Y_t
=
\alpha
+
\sum_{i=1}^{p}\phi_iY_{t-i}
+
\sum_{i=1}^{p}\gamma_iX_{t-i}
+
u_t.
```

<br>

The null hypothesis is

<br>

```math
\gamma_1=\cdots=\gamma_p=0.
```

<br>

Rejection means lagged $X$ contains incremental predictive information for $Y$ under that specification. It does not establish structural economic causality.

## 10. Rolling lag stability

The lag scan is repeated through rolling windows.

Two important diagnostics are:

<br>

```math
\Pr(\hat{k}_t=k^*)
```

<br>

for how often a selected full-sample lag remains the strongest lag, and

<br>

```math
\Pr
(
|\rho_t(k^*)|
>
|\rho_t(0)|
)
```

<br>

for how often delayed dependence dominates same-day dependence.

This prevents one full-sample lag estimate from being treated as permanent.

## 11. Discovery funnel

For a current screened universe, a cheap same-day correlation stage reduces the number of pairs entering deeper lag analysis.

Only sufficiently correlated pairs are evaluated with the full lag procedure.

This is a computational funnel, not statistical evidence by itself.

## 12. Scanner-wide false-discovery rate

Deep-stage pair p-values are corrected using the Benjamini-Hochberg procedure.

If ordered p-values satisfy

<br>

```math
p_{(1)}
\le
p_{(2)}
\le
\cdots
\le
p_{(m)},
```

<br>

the procedure compares them with

<br>

```math
\frac{i}{m}q
```

<br>

for target FDR level $q$.

The project distinguishes:

- validated: scanner FDR <= 5%
- exploratory: 5% < scanner FDR <= 10%
- unvalidated: scanner FDR > 10%

This correction is conditional on the screened deep-stage candidate set.

## 13. Opportunity Score

The discovery ranking score combines relationship strength and stability:

<br>

```math
S_{ij}
=
100
(
w_CC_{ij}
+
w_LL_{ij}
+
w_SS^{\mathrm{sign}}_{ij}
+
w_WS^{\mathrm{win}}_{ij}
).
```

<br>

The score is a relationship-ranking device. It is not an estimated alpha or expected return.

## 14. Walk-forward model estimation

At rebalance date $T$, model parameters are estimated only from a trailing formation window:

<br>

```math
\mathcal{D}_T = (r_t : T-W \le t < T)
```

<br>

Leader, follower, horizon and forward beta are re-estimated at every rebalance.

The next test block is never used to fit its own model.

## 15. Leader shock

For current leader return $r_t^L$, trailing mean $\mu_L$ and standard deviation $\sigma_L$:

<br>

```math
z_t
=
\frac{r_t^L-\mu_L}{\sigma_L}.
```

<br>

The primary research specification uses

<br>

```math
|z_t|\ge1.5.
```

<br>

The threshold was frozen after a small, pre-specified sensitivity set rather than repeatedly optimised.

## 16. Forward-response regression

For estimated leader $L$, follower $F$ and horizon $h$:

<br>

```math
R^F_{t,t+h}
=
\alpha
+
\beta r_t^L
+
\varepsilon_t.
```

<br>

The current signal requires the beta sign to agree with the discovered lag relationship.

## 17. Shock-linked model response

The monitor reports:

<br>

```math
\widehat{R}^{F,\mathrm{shock}}_{t,t+h}
=
\hat{\beta}r_t^L.
```

<br>

This is the component of the fitted forward-return model associated with the current leader move.

It excludes the intercept and should not be interpreted as a guaranteed follower return.

## 18. Execution convention

The leader shock is observed after a completed daily close.

Follower exposure begins at the next available adjusted open.

For a one-observation holding period:

<br>

```math
R^{F}_{t+1}
=
\frac{
P^{F,\mathrm{close}}_{t+1}
}{
P^{F,\mathrm{open}}_{t+1}
}
-1.
```

<br>

For a longer horizon, the position begins at the next open and remains exposed until the designated future close.

This prevents look-ahead execution inside the interval that generated the leader signal.

## 19. Transaction costs

A one-way transaction-cost rate $c$ is applied when exposure changes.

For portfolio turnover $\tau_t$:

<br>

```math
\mathrm{Cost}_t
=
c\tau_t.
```

<br>

Net return is

<br>

```math
R_t^{\mathrm{net}}
=
R_t^{\mathrm{gross}}
-
\mathrm{Cost}_t.
```

<br>

## 20. Scanner portfolio

At historical rebalance date $T$:

<br>

```math
\mathcal{C}_T = \mathrm{Scanner}(r_t : T-W \le t < T)
```

<br>

The selected set is

<br>

```math
\mathcal{S}_T
=
\mathrm{TopN}(\mathcal{C}_T).
```

<br>

With equal relationship sleeves:

<br>

```math
w_{j,T}
=
\frac{1}{N_T}.
```

<br>

Portfolio return is

<br>

```math
R_t^P
=
\sum_{j=1}^{N_T}
w_{j,T}R_{j,t}.
```

<br>

If no relationship passes the configured gate, the scanner can hold cash.

## 21. Break-even transaction cost

A descriptive break-even one-way cost can be approximated from gross contribution and one-way turnover:

<br>

```math
c_{\mathrm{BE}}
\approx
\frac{
R^{\mathrm{gross}}
}{
\tau^{\mathrm{one-way}}
}.
```

<br>

It is reported in basis points and should be interpreted cautiously when the number of trades is small.

## 22. Investment Monitor tiers

The monitor combines current timing with historical scanner attribution.

### Tier A

Current signal passes and the same relationship has strong positive scanner support.

### Tier B

Current signal passes but scanner history is absent, negative or too limited.

### Tier C

Current relationship remains usable but the current leader shock does not pass the selected threshold.

Historically supported Tier C relationships receive priority within the watch list.

### Tier D

Current direction or relation changed, the model became ineligible, or the beta-lag alignment failed.

## 23. Core interpretation rule

The project keeps three concepts separate:

<br>

```math
\text{statistical significance}
\neq
\text{economic significance}
\neq
\text{tradable profitability}.
```

<br>

A relationship is not promoted simply because one of those conditions is satisfied.
