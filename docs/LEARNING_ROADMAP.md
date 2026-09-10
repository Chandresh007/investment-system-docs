# Learning Roadmap for Building and Understanding the Investment Research System

This roadmap tells you what to learn, in what order, and how each topic maps to
the investment research system. It assumes basic investing and programming
knowledge, but not institutional quantitative research experience.

Depth labels:

- BASIC: enough to recognize and use the concept.
- INTERMEDIATE: enough to calculate, test, and debug it.
- DEEP: enough to design systems around it and detect subtle errors.

---

# Learning Module 1 - Accounting

Depth: DEEP

Why you need it: Phase 2.2 fundamental features are built from accounting facts.
If you do not understand the income statement, balance sheet, and cash-flow
statement, revenue growth and FCF metrics become just numbers without meaning.

Concepts to learn:

- income statement
- balance sheet
- cash-flow statement
- revenue
- COGS
- gross profit
- operating expenses
- operating income
- net income
- EPS
- diluted shares
- operating cash flow
- CapEx
- free cash flow
- margins
- operating leverage
- working capital

Math you need:

- percentage change
- ratios
- margins
- subtraction and sign conventions

Project usage:

- `financial_facts`
- `FundamentalFeatureRunner`
- `FundamentalCalculator`
- revenue growth, EPS growth, margins, FCF, FCF conversion

You should be able to explain:

- why revenue growth and EPS growth are different
- why FCF can differ from net income
- why gross margin and operating margin measure different things
- why working capital can make cash flow noisy

Suggested searches:

- `financial statements explained`
- `income statement balance sheet cash flow relationship`
- `free cash flow calculation`
- `operating leverage investing`
- `gross margin vs operating margin`
- `working capital cash flow explained`

---

# Learning Module 2 - SEC Filings / XBRL

Depth: DEEP

Why you need it: The project uses SEC Company Facts/XBRL as the foundation for
fundamental facts. The hard part is not downloading data; it is understanding
what each fact means and when it became knowable.

Concepts:

- 10-K
- 10-Q
- 8-K
- EDGAR
- XBRL
- SEC Company Facts
- accession number
- filing date
- fiscal period metadata
- taxonomy concepts
- units
- period start and period end

Math:

- date intervals
- fiscal period matching
- comparing same fiscal quarter across years

Project usage:

- `FinancialFact`
- `SECFactResolver`
- `PeriodResolver`
- migrations adding fiscal identity fields
- tests for fiscal ambiguity

You should be able to explain:

- why `fy/fp` is more authoritative than calendar month
- why Q2 YTD revenue is not the same as standalone Q2 revenue
- why a backtest must use filing date availability

Suggested searches:

- `SEC EDGAR tutorial`
- `XBRL company facts API`
- `SEC 10-K 10-Q explained`
- `XBRL tags us-gaap revenues`
- `fiscal year vs calendar year accounting`

---

# Learning Module 3 - Corporate Finance

Depth: INTERMEDIATE

Why you need it: Corporate finance gives context for why growth, margins, FCF,
and valuation matter.

Concepts:

- enterprise value
- market capitalization
- debt
- cash
- ROIC
- WACC conceptually
- operating leverage
- capital intensity
- reinvestment
- growth quality

Math:

- enterprise value formula
- return on invested capital
- weighted averages conceptually

Project usage:

- future valuation features
- future quality factors
- interpreting FCF and reinvestment

You should be able to explain:

- why two companies with the same revenue growth can have different quality
- why capital-light growth is often more valuable than capital-intensive growth
- why debt and cash matter to valuation

Suggested searches:

- `enterprise value explained`
- `ROIC explained investing`
- `WACC intuitive explanation`
- `capital intensive business model`
- `quality of growth investing`

---

# Learning Module 4 - Valuation

Depth: INTERMEDIATE

Why you need it: The system may find improving companies, but valuation tells
you how much investors are already paying for that improvement.

Concepts:

- P/E
- forward P/E
- EV/Sales
- EV/EBITDA
- price/FCF
- FCF yield
- DCF basics
- terminal value
- discount rates
- multiple expansion/compression
- growth-adjusted valuation

Math:

```text
P/E = Price / EPS
EV = MarketCap + Debt - Cash
EV/Sales = EV / Revenue
FCFYield = FCF / MarketCap
```

Project usage:

- planned valuation features
- ranking and thesis invalidation

You should be able to explain:

- why high growth can justify a high multiple but not an infinite multiple
- why a stock can fall even while EPS rises
- what multiple compression means

Suggested searches:

- `valuation multiples explained`
- `DCF model basics`
- `terminal value DCF explained`
- `EV EBITDA vs PE`
- `multiple compression growth stocks`

---

# Learning Module 5 - Basic Statistics

Depth: DEEP

Why you need it: The system uses averages, medians, standard deviations,
percentiles, z-scores, correlations, and eventually backtest statistics.

Concepts:

- mean
- median
- variance
- standard deviation
- sample vs population variance
- covariance
- correlation
- percentile
- quantiles
- z-scores
- distributions
- outliers

Project usage:

- analyst consensus mean/median/stddev
- market volatility
- factor normalization
- signal evaluation

You should be able to explain:

- why median can be more robust than mean
- why sample standard deviation divides by `n - 1`
- why z-scores allow differently scaled metrics to be compared
- why outliers can distort averages

Suggested searches:

- `statistics mean median variance standard deviation explained`
- `sample vs population standard deviation`
- `z score explained`
- `correlation covariance explained`
- `percentiles quantiles explained`

---

# Learning Module 6 - Returns

Depth: DEEP

Why you need it: Market features are based on returns, especially log returns.

Concepts:

- simple returns
- log returns
- cumulative returns
- compounding
- annualization
- excess returns

Formulas:

```text
SimpleReturn = P_t / P_0 - 1
LogReturn = ln(P_t / P_0)
ExcessReturn = StockReturn - BenchmarkReturn
```

Project usage:

- all `return_*` features
- relative strength
- industry basket returns
- backtest performance

You should be able to explain:

- why returns are scale-independent
- why log returns add over time
- why excess return gives context

Suggested searches:

- `simple returns vs log returns`
- `cumulative return calculation`
- `annualized return explained`
- `excess return investing`

---

# Learning Module 7 - Risk

Depth: INTERMEDIATE

Why you need it: A stock with high return and extreme drawdown may not be better
than a steadier stock with lower return.

Concepts:

- volatility
- downside volatility
- Sharpe ratio
- Sortino ratio
- drawdown
- beta
- alpha

Project usage:

- `realized_vol_63`
- `max_dd_252`
- `vol_regime`
- future backtest metrics

You should be able to explain:

- why volatility is not the same as permanent loss
- why drawdown is path-dependent
- why Sharpe ratio penalizes volatility

Suggested searches:

- `volatility investing explained`
- `Sharpe ratio explained`
- `Sortino ratio explained`
- `maximum drawdown calculation`
- `alpha beta investing explained`

---

# Learning Module 8 - Time Series

Depth: DEEP

Why you need it: Phase 2.4 market features depend on rolling windows, lags,
moving averages, momentum, and stale observations.

Concepts:

- lags
- rolling windows
- moving averages
- momentum
- trend
- autocorrelation
- stationarity conceptually
- regime changes

Project usage:

- `MarketPriceResolver`
- `previous_sessions`
- exact observation windows
- SMA and volatility features

You should be able to explain:

- why a 21-day return needs 22 prices
- why stale data must be detected
- why rolling windows are sensitive to missing observations

Suggested searches:

- `time series rolling window explained`
- `moving average finance explained`
- `momentum investing explained`
- `autocorrelation time series intuitive`
- `regime change finance`

---

# Learning Module 9 - Cross-Sectional Quant Research

Depth: INTERMEDIATE

Why you need it: Future ranking compares many companies at the same point in
time.

Concepts:

- ranking
- percentile ranks
- z-scores
- winsorization
- sector neutralization
- factor exposure
- factor portfolios
- decile analysis

Project usage:

- future factors
- future ranking
- future backtests

You should be able to explain:

- why cross-sectional ranking differs from time-series analysis
- why sector neutralization can matter
- why top-decile analysis is useful

Suggested searches:

- `cross sectional stock ranking quant`
- `factor z score winsorization`
- `sector neutral factor investing`
- `decile analysis factor returns`

---

# Learning Module 10 - Factor Investing

Depth: INTERMEDIATE

Why you need it: The project will combine raw features into factors like growth,
quality, value, and momentum.

Concepts:

- momentum
- value
- quality
- growth
- profitability
- investment factor
- earnings momentum
- Fama-French factor models conceptually

Project usage:

- planned Phase 2.7 factors
- planned scoring and calibration

You should be able to explain:

- how traditional factors differ from this custom growth-inflection engine
- why factors require clean definitions and PIT data
- why factor returns vary by regime

Suggested searches:

- `factor investing explained`
- `Fama French five factor model explained`
- `quality factor investing`
- `earnings momentum factor`
- `growth investing quantitative signals`

---

# Learning Module 11 - Analyst Estimates

Depth: DEEP

Why you need it: Phase 2.5 is planned around analyst EPS/revenue estimates,
consensus, revisions, breadth, and forward fiscal periods.

Concepts:

- sell-side estimates
- consensus EPS
- consensus revenue
- estimate revisions
- breadth
- dispersion
- earnings surprise
- guidance
- whisper expectations
- withdrawals
- provider corrections
- GAAP vs non-GAAP basis
- absolute fiscal target versus relative horizon

Project usage:

- Phase 2.5 DESIGNED / NOT IMPLEMENTED
- future estimate momentum features
- future earnings surprise work

You should be able to explain:

- why consensus uses the latest estimate per contributor
- why new analyst initiations are not revisions
- why `FQ1` is not a stable identity
- why negative EPS makes percentage revisions hard

Suggested searches:

- `analyst estimate revisions investing`
- `consensus EPS explained`
- `earnings surprise estimate revision breadth`
- `I/B/E/S estimates methodology overview`
- `GAAP vs non GAAP EPS estimates`

---

# Learning Module 12 - Backtesting

Depth: DEEP

Why you need it: Backtesting is how the system will test whether signals worked
historically. This is also where many false strategies are created by accident.

Concepts:

- look-ahead bias
- survivorship bias
- data snooping
- leakage
- train/test
- walk-forward validation
- transaction costs
- slippage
- portfolio turnover
- rebalance assumptions
- delisted stocks

Project usage:

- future Phase 2.9 backtester
- all PIT architecture
- market corporate-action handling

You should be able to explain:

- why a backtest must reconstruct exactly what was known at T
- why today's universe cannot be used for 2018
- why trading costs can destroy paper alpha

Suggested searches:

- `backtesting look ahead bias`
- `survivorship bias quantitative finance`
- `walk forward validation trading strategy`
- `transaction costs slippage backtest`
- `data snooping bias finance`

---

# Learning Module 13 - Research Statistics

Depth: DEEP

Why you need it: Finding a pattern once proves almost nothing. Research
statistics helps decide whether results are likely real or noise.

Concepts:

- null hypothesis
- statistical significance
- p-values conceptually
- confidence intervals
- multiple testing
- false discovery
- effect size
- bootstrap
- robustness tests

Project usage:

- future signal evaluation
- calibration
- ranking validation

You should be able to explain:

- why p-values are not proof
- why multiple testing creates false discoveries
- why effect size matters more than just significance

Suggested searches:

- `statistical significance p value explained`
- `confidence interval intuitive explanation`
- `multiple testing false discovery rate`
- `bootstrap statistics explained`
- `effect size vs p value`

---

# Learning Module 14 - Machine Learning

Depth: BASIC to INTERMEDIATE, later

Why you need it: ML may eventually help combine features, but jumping here too
early can hide data leakage and overfitting.

Concepts:

- linear regression
- logistic regression
- decision trees
- random forests
- gradient boosting
- regularization
- feature importance
- cross-validation

Project usage:

- possible future modeling
- calibration and ranking experiments

You should be able to explain:

- why cross-validation must respect time
- why feature importance can mislead
- why simple models are often better early in research

Suggested searches:

- `linear regression explained`
- `logistic regression explained`
- `decision trees random forest gradient boosting`
- `regularization machine learning explained`
- `time series cross validation finance`

---

# Learning Module 15 - Database / Data Engineering

Depth: DEEP

Why you need it: The system is a data product as much as a research model.
Correct primary keys, indexes, immutability, and migrations make the research
reproducible.

Concepts:

- PostgreSQL
- primary keys
- foreign keys
- indexes
- uniqueness
- transactions
- idempotency
- migrations
- temporal data
- immutable raw storage
- JSON provenance

Project usage:

- SQLAlchemy models
- Alembic migrations
- `raw_data_objects`
- `feature_values`
- uniqueness constraints

You should be able to explain:

- why raw data is immutable
- why uniqueness constraints prevent false duplicates
- why indexes matter for PIT queries
- why migrations are versioned

Suggested searches:

- `PostgreSQL indexes explained`
- `database primary key foreign key unique constraint`
- `Alembic migrations SQLAlchemy`
- `idempotent data pipeline`
- `temporal database design`

---

# Learning Module 16 - Point-In-Time Systems

Depth: DEEP

Why you need it: PIT correctness is the central architectural invariant.

Concepts:

- event time
- available time
- retrieval time
- research snapshot time
- bitemporal databases
- historical reconstruction
- data corrections

Key rule:

```text
available_at <= T
```

Project usage:

- SEC filed dates
- market retrieved times
- corporate action availability
- future estimate event availability
- historical universe membership

You should be able to explain:

- why event time and available time are different
- why provider corrections must be append-only
- why fake historical consensus is forbidden

Suggested searches:

- `point in time financial database`
- `look ahead bias quantitative finance`
- `bitemporal database`
- `event time processing`
- `temporal database design`

---

# Learning Module 17 - Software Testing For Quant Systems

Depth: DEEP

Why you need it: Numerical systems often fail silently. Tests must include
adversarial edge cases, not just happy paths.

Concepts:

- unit tests
- integration tests
- deterministic fixtures
- adversarial tests
- property testing
- regression tests
- golden tests
- independent expected-value calculations

Project usage:

- `tests/unit/test_fundamental_features.py`
- `tests/unit/test_market_features.py`
- `tests/unit/test_market_runner_fixes.py`
- PIT leak tests
- idempotency tests

You should be able to explain:

- why tests should prove future data does not leak
- why expected values should be calculated independently
- why edge cases around zero and signs matter

Suggested searches:

- `unit testing data pipelines`
- `property based testing Python hypothesis`
- `golden tests explained`
- `testing quantitative finance systems`
- `pytest fixtures tutorial`

---

# Learning Module 18 - Portfolio Construction

Depth: INTERMEDIATE, later

Why you need it: A good signal is not yet a portfolio. Position sizing and risk
constraints determine actual investment outcomes.

Concepts:

- position sizing
- diversification
- concentration
- volatility targeting
- sector limits
- liquidity constraints
- turnover
- risk budgeting

Project usage:

- future portfolio/backtesting layer

You should be able to explain:

- why ranking top stocks is not the same as building a portfolio
- why liquidity and concentration limits matter
- why turnover creates costs

Suggested searches:

- `portfolio construction basics`
- `position sizing investing`
- `volatility targeting strategy`
- `portfolio turnover transaction costs`
- `risk budgeting portfolio`

---

# Learning Module 19 - Market Microstructure

Depth: BASIC to INTERMEDIATE

Why you need it: Backtests assume trades can be executed. Real markets have
spreads, slippage, and market impact.

Concepts:

- bid/ask
- spread
- market orders
- limit orders
- liquidity
- slippage
- volume
- market impact

Project usage:

- future execution assumptions
- liquidity filters using dollar volume

You should be able to explain:

- why high spread increases costs
- why large orders move prices
- why dollar volume matters

Suggested searches:

- `bid ask spread explained`
- `market orders limit orders explained`
- `slippage trading explained`
- `market impact liquidity`
- `dollar volume liquidity investing`

---

# Learning Module 20 - AI For Investment Research

Depth: INTERMEDIATE

Why you need it: The project uses AI for synthesis, not deterministic evidence
generation.

Concepts:

- embeddings
- RAG
- structured retrieval
- citations
- tool use
- hallucination
- deterministic computation versus probabilistic language models

Project usage:

- future AI analyst phase
- research snapshot synthesis

You should be able to explain:

- why AI should not fabricate numbers
- why structured evidence should come before narrative
- why citations and provenance matter

Suggested searches:

- `retrieval augmented generation explained`
- `LLM hallucination mitigation`
- `AI tool use structured data`
- `AI for financial research risks`
- `deterministic computation vs LLM`

---

# Final Learning Order

```text
Stage 1: Accounting + SEC + valuation
Stage 2: Statistics + returns + risk
Stage 3: Time series + momentum + factors
Stage 4: Analyst estimates
Stage 5: PIT databases + data engineering
Stage 6: Backtesting + research statistics
Stage 7: Portfolio construction
Stage 8: Machine learning
Stage 9: AI research synthesis
```

---

# 30-Day Curriculum

| Week | Subjects | Search / Read | Formulas | Repository Inspection | Hand Exercise |
| --- | --- | --- | --- | --- | --- |
| 1 | Accounting and SEC filings | `financial statements explained`, `SEC Company Facts API` | margins, FCF | `models/financial_fact.py`, `normalization/sec.py` | Compute revenue growth, gross margin, FCF from a small income statement. |
| 2 | PIT and fiscal identity | `look ahead bias quantitative finance`, `fiscal year vs calendar year` | `available_at <= T` | `research/resolver.py`, fiscal tests | Given five filings, decide which facts are visible on three dates. |
| 3 | Returns, momentum, market features | `log returns`, `momentum investing`, `moving average finance` | log return, SMA, relative return | `research/market_runner.py`, Phase 2.4 spec | Compute 5-day log return and price versus SMA by hand. |
| 4 | Risk, volume, estimates intro | `standard deviation explained`, `analyst revisions investing` | sample stddev, Sharpe, consensus mean | market tests, this blueprint Part XIII | Compute consensus mean/median and a scaled revision. |

---

# 90-Day Curriculum

| Week | Subjects | Search / Read | Formulas | Repository Inspection | Hand Exercise |
| --- | --- | --- | --- | --- | --- |
| 1 | Financial statements | `income statement balance sheet cash flow relationship` | margins | `financial_fact.py` | Link income statement and cash-flow statement items. |
| 2 | FCF and operating leverage | `free cash flow calculation`, `operating leverage investing` | FCF, FCF margin | `FundamentalCalculator` | Compute FCF conversion for three scenarios. |
| 3 | SEC/XBRL | `XBRL company facts API` | fiscal comparisons | SEC normalization/tests | Identify why a calendar quarter rule fails for NVIDIA-style fiscal years. |
| 4 | PIT systems | `bitemporal database`, `look ahead bias` | `available_at <= T` | `FeatureValue`, `raw_data_objects` | Build a timeline of event, filing, retrieval, and research times. |
| 5 | Basic statistics | `sample vs population standard deviation` | mean, median, variance | market calculator | Compute sample volatility manually. |
| 6 | Returns | `simple vs log returns` | log returns, cumulative returns | market runner returns | Show why log returns add. |
| 7 | Market adjustment | `stock split adjustment`, `total return index` | split adjustment, TRI | adjustment engine | Normalize a split and dividend example. |
| 8 | Momentum/relative strength | `relative strength investing` | stock return - benchmark return | benchmark resolver | Compute stock vs industry relative return. |
| 9 | Analyst estimates | `consensus EPS explained`, `estimate revisions` | mean, median, scaled revision | Phase 2.5 review concepts | Classify revisions, initiations, withdrawals. |
| 10 | Backtesting | `walk forward validation trading strategy` | forward returns | tests and project state | Design a PIT-correct rebalance timeline. |
| 11 | Research statistics | `multiple testing false discovery rate` | IC, hit rate | planned docs | Explain why testing 100 signals creates false discoveries. |
| 12 | Factors and ranking | `factor z score winsorization` | z-score, percentile | feature registry | Convert three raw features into ranks. |
| 13 | Integration review | revisit all modules | all core formulas | README + blueprint | Walk a synthetic company from raw data to research thesis. |

---

# 6-Month Curriculum

| Month | Focus | Outcome |
| --- | --- | --- |
| 1 | Accounting, SEC, PIT basics | Understand accepted fundamental architecture. |
| 2 | Statistics, returns, market math | Understand frozen Phase 2.4 features. |
| 3 | Analyst estimates and data engineering | Understand Phase 2.5 design risks. |
| 4 | Backtesting and research statistics | Design leakage-resistant tests. |
| 5 | Factors, ranking, valuation | Understand how evidence may become stock selection. |
| 6 | Portfolio construction, ML, AI synthesis | Understand later phases and why AI stays last. |

Monthly exercises:

- Month 1: reproduce fundamental feature formulas by hand.
- Month 2: reproduce market feature formulas by hand.
- Month 3: design an estimate consensus table and PIT examples.
- Month 4: write a paper backtest design with leakage checks.
- Month 5: build a conceptual factor score and valuation overlay.
- Month 6: write a research snapshot and identify what AI may and may not say.

---

# Mathematics Appendix

## Percentage Change

Formula:

```text
percentage_change = new / old - 1
```

Symbols:

- `new`: current value.
- `old`: prior value.

Example:

```text
new = 125
old = 100
change = 125 / 100 - 1 = 25%
```

Meaning: how much a value increased or decreased relative to its starting point.

Project usage: revenue growth, EPS growth, operating income growth.

Limitation: breaks when `old = 0`; can be hard to interpret when signs change.

## Log Returns

Formula:

```text
r = ln(P_t / P_0)
```

Example:

```text
P_t = 110
P_0 = 100
r = ln(1.10) = 0.0953
```

Meaning: continuously compounded return.

Project usage: Phase 2.4 market returns and relative strength.

Limitation: requires positive prices.

## Mean

Formula:

```text
mean = sum(x_i) / n
```

Example:

```text
values = 2, 4, 6
mean = 12 / 3 = 4
```

Project usage: consensus estimates, dollar-volume averages.

Limitation: sensitive to outliers.

## Median

Formula:

```text
middle value after sorting
```

Example:

```text
2, 4, 100 -> median = 4
```

For even counts:

```text
2, 4, 6, 8 -> median = (4 + 6) / 2 = 5
```

Project usage: planned consensus estimates.

Limitation: ignores magnitude of extreme values.

## Variance

Sample variance:

```text
s^2 = sum((x_i - x_bar)^2) / (n - 1)
```

Example:

```text
values = 2, 4, 6
mean = 4
s^2 = ((2-4)^2 + (4-4)^2 + (6-4)^2) / 2 = 8 / 2 = 4
```

Project usage: volatility and dispersion.

Limitation: squared deviations emphasize outliers.

## Sample Standard Deviation

Formula:

```text
s = sqrt(s^2)
```

Example:

```text
variance = 4
standard deviation = 2
```

Project usage: `realized_vol_63`, planned consensus dispersion.

## Covariance

Formula:

```text
cov(x, y) = sum((x_i - x_bar)(y_i - y_bar)) / (n - 1)
```

Meaning: whether two variables move together.

Example: if stock returns and market returns are both above average on the same
days, covariance is positive.

Project usage: future beta, factor risk, and research statistics.

## Correlation

Formula:

```text
corr(x, y) = cov(x, y) / (std(x) * std(y))
```

Example:

```text
cov = 0.006
std(x) = 0.10
std(y) = 0.20
corr = 0.006 / (0.10 * 0.20) = 0.30
```

Project usage: information coefficient and future factor analysis.

Limitation: linear correlation can miss nonlinear relationships.

## Z-Score

Formula:

```text
z = (x - mu) / sigma
```

Example:

```text
x = 40
mu = 20
sigma = 10
z = 2
```

Meaning: `x` is two standard deviations above average.

Project usage: planned factor normalization.

Limitation: unstable when distributions are skewed or `sigma` is tiny.

## Percentile

Formula concept:

```text
percentile_rank = fraction of observations below x
```

Example:

```text
If 80 out of 100 companies have lower revenue growth, percentile = 80th.
```

Project usage: planned cross-sectional ranking.

Limitation: loses magnitude information.

## Linear Regression

Formula:

```text
y = alpha + beta*x + error
```

Example:

```text
future_return = alpha + beta * revision_score + error
```

Meaning: estimates the average linear relationship between `x` and `y`.

Project usage: possible future research models.

Limitation: correlation is not causation; leakage and overfitting are common.

## Logistic Function

Formula:

```text
p = 1 / (1 + exp(-z))
```

Example:

```text
z = 0
p = 1 / (1 + 1) = 0.5
```

Meaning: maps any real score to a probability-like value between 0 and 1.

Project usage: possible future calibration.

Limitation: probability interpretation requires calibration.

## Sharpe Ratio

Formula:

```text
Sharpe = (R_p - R_f) / sigma_p
```

Example:

```text
R_p = 12%
R_f = 4%
sigma_p = 16%
Sharpe = 0.50
```

Project usage: future backtest evaluation.

Limitation: penalizes upside and downside volatility equally.

## Drawdown

Formula:

```text
DD_t = P_t / Peak_t - 1
```

Example:

```text
Peak = 120
Current = 90
DD = 90 / 120 - 1 = -25%
```

Project usage: `max_dd_252` and future portfolio risk.

## CAGR

Formula:

```text
CAGR = (EndingValue / BeginningValue)^(1 / years) - 1
```

Example:

```text
100 grows to 121 over 2 years
CAGR = (121 / 100)^(1/2) - 1 = 10%
```

Project usage: future long-horizon backtest summaries.

## Discounted Cash Flow

Present value formula:

```text
PV = CF_t / (1 + r)^t
```

Example:

```text
cash flow in one year = 110
discount rate = 10%
PV = 110 / 1.10 = 100
```

Project usage: future valuation education and possible valuation features.

Limitation: highly sensitive to growth and discount-rate assumptions.

## Present Value

Formula:

```text
PV = FutureValue / (1 + r)^t
```

Meaning: what future money is worth today.

Example:

```text
FutureValue = 121
r = 10%
t = 2
PV = 121 / 1.1^2 = 100
```

## Weighted Average

Formula:

```text
weighted_average = sum(w_i * x_i) / sum(w_i)
```

Example:

```text
values = 10, 20
weights = 1, 3
weighted_average = (1*10 + 3*20) / 4 = 17.5
```

Project usage: possible future factor scores and portfolio weights.

## Geometric Compounding

Formula:

```text
EndingValue = BeginningValue * product(1 + r_i)
```

Example:

```text
Start = 100
Year 1 return = 10%
Year 2 return = -10%
Ending = 100 * 1.10 * 0.90 = 99
```

Meaning: returns compound multiplicatively.

Project usage: backtests and portfolio performance.

Limitation: average simple return can misstate compounded outcome.
