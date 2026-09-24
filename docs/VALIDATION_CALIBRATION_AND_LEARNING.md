# Validation, calibration and controlled learning

Status: FUTURE DESIGN. Start with rules and simple baselines. No ML framework,
production formula change, or automatic promotion is implemented here.

## Closed loop and responsibilities

```text
prediction ledger → outcome ledger → backtester → calibration analysis
→ model diagnostics → challenger version → validation → promotion decision
```

The prediction ledger freezes every decision and abstention. The outcome ledger
records mature labels without altering predictions. The backtester replays
historical decisions with pinned data, universes, costs and execution rules.
Calibration maps a score to empirical outcome frequencies for a named label and
horizon. Diagnostics examine why performance differs by cohort. Drift monitoring
raises research tickets. A version registry preserves the champion, challengers,
training manifests, dependencies and promotion/rejection history.

The system may measure, evaluate, calibrate, train candidate models and recommend
changes. It may NOT silently change feature formulas, factor definitions, signal
weights or production models. Every change, including a calibration mapping or
provider normalization change, requires a new version and validation gate.

## Dataset separation

| Set | Purpose | Restriction |
| --- | --- | --- |
| Training | Fit weights, thresholds, transforms or parameters on earlier data | Only labels already available by simulated training time |
| Validation | Choose between hypotheses and calibrate on later, separate data | Once inspected, it is development data, not untouched proof |
| True out-of-sample | Final chronological holdout never used for selection | Freeze candidate and success criteria before opening; reuse spends the holdout |
| Forward shadow | Champion/challenger predictions collected prospectively | Wait for enough labels to mature; do not choose end dates based on results |

Example time-ordered plan: train on eligible earlier years, validate on a later
block, then evaluate a locked candidate on another untouched block. At each
walk-forward origin, rebuild transforms using only earlier eligible training
records and **mature** outcomes. A prediction made in December whose 24m label
matures two years later cannot train a model at the next January boundary.
Exact years and minimum cohort sizes must be registered after coverage audit and
before outcome inspection, not retrofitted to whichever period worked.

Use expanding or rolling windows under a predeclared schedule. Cross-sectional
ranks at T may use the eligible contemporaneous universe; fitted global scaling,
feature selection and imputation cannot use future rows. Freeze unknown/missing
policies; do not impute from later reports. Keep all records for the same decision
time together when splitting, and test company-held-out cohorts as a separate
stress test against memorization.

## Overlapping labels and statistical dependence

A 12m label starting in June overlaps one starting in July. Random row splits
and ordinary independent-observation confidence intervals exaggerate evidence.
Purge training decisions whose label-information intervals overlap the evaluation
interval or were not mature at fit time. When the split construction permits
nearby observations to transfer information, embargo a documented interval around
the held-out block. Derive its length from label spans, release delays and shared
information windows; a magic fixed number of days is not a guarantee. Forward-only
splits have no post-test training block, but still require maturity and overlap checks.

Compute uncertainty with dependence-aware methods such as date-block resampling
and company clustering where justified. Count independent periods and overlapping
security observations separately. Do not confuse 10,000 correlated stock rows with
10,000 independent experiments. Test sensitivity to window and embargo choices
without selecting the best test result.

## Threats and controls

| Threat | Required control |
| --- | --- |
| Overfitting | Small hypothesis budget; simple baseline; frozen objective; complexity penalty and unseen-period validation |
| Lookahead bias | Verified availability for every input, universe and provider correction; source qualification before replay |
| Survivorship bias | Historical eligible names including failed, acquired and delisted companies; explicit coverage/terminal labels |
| Leakage | Separate outcome reads/writes and permissions from snapshot builder; as-of joins, fit-time manifests, no future text in historical synthesis |
| Multiple testing | Register all trials, rejected candidates and primary endpoints; report multiplicity-adjusted evidence or false-discovery analysis appropriate to the experiment |
| Regime dependence | Report bull/bear, high/low rates, recessions, sectors and size cohorts; poor regimes cannot be hidden by pooled averages |
| Selection bias | Score all eligible names, retain exclusions; compare coverage of missing vs observed outcomes |
| Backtest execution fantasy | Next executable timestamp, costs, slippage, liquidity, turnover, halted/delisted names and capacity assumptions |

Regime labels used as model inputs must themselves be known at T. Ex-post recession
or boom labels can be diagnostic slices only. Famous winners and repeatedly viewed
case studies are contaminated development material, never untouched validation.

## Calibration and diagnostics

Start with empirical score buckets and sample counts, observed hit rates and
confidence intervals for a named horizon/label. A bucket for 6m SPY excess return
cannot be reused as a probability of 24m fundamental success. Sparse buckets stay
uncalibrated. Consider logistic or monotonic calibration only after simple buckets
show stable support; fit on a separate chronological calibration sample. ML later
uses the same controls and must beat the simple reference under the same protocol.

Evaluate rank correlation, score-decile spread/monotonicity, median/mean outcomes,
coverage, downside, costs, turnover and objective-specific business outcomes.
For calibrated probabilities, inspect reliability plots, Brier score and log loss
where domains and sample size permit. Always show uncertainty, base rates and
cohort denominators. Risk-adjusted portfolio statistics require an actual portfolio
construction rule; stock labels alone cannot establish strategy Sharpe.

Ablate evidence families to detect duplicate information: Fundamentals, Estimates,
Market, Industry, Valuation, Catalysts, Business Quality, Risk, Macro/Event Context.
A giant opaque AI score is not the intended ranking architecture.

## Drift monitoring

| Monitor | Observe | Response |
| --- | --- | --- |
| Feature distributions | Quantiles, tails, missingness, units by sector/size | Investigate data or cohort change |
| Factor effectiveness | Mature-label rank correlations and decay | Open a challenger hypothesis |
| Signal hit rate | Calibration residuals, false positives/negatives | Review confidence and uncertainty |
| Sector drift | Universe composition and concentration | Check membership/provider changes |
| Market regime drift | Rates, liquidity, volatility and factor exposures | Evaluate predeclared regime diagnostics |
| Provider-data drift | Timestamp precision, revision rates, identities, coverage, schema | Quarantine suspect new data and notify operator |

Monitor on scheduled windows with predeclared thresholds and minimum observations.
Unmatured outcomes are not failures. Data incidents may halt affected publication
under an explicit operational policy; they do not authorize formula rewrites.

## Champion/challenger promotion contract

1. Preserve production champion vN, its formulas, artifacts and calibration.
2. Register challenger vN+1: hypothesis, exact versions, trial ID, objective families,
   minimum economically useful improvement, downside/coverage limits, sample and
   maturity requirements, multiple-testing budget and decision date.
3. Freeze inputs and train/validate in chronological order; compare with champion,
   no-signal/base-rate and simple factor baselines using identical eligible cohorts.
4. Lock candidate before opening the untouched test. Report all cohorts and failures.
5. Run both on future data without changing the champion. Wait for prespecified
   mature observations across independent dates; 24m claims require 24m evidence.
6. A recorded authorized promotion decision reviews uncertainty, robustness, costs,
   data rights, operational reliability and rollback readiness. Insufficient evidence
   means hold, not automatic promotion. No numerical gate is invented after results.
7. Deploy a new version with an effective timestamp. Retain old artifacts and every
   old prediction. Rollback restores an earlier version prospectively, never rewrites history.

See the [ledger](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md) and
[historical validation plan](HISTORICAL_RESEARCH_VALIDATION_PLAN.md). This is a
proposed evaluation protocol, not a claim of empirically validated alpha.

## Qualitative and alternative evidence evaluation

Use the extended [ledger](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md) to evaluate
individual claims, materialized risks, external disagreements and management
corroboration. Source quality, extraction confidence and outcome probability are
separate. Preserve source-version rights and exclude hindsight-contaminated reports.

Assess family contributions with matched claims/horizons, coverage controls,
common-origin dependence checks and held-out ablations. A panel change is not
business growth; an analyst echo is not independent corroboration. Report unknown
and not-comparable cases rather than choosing a narrative winner. New qualitative
rubrics, extraction policies or family combinations are new candidate versions,
subject to the same promotion gates. They cannot silently alter production models.
