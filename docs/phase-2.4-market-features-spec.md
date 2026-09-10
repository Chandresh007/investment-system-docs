# Phase 2.4 Technical Specification: Market Features (v1)

## 1. Objective
Transform raw market observations into deterministic, Point-in-Time (PIT) correct market features. This layer provides the evidence of price leadership, momentum, and risk to be combined with fundamental inflections.

## 2. Corporate-Action PIT Knowledge Model

### 2.1 Temporal Fields
Every corporate action (split, dividend, etc.) must be stored with the following temporal markers:
- `action_type`: (e.g., STOCK_SPLIT, REVERSE_SPLIT, CASH_DIVIDEND).
- `effective_date`: The date the action occurs (e.g., ex-dividend date, split effective date).
- `available_at`: The timestamp when the system first observed/recorded the action.
- `amount_per_share`: The cash amount (for dividends) or split ratio.
- `adjustment_factor`: The numeric multiplier applied to the price.
- `source/provenance`: Origin of the data.

### 2.2 PIT Access Rule
A corporate action is eligible for use in a research snapshot at timestamp $T$ **if and only if**:
$$\text{corporate\_action.available\_at} \le T$$

### 2.3 Deterministic Split Normalization
A split does not automatically adjust historical prices in the system. A split action affects the historical price series for research snapshot $T$ only when it is PIT-available.

**Rule**: The Split-Adjusted Price ($P'_t$) is calculated as:
$$P'_t = P_{\text{raw}, t} \times \prod \text{AdjustmentFactors}(\text{available\_at} \le T \text{ AND } \text{effective\_date} > t)$$

**Adversarial Cases**:
- **CASE A**: Split effective at date $E$, available before $E$. Snapshots after availability normalize observations where $t < E$ onto the post-split basis. Observations where $t \ge E$ are already on the post-split raw basis.
- **CASE B**: Split effective at $E$, but available only after $E$. Snapshots before availability **MUST NOT** use that split information for any date.

---

## 3. Total-Return Price Mathematics

### 3.1 The Canonical Construction Pipeline
Market features are derived from a strict sequential pipeline:
$$\text{Raw Price} \rightarrow \text{PIT-known Split Normalization} \rightarrow \text{Split-Continuous Price} \rightarrow \text{PIT-known Distributions} \rightarrow \text{Total Return Index (TRI)}$$

### 3.2 The Total Return Index (TRI)
The TRI solves total-return/dividend reinvestment semantics.

**Formula**:
$$\text{TRI}_t = \text{TRI}_{t-1} \times \frac{P'_t + D_t}{P'_{t-1}}$$
Where:
- $P'_t$: The split-continuous ex-distribution market price at time $t$.
- $D_t$: The cash distribution per share at time $t$, included **only if** `distribution.available_at` $\le T$.
- $\text{TRI}_0$: The initial split-adjusted price $P'_0$.

**Scope for v1**:
- **Included**: Stock splits, reverse splits, cash dividends.
- **Excluded**: Spin-offs, special dividends. These are marked as `MISSING` if they are the primary driver of the return.

---

## 4. Universal Window & Observation Convention

- **W-Day Price Window**: Contains exactly $W$ valid trading-session price observations, including session $T$.
- **W-Interval Return**: Requires exactly $W+1$ valid price observations to compute $W$ intervals.
- **SMA-W**: Requires exactly $W$ valid price observations.
- **W-Day Drawdown/High**: The window contains exactly $W$ valid price observations including $T$.

| Feature Type | Lookback ($W$) | Required Observations |
| :--- | :--- | :--- |
| Return | $W$ intervals | $W + 1$ |
| SMA | $W$ days | $W$ |
| Volatility | $W$ returns | $W + 1$ |
| Drawdown | $W$ days | $W$ |

---

## 5. Benchmark Methodology

### 5.1 Benchmark Hierarchy (v1)
1. **Broad Market**: `SPY` (S&P 500 ETF) - calculated as TRI.
2. **Sector**: GICS Sector ETF corresponding to the security's sector - calculated as TRI.
3. **Industry**: An **Equally Weighted PIT Basket**.

### 5.2 Industry Basket Determinism
The Industry benchmark is a deterministic daily rebalanced basket.

**Formula**:
$$\text{BasketReturn}_t = \frac{1}{N_t} \sum_{i \in \text{Industry}_t} \ln(\text{TRI}_{i, t} / \text{TRI}_{i, t-1})$$

**Deterministic Rules**:
- **Membership**: Based on historical sector membership where `start_date <= T AND (end_date IS NULL OR end_date > T)`. No inference from current membership.
- **Inclusion**: Only securities with a valid $\text{TRI}_t$ and $\text{TRI}_{t-1}$ are included.
- **Min Constituents**: $N_t \ge 3$. If $N_t < 3$, $\text{BasketReturn}_t = \text{MISSING}$.
- **Weighting**: $1/N$ (Equally Weighted). Weights are renormalized daily based on the valid constituents for that specific session.
- **Delisting**: Delisted securities remain in the basket until their last valid trading date.
- **Timing**: Recomputed daily.

---

## 6. Volume and Dollar Volume

### 6.1 Split-Adjusted Volume
$$\text{AdjVol}_t = \text{RawVol}_t \times \prod \frac{1}{\text{AdjustmentFactors}(\text{available\_at} \le T \text{ AND } \text{effective\_date} > t)}$$
Dividends do not alter share volume.

### 6.2 Dollar Volume
$$\text{DollarVol}_t = P'_t \times \text{AdjVol}_t$$
Where $P'_t$ is the split-continuous market price (NOT the TRI index level).

### 6.3 Volume Ratios
- **volume\_trend\_21**: $\frac{\text{mean}(\text{DollarVol}_{T-20 \dots T})}{\text{mean}(\text{DollarVol}_{T-220 \dots T-21})}$.
- **abnormal\_vol\_1**: $\frac{\text{DollarVol}_T}{\text{mean}(\text{DollarVol}_{T-20 \dots T-1})}$.
- **Zero Handling**: 
    - If denominator is $0 \rightarrow$ `INVALID` with reason `zero_denominator`.
    - If Raw Volume is $0 \rightarrow$ `DollarVol` is $0$ (VALID).
    - Division by zero must never produce $\text{inf}$ or $\text{NaN}$.

---

## 7. Authoritative Feature Registry (market_v1)

| Feature ID | Description | Formula | Input Series | Price Basis | Window | Req. Obs | Unit | Missing Behavior |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `return_1d` | 1D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-1})$ | TRI | TRI | 1 int | 2 | % | `MISSING` if $<2$ obs |
| `return_5d` | 5D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-5})$ | TRI | TRI | 5 int | 6 | % | `MISSING` if $<6$ obs |
| `return_21d` | 21D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-21})$ | TRI | TRI | 21 int | 22 | % | `MISSING` if $<22$ obs |
| `return_63d` | 63D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-63})$ | TRI | TRI | 63 int | 64 | % | `MISSING` if $<64$ obs |
| `return_126d` | 126D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-126})$ | TRI | TRI | 126 int | 127 | % | `MISSING` if $<127$ obs |
| `return_252d` | 252D Return | $\ln(\text{TRI}_t / \text{TRI}_{t-252})$ | TRI | TRI | 252 int | 253 | % | `MISSING` if $<253$ obs |
| `rel_ret_63_mkt` | Rel Return Mkt | $\text{return\_63d}_{\text{stock}} - \text{return\_63d}_{\text{SPY}}$ | TRI | TRI | 63 int | 64 | % | `MISSING` if either `MISSING` |
| `rel_ret_63_sec` | Rel Return Sec | $\text{return\_63d}_{\text{stock}} - \text{return\_63d}_{\text{SEC\_ETF}}$ | TRI | TRI | 63 int | 64 | % | `MISSING` if either `MISSING` |
| `rel_ret_63_ind` | Rel Return Ind | $\text{return\_63d}_{\text{stock}} - \text{return\_63d}_{\text{BASKET}}$ | TRI | TRI | 63 int | 64 | % | `MISSING` if either `MISSING` |
| `dist_52w_high` | Dist 52W High | $(\text{TRI}_t / \max(\text{TRI}_{t-251 \dots t})) - 1$ | TRI | TRI | 252 days | 252 | % | `MISSING` if $<252$ obs |
| `price_vs_sma200` | Price vs SMA 200 | $(\text{P}'_t / \text{SMA}_{200}(t)) - 1$ | $P'$ | Mkt Price | 200 days | 200 | % | `MISSING` if $<200$ obs |
| `sma_slope_50` | SMA 50 Slope | $\frac{\text{SMA}_{50}(t) - \text{SMA}_{50}(t-21)}{\text{SMA}_{50}(t-21)}$ | $P'$ | Mkt Price | 71 days | 71 | % | `MISSING` if any input `MISSING` |
| `realized_vol_63` | Realized Vol 63 | $\text{StdDev}(\text{log\_ret}_{63}) \times \sqrt{252}$ | TRI | TRI | 63 int | 64 | % | `MISSING` if $<64$ obs |
| `max_dd_252` | Max Drawdown 252 | $\min(\text{TRI}_k / \text{Peak}_k - 1)$ | TRI | TRI | 252 days | 252 | % | `MISSING` if $<252$ obs |
| `vol_regime` | Vol Regime Ratio | $\text{real\_vol\_21} / \text{real\_vol\_252}$ | TRI | TRI | 252 int | 253 | Ratio | `MISSING` if any input `MISSING` |
| `vol_trend_21` | Vol Trend 21 | $\frac{\text{mean}(\text{DV}_{t-20 \dots t})}{\text{mean}(\text{DV}_{t-220 \dots t-21})}$ | DV | Mkt Price | 241 days | 241 | Ratio | `MISSING` if den $= 0$ |
| `abnormal_vol_1` | Abnormal Vol 1 | $\text{DV}_t / \text{mean}(\text{DV}_{t-20 \dots t-1})$ | DV | Mkt Price | 21 days | 21 | Ratio | `MISSING` if den $= 0$ |
| `dollar_vol_21` | Avg Dollar Vol 21 | $\text{mean}(\text{DollarVol}_{t-20 \dots t})$ | DV | Mkt Price | 21 days | 21 | $ | `MISSING` if $<21$ obs |
| `mom_accel_63` | Mom Accel | $\text{return\_63d}_t - \text{return\_63d}_{t-63}$ | TRI | TRI | 127 int | 128 | % | `MISSING` if any input `MISSING` |
| `ret_abs_1d` | Abs 1D Return | $|\text{return\_1d}|$ | TRI | TRI | 1 int | 2 | % | `MISSING` if input `MISSING` |
| `ret_abs_21d` | Abs 21D Return | $|\text{return\_21d}|$ | TRI | TRI | 21 int | 22 | % | `MISSING` if input `MISSING` |
| `ret_abs_63d` | Abs 63D Return | $|\text{return\_63d}|$ | TRI | TRI | 63 int | 64 | % | `MISSING` if input `MISSING` |

**Price Basis Definitions**:
- **TRI**: Total Return Index (includes split normalization and reinvested dividends).
- **Mkt Price**: Split-continuous market price ($P'_t$).

---

## 8. Adjustment Engine and Feature Timing

### 8.1 snapshot: as\_of\_close(T)
- **Eligible Market Observations**: All prices where $\text{timestamp} \le \text{MarketClose}(T)$.
- **Eligible Corporate Actions**: All actions where $\text{available\_at} \le T$.
- **Eligible Benchmarks**: All benchmark prices where $\text{timestamp} \le \text{MarketClose}(T)$.
- **Eligible Membership**: All membership records where $\text{start\_date} \le T$ and ($\text{end\_date}$ is NULL or $> T$).

### 8.2 snapshot: as\_of\_open(T)
- **Eligible Market Observations**: All prices where $\text{timestamp} \le \text{MarketClose}(T-1)$.
- **Eligible Corporate Actions**: All actions where $\text{available\_at} \le T-1$.
- **Eligible Benchmarks**: All benchmark prices where $\text{timestamp} \le \text{MarketClose}(T-1)$.
- **Eligible Membership**: All membership records where $\text{start\_date} \le T-1$ and ($\text{end\_date}$ is NULL or $> T-1$).

---

## 9. Adversarial Test Requirements

### A. PIT Logic (Corporate Actions)
- **Split Knowledge**: 
    - Split effective at $E$, available at $A < E$. Verify it applies to $t < E$ for snapshots where $T \ge A$ and does not adjust raw observations where $t \ge E$.
    - Split effective at $E$, available at $A > E$. Verify it does **NOT** apply to $t \ge E$ for snapshots where $T < A$.
- **Dividend Knowledge**:
    - Dividend available at $A$. Verify it is only incorporated into TRI for snapshots where $T \ge A$.
- **Dividend Timing**: Verify that dividends are added to TRI on the `effective_date` (ex-date).

### B. Price & Volume Leakage
- **T+1 Leak**: Price at $T+1$ cannot affect feature at $T$.
- **Action Leak**: Corporate action available at $T+1$ cannot affect feature at $T$.
- **Dollar Vol Basis**: Verify $\text{DollarVol} = P' \times \text{AdjVol}$. Verify that using TRI instead of $P'$ fails the test.

### C. Mathematical Boundaries
- **Observation Counts**: 
    - $W+1$ prices $\rightarrow$ `VALID` for $W$-interval return.
    - $W$ prices $\rightarrow$ `MISSING` for $W$-interval return.
    - $200$ observations $\rightarrow$ `VALID` for SMA-200.
    - $199$ observations $\rightarrow$ `MISSING` for SMA-200.
- **Zero Vol**: Verify zero raw volume results in zero dollar volume (VALID), but zero-denominator ratios result in `INVALID`.

### D. Industry Basket PIT
- **Membership PIT**: Historical membership must be used. Verify that changing today's sector does not change a 2018 industry return.
- **Constituent Delisting**: Verify that a delisted constituent remains in the basket until its last trading date.
- **Minimum Count**: Verify basket return is `MISSING` when $< 3$ constituents are valid.

### E. System Invariants
- **Idempotency**: Same inputs + version $\rightarrow$ Identical `FeatureValue` rows.
- **Provenance**: Every `FeatureValue` must link to the $P_{raw}$ and `CorporateAction` IDs used.

---

## 10. Implementation Boundary
**Phase 2.4 is strictly limited to**:
$\text{RAW MARKET DATA} \rightarrow \text{PIT-SAFE SPLIT-ADJUSTED PRICE} \rightarrow \text{TRI} \rightarrow \text{DETERMINISTIC MARKET FEATURES} \rightarrow \text{FeatureValue}$

**Excluded**: Ranking, Z-scores, Factors, Signals, Backtesting.

## 11. Final Design Decisions

| Decision | v1 Resolution |
| :--- | :--- |
| **Price Pipeline** | Raw $\rightarrow$ Split-Adj $\rightarrow$ TRI. |
| **Split PIT** | Adjusted only if `available_at <= T`. |
| **Dividend PIT** | TRI return uses distributions only if `available_at <= T`. |
| **Dollar Volume** | $\text{Split-Adj Market Price} \times \text{Split-Adj Volume}$. |
| **SMA Basis** | Split-Adjusted Market Price ($P'$). |
| **Observation Window** | $W$-Day Window = $W$ obs; $W$-Interval = $W+1$ obs. |
| **Industry Basket** | Daily rebalanced, equal-weighted, PIT-membership, min 3 constituents. |
| **Timing** | `as_of_close(T)` uses data through $T$; `as_of_open(T)` uses data through $T-1$. |
| **Stale Price Threshold** | Latest observation is stale when it is more than 5 deterministic US equity trading sessions before the requested timestamp. Sessions are generated from the source-controlled calendar utility, not from unrelated securities' price rows. |

**IMPLEMENTATION STATUS: READY FOR REVIEW**
