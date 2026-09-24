# Investment Research System

## Current strategic status — 2026-09-21

Phase 2.5 implementation is complete, ready for independent review, NOT YET FROZEN.
Phase 2.6 has not started. Strategic verdict: **CONDITIONAL PASS**, not production
PIT or predictive-performance certification. The closed-loop section below and
[strategic review](STRATEGIC_ARCHITECTURE_REVIEW.md) qualify older educational claims
of universal immutability/PIT correctness. Formula contracts remain frozen;
source qualification, membership knowledge time and immutable publication remain gates.

## Complete Research, Data, Mathematical, and Algorithmic Blueprint

This document explains the investment research system from first principles. It is
educational, not API documentation. It describes the finance, statistics,
mathematics, data engineering, point-in-time architecture, and investment logic
behind the project.

Status labels used throughout:

| Label | Meaning |
| --- | --- |
| IMPLEMENTED | Code and schema exist in the repository. |
| FROZEN | Accepted implementation should not be reopened without a concrete regression. |
| DESIGNED | Architecture has been reviewed, but implementation is not complete. |
| PLANNED / NOT YET FROZEN | Intended future work; details can still change. |
| CONCEPTUAL | Educational framing or future possibility, not a project commitment. |

Repository source of truth reviewed for this document:

- `README.md`
- `docs/PROJECT_STATE.md`
- `docs/phase-2.2-handoff.md`
- `docs/phase-2.4-market-features-spec.md`
- `docs/data-model.md`
- `docs/architecture.md`
- `src/investment_research/models/`
- `src/investment_research/research/`
- `tests/`
- `database/migrations/`

---

# Part I - What Are We Trying To Build?

The system is a deterministic investment research engine. Its job is to help find
public companies whose business fundamentals and market behavior are improving
before that improvement is fully reflected in the stock price.

The system is not trying to guess tomorrow's price. It is trying to answer a more
institutional question:

```text
At time T, what could an investor have known,
what was changing,
how unusual was that change,
and did similar historical situations lead to attractive outcomes?
```

The engine is designed to detect evidence such as:

- accelerating revenue
- accelerating EPS
- estimate upgrades
- margin expansion
- operating leverage
- rising free cash flow
- market-share gains
- industry or secular tailwinds
- capacity constraints and bottlenecks
- product cycles
- customer wins
- hyperscaler demand
- improving relative strength
- price leadership
- unusual volume
- catalysts
- reasonable valuation relative to growth
- thesis invalidation conditions

## The Core Investment Hypothesis

Markets are reasonably efficient, but information is not always incorporated
instantly. Business improvement can appear first in operating data, then in
reported fundamentals, then in analyst estimates, then in institutional
expectations, and finally in price and valuation.

One possible company journey:

```text
business improvement
        |
        v
reported fundamentals improve
        |
        v
analysts revise forecasts
        |
        v
institutional expectations change
        |
        v
relative price strength improves
        |
        v
more investors recognize the story
        |
        v
valuation expands / earnings compound
```

The system tries to identify that transition systematically. It does this by
turning raw evidence into atomic facts, facts into normalized features, features
into factors, factors into signals, and signals into backtested rankings.

None of these signals guarantees investment success. A company can show strong
revenue acceleration and still disappoint because expectations were already too
high. Analyst upgrades can be late. Price momentum can reverse. Margins can be
temporarily inflated. A macro shock can overwhelm company-level progress. The
system's purpose is not certainty. Its purpose is disciplined evidence.

---

# Part II - The Complete Pipeline

The long-run architecture is:

```text
RAW DATA
    |
    v
TEMPORAL / POINT-IN-TIME ENGINE
    |
    v
ATOMIC FACTS
    |
    v
NORMALIZED FEATURES
    |
    v
FACTORS
    |
    v
DETECTOR SIGNALS
    |
    v
HISTORICAL BACKTESTING
    |
    v
CALIBRATION
    |
    v
RANKING
    |
    v
RESEARCH SNAPSHOT
    |
    v
AI SYNTHESIS
    |
    v
INVESTMENT DECISION
```

| Stage | Input | Output | Why It Exists | What Can Go Wrong | Status |
| --- | --- | --- | --- | --- | --- |
| Raw Data | Provider/API responses | Immutable raw objects | Audit trail and reproducibility | Provider changes, missing payloads, duplicate downloads | IMPLEMENTED |
| Temporal/PIT Engine | Raw objects and timestamps | PIT-eligible records | Prevent look-ahead bias | Wrong `available_at`, future leakage | IMPLEMENTED / FROZEN |
| Atomic Facts | Raw SEC, market, macro, news data | Normalized source-level rows | Preserve facts before interpretation | Bad fiscal identity, wrong units | IMPLEMENTED for fundamentals, market, macro/news basics |
| Normalized Features | Atomic facts | Deterministic metrics | Convert facts into comparable research evidence | Bad denominators, missing data, calendar/fiscal mistakes | IMPLEMENTED for fundamentals, market and estimates |
| Factors | Multiple features | Composite dimensions | Combine related evidence | Overfitting, arbitrary weights | PLANNED |
| Detector Signals | Factors and rules | Explainable signal events | Detect inflections | Screen overfitting, hidden assumptions | PLANNED |
| Historical Backtesting | PIT features/signals | Historical performance evidence | Test whether signals worked | Survivorship and look-ahead bias | PLANNED |
| Calibration | Backtest outputs | Interpretable probabilities/scores | Convert raw signal strength to empirical meaning | Small samples, unstable regimes | PLANNED |
| Ranking | Calibrated scores | Ordered stock list | Prioritize research attention | Crowding, stale data | PLANNED |
| Research Snapshot | All evidence at T | Structured research packet | Human and AI-readable evidence | Missing provenance | PLANNED |
| AI Synthesis | Structured snapshot | Narrative explanation | Explain, summarize, compare evidence | Hallucination, invented numbers | PLANNED |
| Investment Decision | Evidence and human judgment | Buy/watch/pass decision | Portfolio action remains human-controlled | Narrative attachment, risk neglect | CONCEPTUAL |

Each stage depends on the previous stage being correct. A beautiful ranking model
is worthless if it uses future filings. An elegant AI summary is dangerous if it
summarizes fabricated features. This is why the project builds from raw data and
PIT correctness upward.

---

# Part III - What Data Are We Trying To Get?

| Data Category | Examples | Source | Why Needed | PIT Requirement | Current Status |
| --- | --- | --- | --- | --- | --- |
| Company master | Company, ticker, exchange, CIK, fiscal year end | SEC, configured universe, provider metadata | Entity resolution | Must know which security existed at T | IMPLEMENTED basics |
| Historical universe | Listing, delisting, sector, industry | Universe files/providers | Avoid survivorship and peer leakage | Membership must be valid at T | IMPLEMENTED foundation |
| SEC fundamentals | Revenue, EPS, margins, cash flow | SEC Company Facts/XBRL | Business performance | Filing date must be <= T | IMPLEMENTED / FROZEN for Phase 2.2 features |
| Market data | OHLCV, prices, volume | Yahoo market connector in current repo | Momentum, liquidity, risk | Price/retrieval and corporate actions must be PIT | IMPLEMENTED / FROZEN for Phase 2.4 |
| Corporate actions | Splits, reverse splits, dividends | Market data providers | Correct price/return math | Action usable only when available at T | IMPLEMENTED / FROZEN |
| Analyst estimates | EPS/revenue estimates, consensus, revisions | Future commercial provider | Expectations and revision momentum | Estimate event must be available at T | IMPLEMENTATION COMPLETE / INDEPENDENT REVIEW PENDING |
| Peer/industry data | GICS, peers, market share, peer metrics | Future providers and normalized fundamentals | Relative context | Historical membership only | PLANNED |
| Macro data | CPI, rates, unemployment, GDP | FRED connector | Macro backdrop | Observation/revision availability matters | IMPLEMENTED basics / PLANNED expansion |
| News/events | Earnings, product launches, customer wins | RSS/news connectors, future extraction | Catalysts and context | Publication time must be <= T | IMPLEMENTED basics / PLANNED structure |
| Valuation data | Market cap, EV, P/E, EV/Sales, P/FCF | Market + fundamentals + estimates | Price paid for growth | Inputs must be PIT | PLANNED |

## 1. Company Master Data

Company master data answers: "What company and security are we talking about?"

Important fields:

- company name
- ticker
- security identifier
- exchange
- CIK for SEC data
- fiscal year end
- sector
- industry
- listing and delisting dates
- historical universe membership

Survivorship bias occurs when a backtest only includes companies that survived
until today. If a strategy avoids bankruptcies only because the database removed
bankrupt stocks from history, the backtest is fake. The current project has a
`historical_universe` table with inclusive `start_date` and exclusive `end_date`
semantics. It lacks a separate knowledge timestamp and versioned corrections;
these effective intervals alone do not prove historical membership was known at T.

## 2. SEC Fundamental Data

SEC filings are the main source for accepted fundamental features.

Key filings:

- `10-K`: annual report.
- `10-Q`: quarterly report.
- `8-K`: event report; relevant later for earnings releases and guidance.

SEC Company Facts and XBRL provide tagged accounting facts. A fact is tied to a
concept such as revenue, net income, EPS, or operating cash flow. It also has a
period, unit, filing, and accession number.

Accepted and desired metrics include:

- revenue
- cost of revenue
- gross profit
- operating income
- net income
- diluted EPS
- assets
- liabilities
- equity
- cash
- debt
- operating cash flow
- CapEx
- free cash flow when derivable

Important time and fiscal fields:

| Field | Meaning |
| --- | --- |
| `period_end` | The end date of the accounting period described by the fact. |
| `filed_date` / `available_at` | When the fact became knowable. |
| `retrieved_at` | When this system downloaded it. |
| `fiscal_year` | Company fiscal year, from SEC `fy`. |
| `fiscal_period` | Company fiscal period, from SEC `fp`, such as `Q1` or `FY`. |
| `period_type` | Accounting shape: quarterly, annual, YTD, instant, unknown. |

The most important fiscal rule is:

```text
Company fiscal quarter is not the same thing as calendar quarter.
```

NVIDIA-style example:

```text
Calendar date: April 26, 2026
Company fiscal identity: FY2027 Q1
```

If code looked only at the calendar month, April would look like calendar Q2.
That would be wrong for a non-calendar fiscal-year company. The accepted Phase
2.2 rule is that fiscal identity comes from source fiscal metadata such as SEC
`fy/fp`, not from calendar month.

YTD versus standalone quarterly data also matters. A Q2 YTD revenue number may
cover six months. A standalone Q2 number covers one quarter. They cannot be
substituted for each other just because both end on the same date.

## 3. Market Data

Market data includes:

- daily price
- daily volume
- ticker/security identity
- trading dates
- splits
- reverse splits
- dividends
- delisting behavior

The Phase 2.4 frozen pipeline is:

```text
raw market price
    |
    v
PIT-known split normalization
    |
    v
split-continuous price
    |
    v
PIT-known dividends
    |
    v
total return index
    |
    v
market features
```

The required raw price is the as-traded price for a date; a provider field called
close is not sufficient proof of that basis. Split-continuous price adjusts
old prices for PIT-known splits so pre-split and post-split prices can be
compared. Total return index also incorporates PIT-known dividends.

## 4. Analyst Estimates - IMPLEMENTED / NOT YET FROZEN

Analyst estimates represent expectations, not actuals.

Desired information:

- analyst EPS estimates
- analyst revenue estimates
- analyst identity
- broker
- estimate timestamp
- revision timestamp
- target fiscal period
- GAAP/non-GAAP basis
- consensus
- estimate count
- dispersion
- upgrades/downgrades
- withdrawals

The critical design principle is to distinguish absolute target fiscal identity
from relative forward horizon.

Stable target identity:

```text
FY2027 Q2
```

Relative horizon:

```text
FQ1
```

`FY2027 Q2` means the same quarter forever. `FQ1` means "the next unreported
quarter as of T." Before FY2027 Q1 is reported, FQ1 may be FY2027 Q1. After Q1
is reported and known, FQ1 becomes FY2027 Q2. The stored estimate identity must
remain stable even when its relative horizon changes.

## 5. Industry / Peer Data - PLANNED

Peer and industry data will eventually include:

- GICS sector
- industry
- peer group
- historical membership
- market share
- peer growth
- peer margins
- relative financial performance

Example:

```text
Company revenue growth = 40%
Industry median revenue growth = 15%
Company minus industry = +25 percentage points
```

That may indicate market-share gain, product leadership, or cycle advantage.

## 6. Macro Data

FRED ingestion exists in the repository. Macro series that are available or
reasonable extensions include:

- CPI
- core CPI
- PCE
- Fed Funds
- 2-year Treasury
- 10-year Treasury
- unemployment
- GDP

Macro matters because rates, inflation, credit, and employment affect valuation,
demand, and risk appetite. But this system is company-first. Macro context should
not dominate company-level evidence unless the strategy explicitly proves that it
does.

## 7. News / Events / Catalysts

News and event data can cover:

- earnings
- product launches
- customer wins
- hyperscaler contracts
- capacity expansion
- regulatory changes
- acquisitions
- management commentary
- guidance
- supply constraints

The architecture must separate objective event extraction from subjective AI
interpretation. A structured event might say, "Company announced a new customer
contract on date X." AI can summarize why it may matter, but AI must not invent
the event or a numeric impact.

## 8. Valuation Data - PLANNED

Likely later inputs:

- enterprise value
- market cap
- net debt
- EV/Sales
- EV/EBITDA
- P/E
- P/FCF
- PEG-style concepts
- FCF yield
- forward valuation

Valuation alone is not a complete quality or timing signal. A cheap stock can be
cheap for a reason. A high-growth stock can deserve a premium, but not an
unlimited one.

---

# Part IV - How Are We Getting The Data?

The ingestion pattern is:

```text
provider/API
    |
    v
raw immutable response
    |
    v
SHA256/content hash
    |
    v
raw_data_object
    |
    v
normalizer
    |
    v
canonical database rows
    |
    v
PIT timestamps
```

The current raw storage layer saves the raw response content and records metadata
in `raw_data_objects`, including source, endpoint, request hash, content hash,
file path, content type, size, and retrieval timestamp.

## Why Keep Raw Objects?

Raw objects exist because normalized rows are interpretations of source data. If
a normalizer has a bug, raw objects let us reprocess without asking the provider
for the historical response again. They also let an auditor trace a feature back
to the original evidence.

Important data engineering concepts:

| Concept | Meaning | Why It Matters |
| --- | --- | --- |
| Idempotency | Running the same ingestion twice should not create false duplicates. | Prevents inflated counts and unstable features. |
| Caching | Avoid repeated unnecessary provider calls. | Reduces cost and rate-limit pressure. |
| Freshness | Know when data is stale. | Prevents stale prices or stale estimates from being treated as current. |
| Request hash | Stable identity for a request. | Helps detect repeated requests. |
| Content hash | SHA256 fingerprint of payload bytes. | Helps detect identical or changed content. |
| Provenance | Link output rows to source objects. | Makes features reproducible. |
| Validation | Check units, timestamps, identifiers, and required fields. | Prevents malformed data from entering calculations. |
| Provider abstraction | Provider-specific code stops at normalization. | Core research logic is not hard-coded to one vendor. |

SEC ingestion currently goes through raw SEC responses and normalization into
`financial_facts`. Market ingestion normalizes price observations into
`market_prices`. FRED and news ingestion have connector/normalization structures
for economic and news data.

Implemented synthetic estimates ingestion follows the same boundary; production qualification remains pending:

```text
raw provider connector
    |
    v
immutable raw object
    |
    v
provider-specific normalizer
    |
    v
canonical EstimateObservation
    |
    v
PIT consensus resolver
    |
    v
estimate feature calculator
```

If a commercial estimate provider does not supply historical PIT estimate
events, the system cannot reconstruct true historical revisions. Today's current
consensus cannot be used to fabricate what analysts believed years ago.

---

# Part V - Point-In-Time Research

Point-in-time correctness means:

```text
Use a record at research time T only if available_at <= T.
```

This is one of the most important rules in the project.

Example:

```text
Company reports Q2 numbers on August 20.
Research snapshot: August 10
```

The August 10 snapshot must not use the Q2 report because investors could not
yet know it.

```text
Research snapshot: August 21
```

The August 21 snapshot may use the Q2 report if the filing or release was
available by then.

## Types of Bias

Look-ahead bias: using future information in a historical decision.

Hindsight bias: interpreting a past situation using what later happened.

Survivorship bias: excluding companies that disappeared before today.

Future corporate-action leakage: adjusting historical prices using a split that
was not known at the research time.

Future analyst-estimate leakage: using revisions published after the snapshot.

Historical index membership leakage: using today's sector or index membership
for a past date.

## Adversarial Examples

1. A filing arrives on May 10. A backtest signal on May 1 cannot use it.
2. A 2-for-1 split is effective June 1 but only observed by the system June 3.
   A June 2 snapshot cannot use the split adjustment if it was not PIT-known.
3. A company delisted in 2020 must still appear in a 2019 historical universe.
4. A provider corrects a 2018 estimate history in 2026. A 2019 backtest cannot
   act as if the correction was known in 2019.
5. A company reports on a non-calendar fiscal year. Calendar Q2 cannot be used
   as a substitute for fiscal Q1.

A backtest without PIT correctness can look excellent because it unknowingly
selects stocks using tomorrow's answers.

---

# Part VI - Fundamental Feature Mathematics

Phase 2.2 fundamental features are IMPLEMENTED. The accepted implementation uses
`FundamentalFeatureRunner`, `SECFactResolver`, `PeriodResolver`, and
`FundamentalCalculator`.

Important implementation rules:

- `FinancialFact.filed_date <= available_at` for PIT fact eligibility.
- Fiscal matching uses `fiscal_year` and `fiscal_period`.
- Standalone quarterly and YTD facts must not be mixed.
- Q1 can be classified as `UNKNOWN` for period shape when duration is ambiguous,
  but explicit fiscal identity is still preserved.
- `FeatureValue.available_at` is the maximum availability of all inputs.
- Missingness is explicit.
- Zero denominators produce `INVALID` for ratio features.
- Sign changes in implemented growth formulas produce `INVALID`.

## Revenue YoY Growth

Formula:

```text
RevenueGrowthYoY = Revenue_t / Revenue_t-4Q - 1
```

Symbols:

- `Revenue_t`: revenue for the current fiscal quarter.
- `Revenue_t-4Q`: revenue for the same fiscal quarter one fiscal year earlier.

Example:

```text
FY2027 Q2 revenue = 150
FY2026 Q2 revenue = 100
RevenueGrowthYoY = 150 / 100 - 1 = 0.50 = 50%
```

Investment intuition: revenue growth shows whether customer demand is rising.
YoY comparison reduces seasonality because Q2 is compared with Q2.

Limitations: acquisitions, accounting changes, or one-time demand spikes can
inflate growth. If prior revenue is zero, the implementation marks the feature
`INVALID` because the ratio denominator is zero.

## Revenue QoQ Growth

Formula:

```text
RevenueGrowthQoQ = Revenue_t / Revenue_t-1Q - 1
```

Example:

```text
FY2027 Q2 revenue = 150
FY2027 Q1 revenue = 120
RevenueGrowthQoQ = 150 / 120 - 1 = 0.25 = 25%
```

Intuition: QoQ growth detects sequential acceleration or slowdown more quickly
than YoY growth.

Limitations: seasonality can mislead. A retailer's Q4 may naturally exceed Q3.
That is why YoY and QoQ should be interpreted together.

## Revenue Growth Acceleration

Formula:

```text
Acceleration = CurrentGrowth - PreviousGrowth
```

Example:

```text
FY2027 Q2 YoY growth = 50%
FY2027 Q1 YoY growth = 25%
Acceleration = 50% - 25% = +25 percentage points
```

Intuition: this is a first-derivative concept. Growth tells us direction;
acceleration tells us whether the direction is improving.

Limitations: acceleration can spike from a depressed base. It can also reverse
quickly after a product cycle peak.

## Gross Margin

Formula:

```text
GrossMargin = GrossProfit / Revenue
```

Example:

```text
Gross profit = 55
Revenue = 100
GrossMargin = 55 / 100 = 55%
```

Intuition: gross margin measures how much revenue remains after direct production
costs. Rising gross margin can indicate pricing power, mix improvement, or scale.

Limitations: product mix, inventory accounting, and temporary supply constraints
can distort gross margin.

## Gross Margin YoY Change

Formula:

```text
DeltaGrossMarginYoY = GrossMargin_t - GrossMargin_t-4Q
```

Example:

```text
Current gross margin = 52%
Prior-year same quarter gross margin = 45%
DeltaGrossMarginYoY = 52% - 45% = +7 percentage points
```

Intuition: margin expansion combined with revenue growth is stronger than
revenue growth alone.

Limitations: cost deferrals and temporary input-cost movements can reverse.

## Operating Margin

Formula:

```text
OperatingMargin = OperatingIncome / Revenue
```

Example:

```text
Operating income = 25
Revenue = 100
OperatingMargin = 25 / 100 = 25%
```

Intuition: operating margin measures profitability after operating expenses.

Limitations: restructuring charges or stock compensation treatment can affect
comparability.

## Operating Margin YoY Change

Formula:

```text
DeltaOperatingMarginYoY = OperatingMargin_t - OperatingMargin_t-4Q
```

Example:

```text
Current operating margin = 28%
Prior-year operating margin = 20%
Change = +8 percentage points
```

Intuition: this captures operating leverage. If revenue rises faster than
operating expenses, operating income can grow faster than revenue.

## Operating Income YoY Growth

Formula:

```text
OperatingIncomeGrowthYoY = OperatingIncome_t / OperatingIncome_t-4Q - 1
```

Example:

```text
Operating income rises from 20 to 30.
Growth = 30 / 20 - 1 = 50%
```

Intuition: operating income growth shows whether business improvement is reaching
the operating profit line.

Limitation: if the prior period is negative and the current is positive, the
implemented growth function marks sign-change growth `INVALID`.

## Diluted EPS YoY Growth

Formula:

```text
EPSGrowthYoY = DilutedEPS_t / DilutedEPS_t-4Q - 1
```

Example:

```text
Current diluted EPS = 1.10
Prior-year diluted EPS = 0.55
EPSGrowthYoY = 1.10 / 0.55 - 1 = 100%
```

Intuition: EPS combines operating performance, taxes, interest, and share count.

Limitations: buybacks, tax changes, one-time items, and GAAP/non-GAAP differences
can distort interpretation.

## EPS Growth Acceleration

Formula:

```text
EPSAcceleration = EPSGrowthYoY_t - EPSGrowthYoY_t-1Q
```

Example:

```text
Current EPS YoY growth = 100%
Previous EPS YoY growth = 50%
EPSAcceleration = +50 percentage points
```

Intuition: EPS acceleration is often watched because earnings revisions and price
momentum can follow sustained earnings acceleration.

## Operating Cash Flow YoY Growth

Formula:

```text
OCFGrowthYoY = OCF_t / OCF_t-4Q - 1
```

Example:

```text
OCF rises from 80 to 100.
OCFGrowthYoY = 100 / 80 - 1 = 25%
```

Intuition: cash flow can validate or contradict accounting earnings.

Limitations: working-capital timing can make quarterly OCF noisy.

## Free Cash Flow

Implemented formula:

```text
FCF = OperatingCashFlow - abs(CapEx)
```

The implementation uses `abs(CapEx)` because SEC cash-flow statement CapEx may
appear with different signs.

Example:

```text
Operating cash flow = 80
CapEx = 20
FCF = 80 - abs(20) = 60
```

Intuition: free cash flow estimates cash left after reinvestment in property,
plant, equipment, and similar capital assets.

Limitations: growth companies may have low FCF because they are reinvesting
heavily; that is not automatically bad.

## FCF YoY Growth

Formula:

```text
FCFGrowthYoY = FCF_t / FCF_t-4Q - 1
```

Example:

```text
Current FCF = 60
Prior-year FCF = 40
FCFGrowthYoY = 60 / 40 - 1 = 50%
```

Limitation: FCF can be volatile due to working capital and CapEx timing.

## FCF Margin

Formula:

```text
FCFMargin = FCF / Revenue
```

Example:

```text
FCF = 60
Revenue = 200
FCFMargin = 60 / 200 = 30%
```

Intuition: FCF margin shows cash generation per dollar of sales.

## FCF Conversion

Implemented formula:

```text
FCFConversion = FCF / NetIncome
```

Example:

```text
FCF = 60
Net income = 50
FCFConversion = 60 / 50 = 1.20
```

Intuition: values above 1.0 can indicate earnings are converting strongly into
cash. Values below 1.0 may indicate working-capital drag, heavy CapEx, or lower
cash quality.

Limitations: net income can be near zero or negative. The implemented margin
calculator marks zero denominators `INVALID`.

---

# Part VII - Market Feature Mathematics

Phase 2.4 market features are COMPLETE / FROZEN. The implementation is in
`MarketFeatureRunner`, `MarketAdjustmentEngine`, `MarketPriceResolver`,
`MarketBenchmarkResolver`, `IndustryBasketResolver`, and `MarketCalculator`.

## Split Normalization

Example before a 2-for-1 split:

```text
raw price = $100
raw volume = 1,000
```

Normalized onto post-split basis:

```text
split-adjusted price = $50
split-adjusted volume = 2,000
```

Economic turnover is preserved:

```text
100 * 1,000 = 50 * 2,000 = 100,000
```

Frozen formula for split-continuous price:

```text
P'_t = P_raw,t * product(adjustment_factors)
```

Only actions with `available_at <= T` are used, and only for observations before
the split effective date.

Frozen formula for adjusted volume:

```text
AdjVol_t = RawVol_t * product(1 / adjustment_factor)
```

## Total Return Index

Formula:

```text
TRI_t = TRI_(t-1) * (P'_t + D_t) / P'_(t-1)
```

Symbols:

- `TRI_t`: total return index at time `t`.
- `TRI_(t-1)`: prior total return index.
- `P'_t`: split-continuous price at `t`.
- `D_t`: dividend at `t`, only if PIT-known.

Example:

```text
TRI_(t-1) = 100
P'_(t-1) = 50
P'_t = 52
D_t = 1
TRI_t = 100 * (52 + 1) / 50 = 106
```

Intuition: the investor earned both price appreciation and dividend value.

## Log Returns

Formula:

```text
r = ln(P_t / P_(t-n))
```

Example:

```text
P_t = 110
P_(t-n) = 100
r = ln(110 / 100) = ln(1.10) = 0.0953
```

Log returns are useful because multi-period log returns add:

```text
ln(P2/P1) + ln(P3/P2) = ln(P3/P1)
```

Limitations: log returns require positive prices.

## Observation Count Rule

Frozen Phase 2.4 rule:

```text
W-interval return = W + 1 prices
W-day SMA/high/drawdown window = W prices
```

For a 21-interval return, the system needs 22 valid price observations.

## Market Feature Table

| Feature | Formula | Required Observations | Intuition | Limitation |
| --- | --- | --- | --- | --- |
| `return_1d` | `ln(TRI_t / TRI_t-1)` | 2 | One-session return | Noisy |
| `return_5d` | `ln(TRI_t / TRI_t-5)` | 6 | One-week move | Can be event-driven |
| `return_21d` | `ln(TRI_t / TRI_t-21)` | 22 | About one trading month | Short-term reversals |
| `return_63d` | `ln(TRI_t / TRI_t-63)` | 64 | About one quarter momentum | Can be crowded |
| `return_126d` | `ln(TRI_t / TRI_t-126)` | 127 | About six months | Slower signal |
| `return_252d` | `ln(TRI_t / TRI_t-252)` | 253 | About one year | May lag turning points |
| `rel_ret_63_mkt` | stock 63d return - SPY 63d return | stock and SPY windows | Market-relative leadership | SPY may be broad context only |
| `rel_ret_63_sec` | stock 63d return - sector ETF 63d return | stock and sector windows | Sector-relative leadership | ETF mapping is coarse |
| `rel_ret_63_ind` | stock 63d return - industry basket 63d return | stock and industry windows | Peer-relative leadership | Needs enough valid peers |
| `dist_52w_high` | `TRI_t / max(TRI_t-251...t) - 1` | 252 | Distance from recent high | High may be stale |
| `price_vs_sma200` | `P'_t / SMA200 - 1` | 200 | Long-trend position | Can whipsaw |
| `sma_slope_50` | `(SMA50_t - SMA50_t-21) / SMA50_t-21` | 71 | Trend direction | Slow to react |
| `realized_vol_63` | sample stddev of 63 daily log returns * `sqrt(252)` | 64 | Risk level | Backward-looking |
| `max_dd_252` | minimum drawdown over 252 prices | 252 | Recent downside severity | Path-dependent |
| `vol_regime` | 21d vol / 252d vol | 253 | Current risk versus long risk | Can spike temporarily |
| `vol_trend_21` | mean current 21d dollar volume / mean prior 220d dollar volume | 241 | Liquidity trend | Volume can be news-noise |
| `abnormal_vol_1` | current dollar volume / prior 20d mean dollar volume | 21 | Unusual attention | One-day event noise |
| `dollar_vol_21` | mean 21d dollar volume | 21 | Institutional liquidity | Not a return signal |
| `mom_accel_63` | current 63d return - prior 63d return | 127 in implementation | Momentum acceleration | Sensitive near inflection |
| `ret_abs_1d` | absolute value of `return_1d` | 2 | Move magnitude | Directionless |
| `ret_abs_21d` | absolute value of `return_21d` | 22 | Monthly move magnitude | Directionless |
| `ret_abs_63d` | absolute value of `return_63d` | 64 | Quarterly move magnitude | Directionless |

Note: the Phase 2.4 spec table lists `mom_accel_63` as requiring 128
observations, while the implementation checks 127 TRI values. This document does
not change the frozen code; it records the discrepancy for awareness.

## Simple Numerical Examples

Return:

```text
TRI 63 sessions ago = 100
TRI today = 128
return_63d = ln(128 / 100) = 0.2469
```

Relative return:

```text
stock return_63d = 0.2469
SPY return_63d = 0.0677
rel_ret_63_mkt = 0.2469 - 0.0677 = 0.1792
```

Price versus SMA:

```text
P'_t = 120
SMA200 = 100
price_vs_sma200 = 120 / 100 - 1 = 20%
```

Abnormal volume:

```text
DollarVol_today = 50,000,000
Mean prior 20-day DollarVol = 20,000,000
abnormal_vol_1 = 50,000,000 / 20,000,000 = 2.5
```

## Worked Market Feature Reference

The examples below use simplified numbers. The implementation uses TRI for
return features and split-continuous price `P'` for SMA/dollar-volume features.

| Feature | Tiny Worked Example |
| --- | --- |
| `return_1d` | Yesterday TRI = 100, today TRI = 101. `ln(101/100) = 0.00995`. A positive one-day return. |
| `return_5d` | TRI 5 sessions ago = 100, today = 105. `ln(105/100) = 0.0488`. About a one-week advance. |
| `return_21d` | TRI 21 sessions ago = 100, today = 110. `ln(110/100) = 0.0953`. About one-month momentum. |
| `return_63d` | TRI 63 sessions ago = 100, today = 128. `ln(128/100) = 0.2469`. Quarterly momentum. |
| `return_126d` | TRI 126 sessions ago = 100, today = 140. `ln(140/100) = 0.3365`. Six-month momentum. |
| `return_252d` | TRI 252 sessions ago = 100, today = 160. `ln(160/100) = 0.4700`. One-year momentum. |
| `rel_ret_63_mkt` | Stock 63d log return = 0.2469, SPY = 0.0677. Difference = `0.1792`. Stock beat market. |
| `rel_ret_63_sec` | Stock 63d = 0.2469, sector ETF = 0.1500. Difference = `0.0969`. Stock beat sector. |
| `rel_ret_63_ind` | Stock 63d = 0.2469, PIT industry basket = 0.1133. Difference = `0.1336`. Stock beat peers. |
| `dist_52w_high` | Today TRI = 92, 252-day high TRI = 100. `92/100 - 1 = -8%`. Stock is 8% below high. |
| `price_vs_sma200` | `P'_t = 120`, SMA200 = 100. `120/100 - 1 = 20%`. Price is above long trend. |
| `sma_slope_50` | Current SMA50 = 110, SMA50 21 sessions ago = 100. `110/100 - 1 = 10%`. Medium trend is rising. |
| `realized_vol_63` | Daily sample stddev = 1.5%. Annualized = `0.015 * sqrt(252) = 23.8%`. |
| `max_dd_252` | Path high = 120, later low = 90. `90/120 - 1 = -25%`. Worst observed drawdown is 25%. |
| `vol_regime` | 21d vol = 30%, 252d vol = 20%. `30%/20% = 1.5`. Recent risk is elevated. |
| `vol_trend_21` | Mean recent 21d dollar volume = 50M, prior baseline = 25M. Ratio = `2.0`. Liquidity doubled. |
| `abnormal_vol_1` | Today dollar volume = 60M, prior 20d average = 20M. Ratio = `3.0`. Unusual volume. |
| `dollar_vol_21` | 21 daily dollar-volume values sum to 1.05B. Mean = `1.05B/21 = 50M`. |
| `mom_accel_63` | Current 63d return = 24%, prior 63d return = 8%. Difference = `+16 percentage points`. |
| `ret_abs_1d` | `return_1d = -3%`. Absolute return = `3%`. Captures move size, not direction. |
| `ret_abs_21d` | `return_21d = +9%`. Absolute return = `9%`. Measures monthly move magnitude. |
| `ret_abs_63d` | `return_63d = -18%`. Absolute return = `18%`. Measures quarterly move magnitude. |

Each example has a limitation. Returns can reverse, relative strength can reflect
temporary flows, volume can be mechanical, and volatility is backward-looking.
The system treats these as evidence, not conclusions.

---

# Part VIII - Momentum

Momentum is the tendency for assets that have been outperforming to continue
outperforming over certain horizons.

Types:

- absolute momentum: the stock's own return is positive or strong.
- relative momentum: the stock outperforms a benchmark.
- acceleration: recent momentum is stronger than prior momentum.
- short horizon: days to weeks, often news-driven.
- medium horizon: one to six months, often institutional accumulation.
- long horizon: one year, often captures persistent business recognition.

Example:

```text
Stock return = +20%
Market return = +15%
Relative return = +5 percentage/log-return points
```

Compare:

```text
Stock return = +5%
Market return = -10%
Relative return = +15 percentage/log-return points
```

The second stock rose less in absolute terms but performed much better relative
to the market environment.

Why momentum can persist:

- investors underreact to new information
- institutions build positions gradually
- analysts revise forecasts slowly
- business trends persist
- constraints prevent immediate arbitrage

Momentum can fail through crowding, sharp reversals, valuation shocks, earnings
disappointments, or regime changes.

---

# Part IX - Relative Strength

Relative strength is:

```text
RelativeStrength = StockReturn - BenchmarkReturn
```

The frozen Phase 2.4 benchmark hierarchy:

1. Broad market: SPY.
2. Sector: deterministic sector ETF mapping.
3. Industry: PIT-correct equal-weight industry basket.

Industry comparison is often more informative than market comparison. A
semiconductor stock outperforming utilities may not mean much. A semiconductor
stock outperforming other semiconductor stocks may signal company-specific
leadership.

Frozen industry basket method:

```text
r_industry,t = (1 / N_t) * sum(r_i,t)
```

Symbols:

- `r_industry,t`: industry return on session `t`.
- `N_t`: number of valid constituents on session `t`.
- `r_i,t`: constituent `i` return on session `t`.

For 63-session log returns:

```text
R_industry,63 = sum(r_industry,t for 63 sessions)
RelativeStrength = R_stock,63 - R_industry,63
```

Historical membership matters. A company that was not in the industry at the
time cannot be retroactively included because it belongs today.

---

# Part X - Volatility And Risk Math

Daily log returns:

```text
r_1, r_2, ..., r_63
```

Sample mean:

```text
r_bar = (1 / n) * sum(r_i)
```

Sample standard deviation:

```text
s = sqrt(sum((r_i - r_bar)^2) / (n - 1))
```

Annualized volatility:

```text
sigma_annual = s * sqrt(252)
```

Symbols:

- `n`: number of daily returns.
- `r_i`: return on day `i`.
- `r_bar`: average daily return.
- `s`: sample daily standard deviation.
- `252`: approximate US trading sessions per year.

Example:

```text
daily sample standard deviation = 1.5% = 0.015
sigma_annual = 0.015 * sqrt(252) = 0.238 = 23.8%
```

The square-root-of-time rule assumes returns are roughly independent over time.
That is useful but imperfect, especially during crises.

## Maximum Drawdown

Formula:

```text
Peak_t = max(P_1 ... P_t)
DD_t = P_t / Peak_t - 1
MaxDD = min(DD_t)
```

Example path:

```text
100, 120, 90, 130, 104
```

At 90, peak is 120:

```text
DD = 90 / 120 - 1 = -25%
```

At 104, peak is 130:

```text
DD = 104 / 130 - 1 = -20%
```

Max drawdown is `-25%`.

Volatility regime compares short-term volatility to long-term volatility:

```text
vol_regime = realized_vol_21 / realized_vol_252
```

A value above 1 means recent volatility is higher than longer-run volatility.

---

# Part XI - Moving Averages

Simple moving average:

```text
SMA_N = (1 / N) * sum(P_i)
```

Symbols:

- `N`: number of observations.
- `P_i`: price observation.

Example:

```text
Prices = 100, 102, 104
SMA_3 = (100 + 102 + 104) / 3 = 102
```

`price_vs_sma200`:

```text
price_vs_sma200 = P'_t / SMA200 - 1
```

Example:

```text
P'_t = 120
SMA200 = 100
price_vs_sma200 = 20%
```

`sma_slope_50`:

```text
sma_slope_50 = (SMA50_t - SMA50_t-21) / SMA50_t-21
```

Example:

```text
SMA50_t = 110
SMA50_t-21 = 100
sma_slope_50 = 10%
```

Being above the 200-day average says price is above a long-term trend. A rising
50-day average says the trend itself is improving.

---

# Part XII - Volume

Share volume counts shares traded. Adjusted volume puts old volume on a
split-comparable basis. Dollar volume measures economic trading value:

```text
DollarVol = SplitAdjustedPrice * SplitAdjustedVolume
```

Example:

```text
price = 50
adjusted volume = 2,000,000
DollarVol = 100,000,000
```

Dollar volume matters because institutional investors need liquidity. A small
stock can have a high percentage return but not enough liquidity for meaningful
institutional ownership.

High volume with rising price can indicate accumulation. High volume with falling
price can indicate distribution or forced selling. Volume can also create false
positives around index rebalances, options expirations, one-time news, or
mechanical trading.

---

# Part XIII - Analyst Estimates - IMPLEMENTED / NOT YET FROZEN

Analyst estimate systems convert individual forecasts into expectation signals.

```text
individual estimates
        |
        v
consensus
        |
        v
revisions
        |
        v
breadth
        |
        v
revision acceleration
```

## Consensus

Example estimates:

```text
2.00, 2.10, 2.20, 2.30
```

Mean:

```text
mean = (2.00 + 2.10 + 2.20 + 2.30) / 4 = 2.15
```

Median:

```text
median = (2.10 + 2.20) / 2 = 2.15
```

Dispersion can be measured by standard deviation. High dispersion means analysts
disagree, often because the business outlook is uncertain.

## Revisions

Same analyst:

```text
old EPS estimate = 2.00
new EPS estimate = 2.30
absolute revision = +0.30
```

Naive percentage revision:

```text
new / old - 1
```

This breaks around zero or negative EPS. If old EPS is `0.01` and new EPS is
`0.11`, the percentage change is 1000%, even though the economic change is ten
cents. If old EPS is `-0.10` and new EPS is `0.10`, a simple percentage formula
is not a stable measure of improvement.

Implemented scaled revision (the Phase 2.5 contract supersedes earlier proposals):

```text
scaled_revision = (new - old) / max(abs(old), abs(new))
both old and new zero -> 0
```

For `old=-0.10`, `new=0.10`, absolute change is `0.20`, scaled change is `2`.
There is no positive floor. Scaled EPS is not a percentage-growth measure; inspect
absolute change too. Consensus movement includes composition and provider corrections,
so it must be distinguished from economic revision activity.

## Revision Breadth

```text
7 contributors up, 2 down, 1 unchanged
breadth = (U-D)/(U+D) = 5/9
```

Use one latest comparable economic update per contributor within `(T-30d,T]`.
Latest UNCHANGED removes the directional vote. No directional updates means MISSING.
Revision count counts distinct UP/DOWN economic events, not contributors. NEW,
WITHDRAWAL and CORRECTION do not count or vote. Coverage must prove that a zero
count is real. See the frozen [estimate contract](phase-2.5-estimates-spec.md).

## Absolute Target Versus Relative Horizon

Stable identity:

```text
FY2027 Q2 EPS estimate
```

Relative horizon:

```text
FQ1
```

Before FY2027 Q1 actual is reported, `FQ1` might refer to FY2027 Q1. After Q1 is
reported and known, `FQ1` rolls to FY2027 Q2. The historical FY2027 Q1 consensus
must remain queryable; it simply stops qualifying as the next forward quarter.

---

# Part XIV - Peer And Industry Analysis - PLANNED

Peer-relative fundamentals can compare company features to industry features.

Example:

```text
company revenue growth = 40%
industry median revenue growth = 15%
relative growth = +25 percentage points
```

Possible interpretation: the company is gaining share or benefiting from a
product cycle.

Counterexample:

```text
company revenue growth = 10%
industry median revenue growth = 30%
relative growth = -20 percentage points
```

The company is growing, but it may be losing share.

Limitations: peer groups can be wrong, business mixes differ, and industry data
may be incomplete.

---

# Part XV - Factors - PLANNED

Definitions:

| Term | Meaning |
| --- | --- |
| Raw feature | One deterministic metric, such as revenue growth. |
| Factor | A combined dimension, such as growth acceleration. |
| Signal | A detector event, such as growth inflection. |
| Score | A normalized value used for ranking. |

Example factor inputs:

```text
Revenue Growth
Revenue Acceleration
EPS Growth
Margin Expansion
FCF Growth
```

These may feed:

```text
Growth / Acceleration Factor
```

Possible construction methods:

- weighted averages
- percentile ranks
- z-scores
- winsorization
- robust scaling
- sector-neutral scores

Z-score:

```text
z = (x - mu) / sigma
```

Symbols:

- `x`: company feature value.
- `mu`: cross-sectional average.
- `sigma`: cross-sectional standard deviation.

Example:

```text
x = 40% revenue growth
mu = 20%
sigma = 10%
z = (40% - 20%) / 10% = 2.0
```

Interpretation: the company is two standard deviations above the group average.

Limitation: z-scores are sensitive to outliers and unstable distributions.

Factor normalization must be PIT-correct. The peer universe and all feature
values must be known at T.

---

# Part XVI - Detector Signals - PLANNED

Detectors are explainable conditions, not arbitrary screens.

## Growth Inflection

Possible evidence:

```text
revenue growth rising
+ EPS growth rising
+ margin improving
```

## Estimate Revision Inflection

Possible evidence:

```text
consensus rising
+ revision breadth positive
+ revision count accelerating
```

## Market Confirmation

Possible evidence:

```text
stock outperforming SPY
+ stock outperforming sector
+ stock outperforming industry
+ above SMA200
+ positive SMA slope
```

## Capacity / Bottleneck

Structured evidence may include supply constraints, capacity additions, backlog,
or utilization.

## Catalyst

Catalysts may include earnings, product cycles, new customers, regulatory
decisions, or capacity ramps.

Signals should remain explainable. A user should be able to inspect every input.

---

# Part XVII - How Features May Be Combined - PLANNED

## Linear Weighted Score

```text
Score = w1*Growth + w2*Acceleration + w3*EstimateRevision + w4*Margin + w5*Momentum
```

Symbols:

- `w1...w5`: weights.
- each term: normalized factor value.

Weights can be equal, expert-defined, or statistically estimated. The risk is
that weights fitted on history may overfit.

## Rule-Based Detector

Example:

```text
Revenue acceleration > threshold
AND EPS acceleration > threshold
AND relative strength > threshold
```

This is interpretable but can be brittle near thresholds.

## Ranking Model

Rank companies cross-sectionally based on scores. Ranking is useful when capital
or attention is limited.

## Statistical Model

Regression or logistic models can estimate relationships between features and
future returns or future outperformance.

## Machine Learning

Trees or boosting can model nonlinear interactions. The project should start
simple and interpretable because complex models can hide leakage and overfitting.

---

# Part XVIII - Backtesting - PLANNED

At every historical timestamp T:

```text
1. reconstruct exactly what was known at T
2. compute features
3. generate signals
4. form ranking or portfolio
5. wait until executable next timestamp
6. measure future returns
```

Backtesting must handle:

- no future data
- train/validation/test separation
- walk-forward testing
- transaction costs
- slippage
- liquidity
- delisted stocks
- survivorship bias
- corporate actions
- rebalance frequency
- turnover

Definitions:

- in-sample: data used to design or fit the strategy.
- out-of-sample: data not used during design.
- rolling window: train on a moving historical window.
- expanding window: train on all data available up to T.
- walk-forward: repeatedly train on the past and test on the future.

---

# Part XIX - Evaluating Whether A Signal Works - PLANNED

Important metrics:

- average forward return
- median forward return
- hit rate
- excess return
- alpha
- information coefficient
- Spearman rank correlation
- Sharpe ratio
- Sortino ratio
- maximum drawdown
- turnover
- win/loss ratio

Sharpe ratio:

```text
Sharpe = (R_p - R_f) / sigma_p
```

Symbols:

- `R_p`: portfolio return.
- `R_f`: risk-free return.
- `sigma_p`: portfolio return volatility.

Example:

```text
portfolio return = 12%
risk-free return = 4%
volatility = 16%
Sharpe = (12% - 4%) / 16% = 0.50
```

Information coefficient:

```text
IC = corr(signal_t, future_return)
```

If the signal ranks stocks, Spearman rank correlation can be useful because it
tests rank ordering rather than exact linear magnitude.

Top-decile analysis:

```text
1. sort stocks by score
2. split into ten groups
3. compare future returns by group
```

A healthy signal often shows monotonicity: higher deciles perform better than
lower deciles.

---

# Part XX - Calibration - PLANNED

Raw detector output is not automatically a probability.

Calibration asks:

```text
When the historical score looked like this,
how often did the stock outperform?
```

Possible approaches:

- empirical hit-rate buckets
- isotonic regression
- logistic calibration
- Platt scaling

Example:

```text
Historical score bucket 80-90
Observed six-month outperformance rate = 63%
```

That does not mean a future stock has guaranteed 63% odds. Confidence intervals
matter, especially when the sample is small.

---

# Part XXI - Valuation - PLANNED

Growth alone is not enough. A great company can be a poor investment if the
price already assumes perfection.

Common formulas:

```text
MarketCap = SharePrice * SharesOutstanding
EV = MarketCap + Debt - Cash
P/E = SharePrice / EPS
EV/Sales = EnterpriseValue / Revenue
EV/EBITDA = EnterpriseValue / EBITDA
P/FCF = MarketCap / FreeCashFlow
FCFYield = FreeCashFlow / MarketCap
PEG = P/E / ExpectedGrowth
```

Example:

```text
Market cap = 100 billion
Debt = 10 billion
Cash = 20 billion
EV = 100 + 10 - 20 = 90 billion
Revenue = 30 billion
EV/Sales = 90 / 30 = 3.0x
```

Multiple expansion occurs when investors pay a higher valuation multiple.
Multiple compression occurs when they pay a lower multiple. A company can grow
earnings while the stock falls if the valuation multiple compresses enough.

---

# Part XXII - Thesis And Invalidation

The eventual system should produce:

```text
Investment Thesis
Evidence
Catalysts
Risks
Invalidation Conditions
```

Example thesis:

```text
Revenue acceleration, rising estimates, and market leadership indicate an early
AI infrastructure demand inflection.
```

Example invalidation conditions:

- revenue growth falls below a defined threshold
- estimate revisions turn negative
- gross margin deteriorates
- industry relative strength breaks
- guidance contradicts the demand thesis

Explicit invalidation prevents narrative attachment. The system should make it
hard to keep believing a thesis after the evidence changes.

---

# Part XXIII - AI Layer - PLANNED

AI should:

- summarize structured evidence
- explain changes
- identify conflicts
- produce readable research
- compare thesis and evidence
- summarize news and catalysts

AI must not:

- fabricate numbers
- silently calculate canonical metrics
- secretly rank stocks
- override PIT state
- invent missing evidence

The deterministic system comes first. AI is synthesis only.

---

# Part XXIV - Why Could This System Work?

The serious economic and behavioral rationale:

- earnings momentum: strong earnings trends can persist.
- analyst underreaction: analysts may revise gradually.
- information diffusion: investors process complex information at different
  speeds.
- institutional position building: large funds cannot build positions instantly.
- fundamental momentum: strong demand and operating leverage can persist.
- revision momentum: estimate upgrades can cluster.
- price momentum: relative strength can reflect improving expectations.
- quality: high margins and cash conversion can support durability.
- market leadership: leaders can attract capital and premium valuations.

Multiple independent confirmations can improve evidence quality:

```text
fundamentals improving
        +
analysts upgrading
        +
price outperforming
        +
industry supportive
```

This may provide stronger evidence, but incremental value must be validated;
correlated inputs are not independent confirmations.

Risks:

- crowding
- regime changes
- valuation excess
- reflexivity
- false positives
- macro shocks
- accounting distortions
- overfitting

The system does not guarantee alpha. It tries to reduce avoidable mistakes and
make evidence comparable across time and companies.

---

# Part XXV - Example Company Journey

Synthetic Company XYZ:

| Quarter | Revenue | YoY Growth | Gross Margin | EPS | EPS Consensus |
| --- | ---: | ---: | ---: | ---: | ---: |
| Q1 | 100 | 10% | 45% | 0.50 | 0.90 |
| Q2 | 125 | 25% | 48% | 0.75 | 1.00 |
| Q3 | 165 | 32% | 52% | 1.10 | 1.15 |

Market data:

```text
XYZ 63d return = +28%
SPY 63d return = +7%
Industry 63d return = +12%
```

Relative strength:

```text
Relative to SPY = 28% - 7% = +21 percentage points
Relative to industry = 28% - 12% = +16 percentage points
```

Revenue acceleration from Q2 to Q3:

```text
Q3 growth = 32%
Q2 growth = 25%
Acceleration = 32% - 25% = +7 percentage points
```

Gross margin change from Q1 to Q3:

```text
52% - 45% = +7 percentage points
```

EPS growth from Q2 to Q3:

```text
1.10 / 0.75 - 1 = 46.7%
```

Estimate revision:

```text
Same absolute FY2027 Q4 target, observed at three research dates:
T1 consensus = 0.90
T2 consensus = 1.00
T3 consensus = 1.15
Revision from T2 to T3 for the same target = +0.15
```

System path:

```text
raw SEC filing
    -> PIT filter by filed date
    -> atomic financial facts
    -> revenue, margin, EPS, FCF features
raw market prices and actions
    -> PIT split and dividend adjustment
    -> TRI
    -> momentum, relative strength, risk, volume features
raw estimate feed
    -> PIT estimate observations
    -> consensus and revision features
features
    -> growth, quality, momentum, revision factors
factors
    -> detector signals
signals
    -> backtested and calibrated ranking
ranking
    -> research snapshot
snapshot
    -> AI explanation
human
    -> investment decision
```

Interpretation: XYZ shows improving revenue, expanding gross margin, rising EPS,
rising consensus, and price leadership versus both market and industry. That is
a coherent evidence package. It still needs valuation, risk, catalyst quality,
and backtested calibration before becoming an investment decision.

---

# Part XXVI - How This System Can Fool Us

| Failure Mode | How It Fools Us | Mitigation |
| --- | --- | --- |
| Future leakage | Uses facts not known at T | Enforce `available_at <= T`. |
| Survivorship bias | Ignores failed/delisted companies | Historical universe with PIT membership. |
| Incorrect fiscal matching | Compares wrong quarters | Use fiscal identity, not calendar month. |
| YTD vs quarterly confusion | Six-month fact treated as one-quarter fact | Preserve period shape and mark ambiguity. |
| Split errors | False returns around splits | PIT split normalization and tests. |
| Duplicate observations | Inflated counts or consensus | Idempotent keys and contributor dedupe. |
| Stale prices | Old price treated as current | Stale-session policy. |
| Wrong peer group | Bad relative conclusions | Historical sector/industry membership. |
| Analyst identity duplicates | Same analyst counted twice | Canonical contributor identity. |
| Stale analyst consensus | Old estimates treated as fresh | Estimate availability and withdrawal handling. |
| GAAP/non-GAAP mixing | Apples-to-oranges EPS consensus | Basis included in grouping. |
| Currency mismatch | Revenue estimates in different currencies | Reporting-currency policy or explicit exclusion. |
| Overfitting | Strategy fits noise | Out-of-sample and walk-forward tests. |
| Data mining | Many tests find random winners | Multiple-testing controls and simplicity. |
| Small sample size | Unstable conclusions | Confidence intervals and robustness checks. |
| Regime dependency | Signal works only in one market environment | Regime analysis and long history. |
| Selection bias | Only studies remembered winners | Predefined universe and rules. |
| P-hacking | Repeated tuning until result looks good | Versioned hypotheses and holdout tests. |
| Transaction costs | Gross returns overstate net returns | Model costs, slippage, turnover. |
| Liquidity | Cannot trade signal size | Dollar-volume features and constraints. |
| Short history | Too few cycles | Treat evidence as provisional. |
| Provider revisions | History changes after the fact | Append-only raw objects and corrections. |
| AI hallucination | Narrative invents evidence | AI uses structured snapshot only. |

---

# Part XXVII - Project Phase Map

| Phase | Name | Status |
| --- | --- | --- |
| Phase 1 | Data Foundation | COMPLETE / ACCEPTED |
| Phase 2.1 | Temporal/PIT Foundation | COMPLETE / ACCEPTED |
| Phase 2.2 | Fundamental Features | COMPLETE |
| Phase 2.4 | Market Features | COMPLETE / FROZEN |
| Phase 2.5 | Analyst Estimates | IMPLEMENTATION COMPLETE / NOT YET FROZEN |
| Phase 2.6 | Peer / Industry Features | PLANNED |
| Phase 2.7 | Factors | PLANNED |
| Phase 2.8 | Signals / Detectors | PLANNED |
| Phase 2.9 | Historical Backtester | PLANNED |
| Phase 2.10 | Calibration / Ranking | PLANNED |
| Phase 2.11 | AI Analyst / deeper snapshot | PLANNED; valuation and evidence prerequisites |
| Phase 2.12 | News/event intelligence expansion | PROPOSED |
| Phase 2.13 | Business quality / source expansion | PROPOSED |

---

# Part XXVIII - Complete Data To Decision Map

```text
SEC FILINGS -------------------+
                               |
ANALYST ESTIMATES -------------+
                               |
MARKET PRICES -----------------+
                               |
CORPORATE ACTIONS -------------+
                               |
INDUSTRY DATA -----------------+
                               |
MACRO DATA --------------------+
                               |
NEWS / CATALYSTS --------------+
                               |
                               v
                         RAW DATA STORE
                               |
                               v
                        PIT TIME ENGINE
                               |
                               v
                        NORMALIZED FACTS
                               |
                               v
                            FEATURES
                               |
             +-----------------+-----------------+
             |                 |                 |
             v                 v                 v
          Growth            Quality          Momentum
             |                 |                 |
             +-----------------+-----------------+
                               |
                               v
                       ESTIMATE MOMENTUM
                               |
                               v
                        DETECTOR SIGNALS
                               |
                               v
                      BACKTEST / VALIDATE
                               |
                               v
                        CALIBRATED SCORE
                               |
                               v
                         STOCK RANKING
                               |
                               v
                       RESEARCH SNAPSHOT
                               |
                               v
                         AI EXPLANATION
                               |
                               v
                         HUMAN DECISION
```

The guiding principle is simple:

```text
Evidence first.
Time correctness always.
Deterministic calculations before interpretation.
AI last.
```

# Part XXIX - Closed-Loop Research, Validation and Promotion

This is the complete target lifecycle, not implemented runtime functionality:

```text
RAW DATA
↓
PIT DATA
↓
FEATURES
↓
FACTORS
↓
SIGNALS
↓
RESEARCH SNAPSHOT
↓
PREDICTION LEDGER
↓
FUTURE OUTCOMES
↓
OUTCOME LEDGER
↓
BACKTEST
↓
CALIBRATION
↓
CHALLENGER MODEL
↓
VALIDATION
↓
VERSIONED PROMOTION
```

Backtesting/calibration are development dependencies before a validated champion
is deployed. At runtime the champion produces snapshots without seeing future
outcomes. Evaluators later join matured labels through a separate data boundary.
Promotion changes future runs only. AI consumes the frozen snapshot for cited
synthesis; it is not a hidden source of truth, calculator or opaque ranking model.

The system may measure, evaluate, calibrate, train candidates and recommend changes.
It may NOT silently modify feature formulas, factor definitions, signal weights
or production models after a correct/incorrect outcome. Preserve every version,
trial and promotion decision. See [controlled learning](VALIDATION_CALIBRATION_AND_LEARNING.md).

## Canonical research packet and multiple objectives

A packet records research time, actual creation time, security, source/build and
feature/factor/signal/model/calibration versions, values/statuses, rank universe,
thesis/catalyst/risk/invalidation assertions and horizons. Predictions and outcomes
are append-only, including abstentions and failures. Legacy FeatureValue rows alone
are not an immutable ledger. See [ledger contract](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md).

Evaluate 3/6/12/24m absolute and SPY/sector/industry excess returns, forward drawdown,
risk-adjusted paths where justified, delivered growth/margins/FCF, future estimate
activity and assertion outcomes. Early growth, compounders, catalysts and risk
have different primary objectives. Predeclare those choices and report secondary
outcomes; do not optimize one arbitrary label or equate stock price with thesis truth.

## Evidence families and valuation/quality dependencies

Keep Fundamentals, Estimates, Market, Industry, Valuation, Catalysts, Business
Quality, Risk and Macro/Event Context separately inspectable. Missing families
remain visible. Test incremental contributions and double counting.

A named valuation workstream includes DCF/intrinsic value, FCF yield, EV/Revenue,
EV/EBITDA, economically valid P/E and PEG-style relationships, historical own bands,
peer/growth-adjusted valuation, scenarios and margin of safety. Assumptions and
share/enterprise-value bridges must be PIT and versioned. This is required before
Phase 2.11 presents a complete investment research judgment, even if detector
experiments precede it. No valuation module is implemented now.

Qualitative evidence includes economic moat, switching costs, network effects,
pricing power, cost/scale advantage, customer/supplier concentration, management
execution, capital allocation, R&D productivity, competitive intensity, TAM, market
share and product leadership. Combine structured evidence with cited AI interpretation;
unknown metrics cannot be invented. See [comparison and roadmap](RESEARCH_PLATFORM_COMPARISON.md).

## Source neutrality and canonical contracts

Replaceable connectors are intentional. Institutional prices and estimates,
transcripts, options, short interest, insider transactions, institutional ownership,
alternative data, supply-chain and industry data must map into canonical internal
contracts: stable identity, semantic basis/unit, timing, revisions, raw provenance,
coverage and legal rights. A vendor change creates a new profile/build, never silently
replaces historical evidence. Current legacy schemas do not yet provide every one
of these guarantees; the estimate subsystem is a stronger starting pattern.

Use [source trust](DATA_SOURCE_TRUST_AND_PIT.md) and the [risk register](DATA_PROVIDER_RISK_REGISTER.md)
for qualifications. FRED vintage-aware persistence is a required future task before
macro backtesting. Historical company classification needs knowledge time as well
as effective time before peer features. Yahoo historical endpoints are not a PIT
certificate. Generic raw storage and legacy feature updates require durability and
snapshot boundaries before immutable-history claims.

## Sequencing without casual renumbering

| Stage | Deliverable / release gate |
| --- | --- |
| Before more feature families | Independent 2.5 review; qualify R1–R4/R6–R7 foundation issues from strategic review; plan recoverability |
| 2.6 Peer / Industry | Bitemporal universe, identifier/classification evidence, delisted coverage; then peer statistics |
| 2.7 Factors | Inspectable/versioned normalization, missingness and component contributions; valuation workstream can run alongside |
| 2.8 Signals / detector | Rules first; minimal immutable snapshots and prediction publication before forward shadow claims |
| 2.9 Historical backtester | Qualified historical sources; outcome ledger, execution and label contracts; replay and broad-universe tests |
| 2.10 Calibration / evaluation | Mature labels, time-ordered walk-forward, overlap control, untouched holdout, drift and champion/challenger protocol |
| 2.11 AI Analyst (existing number retained) | Deeper snapshot and cited synthesis, gated on valuation/quality evidence and basic event/assertion contracts |
| 2.12 proposed extension | Production news/event intelligence with macro/policy exposure context and vintage data |
| 2.13 proposed extension | Expanded business-quality and external-source coverage, following measured need |

These are recommendations, not phase starts. Basic event/schema contracts must
precede their use by AI; the later event phase adds production coverage. If a future
roadmap changes phase numbers, record an explicit mapping and dependency decision.

## Generalization, operations and independent challenge

Historical replays freeze at T, then reveal future outcomes. Golden famous cases
are explanatory and contaminated by inspection; broad historical universes must
include failures and false positives. Unseen years, sectors, sizes and regimes,
plus prospective shadow results, are necessary before promotion. Use purging and
embargo concepts where label intervals overlap, respect maturity and log all trials.
See [historical validation](HISTORICAL_RESEARCH_VALIDATION_PLAN.md).

[News/event design](NEWS_AND_EVENT_INTELLIGENCE_PLAN.md) keeps factual events,
exposure mechanisms and uncertainty distinct from sentiment. Political context
remains neutral; no political ranking or desirability judgment is part of the system.
[Durability](DATA_DURABILITY_AND_RECOVERY.md) requires encrypted off-host database
and raw backups plus disposable restore drills, not data dumps committed to GitHub.
[External handoff](EXTERNAL_REVIEW_HANDOFF.md) asks reviewers to falsify the architecture.

# Part XXX - Business Quality and Strategic Intelligence

Status: FUTURE DESIGN. This extension does not start Phase 2.6, change frozen
formulas or close the strategic review's PIT/data-source remediation gates.
The full contract and public Morningstar study are in
[Qualitative and Alternative Research Architecture](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md).

```text
PIT source versions + evidence hierarchy + permissions
   ├─ BusinessQualityResearch (scoped company/segment dossier)
   ├─ CompanyStrategicEvents (typed occurrence and lifecycle assertions)
   ├─ ProfessionalResearchArtifact (attributed external views)
   └─ Alternative observations (qualified proxies, not assumed facts)
             ↓
claim corroboration + preserved disagreements + typed thesis graph
             ↓
frozen research snapshot / prediction ledger
             ↓
individual claim, risk and evidence-family outcome evaluation
```

These branches share canonical identity, availability, source/version provenance,
coverage and corrections with the numerical engine. They are not hidden AI scores.
Observed facts, management statements, analyst opinions, forecasts and scenarios
remain typed and distinguishable. Deterministic replay makes a recorded judgment
reproducible; it does not make subjective interpretation objectively true.

BusinessQualityResearch covers business model, product portfolio, revenue drivers,
customer base, geographies, competitive landscape, market share/TAM, pricing power,
switching costs, network effects, cost advantage, intangible assets/IP, efficient
scale, distribution advantage, ecosystem strength, R&D effectiveness, capital
intensity, customer/supplier concentration, management execution and capital allocation.
Store evidence, counter-evidence, confidence/reasons, source, available_at and history.
No scoring is implemented or authorized by this design.

Moat hypotheses use INTANGIBLE_ASSETS, SWITCHING_COSTS, NETWORK_EFFECT,
COST_ADVANTAGE and EFFICIENT_SCALE. Future evolution concepts are
MOAT_STRENGTHENING, MOAT_STABLE, MOAT_WEAKENING and UNKNOWN. Each needs scoped
comparative evidence and a versioned review rationale; silence is not stability.
LLM assignment alone is insufficient. We do not clone proprietary moat/star ratings.

CompanyStrategicEvents extends the existing event contract with products/generations,
customer/design wins, factories/capacity, geographies/partnerships, pricing/distribution,
M&A/divestitures, R&D, management/restructuring, debt/repurchases, customer losses,
supplier problems and competitor launches. Keep occurrence date separate from
publication/availability; announcement separate from completion; estimated economic
mechanism separate from observed results. These feed catalysts and invalidations.

Professional research uses provider-neutral versioned artifacts with thesis,
bull/bear factors, valuation assumptions, risks, moat and capital-allocation evidence,
analyst identity when permitted, and raw provenance/rights. Morningstar public
methodology is an intellectual reference. Owner-supplied legitimate subscriber
reports may be reviewed for their research within permissions; systematic access
requires licensing/API verification. No unauthorized scraping or redistribution.

Management claims connect to independent supporting and contradicting evidence.
Common-origin groups prevent a press release, its news copies and analyst echoes
from counting as independent confirmation. Disagreement between our system,
professional research, estimates, markets and management stays visible by matching
claim, scope and horizon. See [evidence hierarchy](RESEARCH_EVIDENCE_HIERARCHY.md)
and [alternative-data roadmap](ALTERNATIVE_DATA_ROADMAP.md).

The thesis graph freezes CLAIM, CAUSE, EXPECTED_OUTCOME, CATALYST, RISK,
INVALIDATION_CONDITION and EVIDENCE nodes and versioned edges. Expected outcomes
need explicit thresholds, fiscal targets, deadlines and adjudication policies.
Later labels ask which claim held, which risk occurred, whether external research
disagreed and whether alternative evidence confirmed management. Causal claims
are not proven merely by the eventual stock return.

Business quality, competitive advantage duration, reinvestment opportunity, ROIC,
growth, margins, cash flow and risk feed explicit DCF/scenario assumptions. This
complements market/estimate momentum. Keep fundamentals, market, estimates,
valuation, moat, news, professional and alternative evidence inspectable separately.
Any later combination must expose contributions/interactions and pass controlled
validation. No single unexplained AI score or silent production self-modification.

Existing phase numbering remains: snapshot contracts inform 2.8, outcome evaluation
2.9, controlled learning 2.10, deeper company research/synthesis 2.11, proposed
production events 2.12 and broader sources 2.13. This design is not permission to
skip foundation remediation or start those implementations.
