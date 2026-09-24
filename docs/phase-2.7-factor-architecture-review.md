# Phase 2.7 factor architecture review

Review date: 2026-09-22. Approved starting checkpoint:
`142caf3bd5c6fc6ee71a46330379f709c72df099` on `master`, equal to
`origin/master` with a clean worktree before this documentation review. Baseline:
**513 passed, 8 existing warnings, zero failures**; Alembic current/head
**`p26008`**.

This is an architecture and implementation contract, not a Phase 2.7
implementation. It adds no application code, model, migration, factor row, source
data, signal, rank, backtest, or production qualification. Foundation and Phases
2.1, 2.2, 2.4, 2.5, and 2.6 retain their frozen formulas and temporal semantics.

## A. Verdict

**CONDITIONAL PASS**

The design is sufficiently exact to implement Phase 2.7. The conditions are
release and evidence gates rather than unresolved factor mathematics:

- Every v1 definition starts as a **candidate**, not an empirically validated
  production factor. The fixed transforms and weights below are transparent
  economic priors. Famous winners were not used to choose them.
- A factor may claim only the weakest quality supported by its exact included
  inputs and factor-specific provider admission. Current synthetic estimates and
  peer evidence therefore force affected factors to `DEVELOPMENT`.
- Implementation must use the exact frozen source IDs, exact source builds, and
  PIT selection contracts below. It may not repair, reinterpret, or recompute a
  frozen Phase 2.2, 2.4, 2.5, or 2.6 formula.
- Factor persistence must be semantically separate from `ResearchFeature` /
  `FeatureValue`, relationally link every component to its exact source artifact,
  and be append-only and idempotent.
- Phase 2.7 stops at inspectable evidence-family values. A factor is not a signal,
  universe rank, probability, recommendation, thesis, or permission to start
  Phase 2.8.

Subject to those conditions, section R recommends **SAFE TO IMPLEMENT PHASE
2.7**.

## B. Phase 2.7 scope

### In scope

- An immutable, explicit registry of nine economically distinct factor families.
- Named subfactors that preserve absolute, peer-relative, trend, revision,
  industry, and risk meanings instead of averaging unrelated inputs.
- Fixed, bounded, monotonic component transforms into `[-1,+1]`.
- Exact component weights, required anchors, group coverage, missingness,
  direction, orientation, and quality rules.
- PIT retrieval from exact fundamental, market, estimate, peer, and industry
  builds pinned by `ResearchBuildManifest`.
- Dedicated factor definitions, values, subfactor values, and relational
  component lineage.
- On-demand immutable factor snapshots, batch execution, deterministic reruns,
  and future backtest-compatible history.
- Candidate-versus-production identity and a future append-only promotion hook.

### Explicitly out of scope

- Buy, sell, watch, avoid, or other detector decisions.
- Conjunctive operating-leverage, growth-inflection, market-confirmation, or
  estimate-inflection signals. Those are Phase 2.8 rules over factors and their
  components.
- A universe-wide factor percentile, stock rank, composite stock score, portfolio,
  position size, expected return, confidence, or calibrated probability.
- Backtesting, outcome labels, weight fitting, ML, automatic learning, or champion
  promotion.
- Valuation, business-quality, moat, management, customer-concentration,
  geopolitical, news, sentiment, catalyst, or alternative-data factors.
- New source features, inferred metrics, current-value fallback, missing-value
  imputation, or changes to frozen source formulas.
- Daily full-universe materialization. Phase 2.7 computes only requested snapshots
  and persists them immutably.
- External provider or production qualification. Deterministic calculation does
  not cure an unqualified source.

### Phase boundary

```text
raw facts
  -> frozen deterministic features and industry metrics
  -> Phase 2.7 factors (this design)
  -> Phase 2.8 signals / detectors
  -> later score calibration and universe ranking
  -> research thesis and cited synthesis
```

Cross-sectional peer percentiles from Phase 2.6 are legitimate lower-level
features. They are not a final universe rank. Phase 2.7 introduces no new
cross-sectional ranking operation.

## C. Exact factor definition and frozen terminology

| Term | Frozen meaning |
| --- | --- |
| **Raw fact** | A provider-derived atomic observation with identity, units, timing, and provenance, such as a filing fact, estimate event, or market-price observation. |
| **Feature** | One deterministic measured property under one frozen formula and source contract, persisted as a `FeatureValue` where applicable. Examples are `revenue_yoy_growth` and `return_63d`. A feature is not required to be desirable; `realized_vol_63` is descriptive. |
| **Factor** | A versioned, inspectable economic evidence family built deterministically from related, frozen lower-level artifacts. It has explicit transforms, weights, coverage, orientation, component contributions, quality, and lineage. A factor may summarize evidence strength; it does not make an investment decision. |
| **Signal** | A versioned detector result over factors, features, states, or trajectories, such as “positive growth with margin expansion and improving revisions.” A signal records the rule/model and every trigger. Signals begin in Phase 2.8. |
| **Score** | A generic scalar representation, not a semantic layer by itself. Every use must be qualified, such as `factor_value`, `signal_strength`, or later `ranking_score`. A Phase 2.7 factor value is an evidence-strength score only. |
| **Rank** | An ordinal or percentile computed across an explicitly identified PIT universe at T, with its denominator and tie policy. It is separate from a factor value. Phase 2.6 peer percentiles rank one raw metric inside one formal peer group; they are still features, not an overall stock rank. |
| **Thesis** | A versioned research claim or claim graph with evidence, counter-evidence, catalysts, risks, invalidation conditions, scope, horizon, and citations. It may consume factors later but cannot be reduced to one unexplained factor score. |

A factor value of `0.82` means strong positive evidence on that factor definition's
bounded scale. It does **not** mean an 82% probability of success, an expected 82%
return, an 82nd percentile universe rank, or a buy recommendation. Probability
calibration belongs to a later phase and must name a label, horizon, cohort, and
calibration version.

## D. V1 factor registry

The smallest valuable v1 registry contains nine top-level factors:

| Public registry key | Logical `factor_id` / version | Subject | Orientation | Economic question |
| --- | --- | --- | --- | --- |
| `growth_v1` | `growth` / `1` | Security | `POSITIVE_EVIDENCE` | Is current business and earnings growth strong in absolute and peer-relative terms? |
| `growth_acceleration_v1` | `growth_acceleration` / `1` | Security | `POSITIVE_EVIDENCE` | Is the rate of growth improving, separately from its current level? |
| `profitability_v1` | `profitability` / `1` | Security | `POSITIVE_EVIDENCE` | Are current gross and operating economics strong in absolute and peer-relative terms? |
| `margin_expansion_v1` | `margin_expansion` / `1` | Security | `POSITIVE_EVIDENCE` | Are gross and operating margins improving? |
| `cash_flow_quality_v1` | `cash_flow_quality` / `1` | Security | `POSITIVE_EVIDENCE` | Are earnings and growth accompanied by cash generation? |
| `estimate_revision_v1` | `estimate_revision` / `1` | Security | `POSITIVE_EVIDENCE` | Are near- and medium-horizon EPS and revenue expectations becoming more positive? |
| `market_leadership_v1` | `market_leadership` / `1` | Security | `POSITIVE_EVIDENCE` | Is the stock in a positive trend and leading relevant benchmarks and peers? |
| `industry_strength_v1` | `industry_strength` / `1` | Industry node | `POSITIVE_EVIDENCE` | Is the exact historical sub-industry showing broad fundamental, estimate, and market support? |
| `market_fragility_v1` | `market_fragility` / `1` | Security | `RISK_HIGHER_WORSE` | Is observed price-path fragility elevated? |

All nine start as `CANDIDATE`. None has validated predictive meaning at this
checkpoint.

### Deliberate omissions

**No omnibus `peer_leadership_v1`.** Averaging growth, margins, revisions, and
returns into one “stronger than peers” scalar would mix different economic
questions and duplicate the relative channel inside several factors. Growth,
Profitability, Margin Expansion, Cash Flow Quality, Estimate Revision, and Market
Leadership each expose their own peer-relative subfactor. A later research packet
can display that vector without creating an opaque peer score.

**No separate `operating_leverage_v1`.** `operating_margin_yoy_change` is already
the direct lower-level evidence and is substantially related to operating-income
growth relative to revenue growth when bases are positive. Counting both heavily
would double count nearly the same economics. V1 keeps `margin_expansion_v1`
honest. A Phase 2.8 operating-leverage detector may require positive Growth,
positive Margin Expansion, and inspect
`operating_income_yoy_growth > revenue_yoy_growth` as an explicit rule.

**Growth level and acceleration remain separate.** A company can grow quickly but
decelerate, or grow modestly while accelerating. Merging them would hide that
distinction precisely where early-inflection research needs it.

**Profitability level and margin change remain separate.** High stable margins and
low but rapidly improving margins are different evidence states.

**Risk is not inverted into alpha.** `market_fragility_v1` uses higher values for
more risk. It is displayed beside positive-evidence factors and is never silently
subtracted from them in Phase 2.7.

### Subfactor hierarchy

Subfactors are named calculation groups inside one factor definition, not extra
top-level factor IDs and not independently ranked securities. Each group retains
a fixed share of the final factor so missing optional components cannot let one
economic channel silently take over another.

```text
growth_v1
  absolute_growth (70%)
  peer_growth (30%)

market_leadership_v1
  trend (35%)
  relative_strength (40%)
  momentum (25%)

industry_strength_v1
  industry_fundamentals (1/3)
  industry_estimates (1/3)
  industry_market (1/3)
```

The complete hierarchy and weights are frozen in section E.

## E. Exact component mapping

### Source-version contract

| Source family | Frozen registry / metric version | Calculation/build identity required by the manifest |
| --- | --- | --- |
| Fundamentals | `ResearchFeature.version = v1` | Exact `fundamental_v1` build selected under the Phase 2.6 quarterly policy |
| Market | `market_v1` | Exact `market_v1` snapshot/session build |
| Estimates | `estimates_v1` | Exact build-qualified `estimates_v1:<provider_profile_id>:<dataset_id>` identity |
| Peer relative | `peer_v1` | Exact build-qualified peer calculation using `classification_sub_industry_v1:1` and `peer_stats_v1` |
| Industry snapshot | `industry_v1` | Exact `IndustrySnapshotMetric` and its group/statistic snapshots at T |

No table below defines a new source feature. Every source key is present in the
frozen registries. `weight` is the component's expected weight in the whole factor;
weights need not be decimal fractions because the algorithm divides exact Decimal
weight units. `R` means a required anchor. `O` means optional subject to the
coverage rules in section G. Every listed subfactor is required.

Normalization codes refer to section F: `PW(a,n,b)` is a bounded three-knot
piecewise-linear transform, `PCTL` is `2p-1`, and `IDENTITY` accepts an already
bounded `[-1,+1]` input.

### `growth_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_growth` (70) | `revenue_yoy_growth` | R | 50 | higher = stronger | `PW(-0.20, 0, 0.40)` |
| `absolute_growth` (70) | `operating_income_yoy_growth` | O | 10 | higher = stronger | `PW(-0.30, 0, 0.50)` |
| `absolute_growth` (70) | `eps_yoy_growth` | O | 10 | higher = stronger | `PW(-0.30, 0, 0.50)` |
| `peer_growth` (30) | `peer_revenue_yoy_growth_percentile` | R | 30 | higher raw percentile = stronger relative growth | `PCTL` |

Revenue deliberately carries 80% of full-coverage evidence when its absolute and
relative observations are combined. Revenue is generally less exposed than EPS
to tax, capital-structure, and share-count effects. EPS remains useful confirmation
when valid but is not required. Missing EPS therefore does not become zero and
does not automatically suppress an otherwise well-covered Growth factor.

`revenue_qoq_growth` is excluded because seasonality varies materially by business
model. `revenue_growth_acceleration` belongs to the next factor. FCF growth belongs
to Cash Flow Quality. Negative/zero/sign-change base behavior remains whatever the
frozen source feature reports; the factor never repairs it.

### `growth_acceleration_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_acceleration` (70) | `revenue_growth_acceleration` | R | 50 | higher = stronger acceleration | `PW(-0.20, 0, 0.20)` |
| `absolute_acceleration` (70) | `eps_growth_acceleration` | O | 20 | higher = stronger acceleration | `PW(-0.30, 0, 0.30)` |
| `peer_acceleration` (30) | `peer_revenue_growth_acceleration_percentile` | R | 30 | higher percentile = stronger relative acceleration | `PCTL` |

The factor is the change in growth, not growth itself. There is no frozen
operating-income acceleration feature, so none is invented. Revenue acceleration
is the anchor; EPS acceleration is optional because EPS bases and sign changes
often make it unavailable. A high Growth score and low Acceleration score remain
visible as two different facts.

### `profitability_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_profitability` (70) | `operating_margin` | R | 45 | higher = stronger | `PW(-0.10, 0, 0.30)` |
| `absolute_profitability` (70) | `gross_margin` | O | 25 | higher = stronger | `PW(0.10, 0.35, 0.70)` |
| `peer_profitability` (30) | `peer_operating_margin_percentile` | R | 18 | higher percentile = stronger relative profitability | `PCTL` |
| `peer_profitability` (30) | `peer_gross_margin_percentile` | O | 12 | higher percentile = stronger relative profitability | `PCTL` |

Operating margin dominates because it incorporates the operating cost structure
and has an economically meaningful zero boundary. Gross margin remains useful but
is more business-model-sensitive. The 30% peer channel partly normalizes that
difference without allowing a high rank in a weak or structurally low-margin
industry to erase weak absolute economics. Negative operating-margin companies
receive negative absolute evidence rather than being removed, provided the frozen
feature itself is valid. FCF margin and conversion are reserved for Cash Flow
Quality to avoid overlap.

### `margin_expansion_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_margin_change` (70) | `operating_margin_yoy_change` | R | 45 | higher = stronger expansion | `PW(-0.10, 0, 0.10)` |
| `absolute_margin_change` (70) | `gross_margin_yoy_change` | O | 25 | higher = stronger expansion | `PW(-0.08, 0, 0.08)` |
| `peer_margin_change` (30) | `peer_operating_margin_yoy_change_percentile` | R | 30 | higher percentile = stronger relative expansion | `PCTL` |

This factor measures changes, not margin levels. It may be positive during a
revenue contraction if costs fall faster; that is real margin-improvement evidence,
not proof of a healthy growth-led operating-leverage setup. Growth remains a
separate factor, and the later detector must state any positive-revenue condition.

### `cash_flow_quality_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_cash_confirmation` (70) | `free_cash_flow_margin` | R | 25 | higher = stronger cash generation | `PW(-0.10, 0, 0.25)` |
| `absolute_cash_confirmation` (70) | `free_cash_flow_conversion` | R | 20 | higher = stronger conversion within bounded domain | `PW(0, 0.80, 1.20)` |
| `absolute_cash_confirmation` (70) | `op_cash_flow_yoy_growth` | O | 15 | higher = stronger cash growth | `PW(-0.30, 0, 0.50)` |
| `absolute_cash_confirmation` (70) | `free_cash_flow_yoy_growth` | O | 10 | higher = stronger FCF growth | `PW(-0.30, 0, 0.50)` |
| `peer_cash_generation` (30) | `peer_free_cash_flow_margin_percentile` | R | 30 | higher percentile = stronger relative cash margin | `PCTL` |

The exact frozen OCF ID is `op_cash_flow_yoy_growth`. `free_cash_flow` itself is
not scored because an unscaled currency amount is not comparable across firms.
FCF margin, conversion, and FCF growth all retain the frozen rule that missing
CapEx means missing FCF; Phase 2.7 never substitutes zero CapEx. Because conversion
can be economically awkward near or below zero earnings, it is bounded, remains
fully visible, and is never interpreted alone. The factor is unavailable if its
required FCF margin or conversion evidence is unavailable or invalid.

### `estimate_revision_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `absolute_revisions` (70) | `eps_adjusted_diluted_consensus_change_scaled_30d_fq1` | R | 20 | higher = more positive | `PW(-0.20, 0, 0.20)` |
| `absolute_revisions` (70) | `eps_adjusted_diluted_revision_breadth_30d_fq1` | O | 10 | higher = broader positive activity | `IDENTITY` |
| `absolute_revisions` (70) | `eps_adjusted_diluted_consensus_change_scaled_30d_fy1` | O | 5 | higher = more positive | `PW(-0.20, 0, 0.20)` |
| `absolute_revisions` (70) | `revenue_reported_consensus_change_pct_30d_fq1` | R | 20 | higher = more positive | `PW(-0.10, 0, 0.10)` |
| `absolute_revisions` (70) | `revenue_reported_revision_breadth_30d_fq1` | O | 10 | higher = broader positive activity | `IDENTITY` |
| `absolute_revisions` (70) | `revenue_reported_consensus_change_pct_30d_fy1` | O | 5 | higher = more positive | `PW(-0.10, 0, 0.10)` |
| `peer_revisions` (30) | `peer_eps_adjusted_diluted_consensus_change_scaled_30d_fq1_percentile` | R | 12 | higher percentile = more positive relative revision | `PCTL` |
| `peer_revisions` (30) | `peer_eps_adjusted_diluted_revision_breadth_30d_fq1_percentile` | O | 6 | higher percentile = broader relative revision | `PCTL` |
| `peer_revisions` (30) | `peer_revenue_reported_consensus_change_pct_30d_fq1_percentile` | R | 12 | higher percentile = more positive relative revision | `PCTL` |

V1 uses one 30-calendar-day horizon to avoid counting the correlated 7/30/90-day
versions as independent evidence. FQ1 carries most weight because it is nearer;
FY1 adds a smaller medium-horizon check. EPS and revenue each have a required FQ1
magnitude anchor. Magnitude and breadth are complementary; magnitude can move
through composition or correction while breadth records documented contributor
direction. Revision counts are persisted diagnostics for activity/coverage but
are not scored: a high count can be many downgrades and is not inherently positive.
No raw EPS absolute-change feature is mixed across currencies or EPS levels.

### `market_leadership_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `trend` (35) | `price_vs_sma200` | R | 15 | higher = stronger trend position | `PW(-0.30, 0, 0.30)` |
| `trend` (35) | `sma_slope_50` | O | 10 | higher = stronger trend slope | `PW(-0.10, 0, 0.10)` |
| `trend` (35) | `dist_52w_high` | O | 10 | higher/closer to zero = stronger | `PW(-0.40, -0.10, 0)` |
| `relative_strength` (40) | `rel_ret_63_mkt` | R | 15 | higher = stronger market-relative return | `PW(-0.20, 0, 0.20)` |
| `relative_strength` (40) | `rel_ret_63_sec` | O | 10 | higher = stronger sector-relative return | `PW(-0.20, 0, 0.20)` |
| `relative_strength` (40) | `peer_return_63d_percentile` | R | 15 | higher percentile = stronger formal-peer return | `PCTL` |
| `momentum` (25) | `return_126d` | R | 15 | higher = stronger medium-horizon momentum | `PW(-0.30, 0, 0.40)` |
| `momentum` (25) | `mom_accel_63` | O | 10 | higher = improving momentum | `PW(-0.20, 0, 0.20)` |

The factor intentionally does not average all 22 market features.
`rel_ret_63_ind` is excluded because the formal Phase 2.6 peer-return percentile
already supplies the industry-comparison channel and the legacy basket uses a
different methodology. `return_63d` is already embedded in both relative-return
features and the peer percentile. `return_252d` adds a slower, highly correlated
horizon. `ret_abs_*` and `dollar_vol_21` are non-directional. `realized_vol_63`,
`max_dd_252`, and `vol_regime` belong to Market Fragility.

`abnormal_vol_1` and `vol_trend_21` remain non-scoring diagnostics. Volume alone
cannot distinguish accumulation from distribution, index rebalancing, or event
noise. A later signal may require positive price evidence and volume confirmation
as an explicit conjunction.

### `industry_strength_v1`

This factor is calculated once for the exact historical sub-industry node and
snapshot, not fabricated separately for every security. A later company research
packet references its node's factor value.

| Subfactor | Source Phase 2.6 industry metric key | Role | Weight units | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `industry_fundamentals` (4/12) | `median_revenue_yoy_growth` | O | 1 | higher = stronger | `PW(-0.20, 0, 0.40)` |
| `industry_fundamentals` (4/12) | `median_revenue_growth_acceleration` | R | 1 | higher = stronger | `PW(-0.20, 0, 0.20)` |
| `industry_fundamentals` (4/12) | `median_operating_margin_yoy_change` | O | 1 | higher = stronger | `PW(-0.10, 0, 0.10)` |
| `industry_fundamentals` (4/12) | `breadth_positive_revenue_acceleration` | O | 1 | higher participation = stronger | `PCTL` |
| `industry_estimates` (4/12) | `median_eps_revision_scaled_30d_fq1` | R | 2 | higher = stronger | `PW(-0.20, 0, 0.20)` |
| `industry_estimates` (4/12) | `breadth_positive_eps_revision_30d_fq1` | R | 2 | higher participation = stronger | `PCTL` |
| `industry_market` (4/12) | `median_return_63d` | R | 2 | higher = stronger | `PW(-0.20, 0, 0.20)` |
| `industry_market` (4/12) | `breadth_above_sma200` | R | 2 | higher participation = stronger | `PCTL` |

The three domains receive equal top-level weight even though they contain different
numbers of metrics. This prevents the four fundamental rows from mechanically
outvoting estimates and market participation. Median and breadth are both retained:
magnitude without participation can be narrow, while breadth without magnitude
can be economically small. All metrics must share the same exact node, taxonomy
release, group snapshot, T, and build.

### `market_fragility_v1`

| Subfactor | Source feature ID | Role | Weight | Direction | Normalization |
| --- | --- | ---: | ---: | --- | --- |
| `observed_market_risk` (100) | `realized_vol_63` | R | 40 | higher = more risk | `PW(0.10, 0.30, 0.70)` |
| `observed_market_risk` (100) | `max_dd_252` | O | 30 | more negative = more risk | `NEGATE_THEN_PW(0.10, 0.30, 0.60)` |
| `observed_market_risk` (100) | `vol_regime` | O | 30 | higher = more risk | `PW(0.75, 1.00, 1.75)` |

This is deliberately named **market** fragility. It does not claim to measure
leverage, liquidity, customer concentration, geopolitics, or business durability,
because no qualified v1 structured inputs cover those subjects. `vol_trend_21` is
not risk by itself. Higher factor values mean more fragility; no desirability
inversion occurs.

## F. Normalization and factor combination

### Canonical component range

Every scoring component is normalized to `[-1,+1]`:

```text
-1 = strong evidence at or beyond the definition's lower anchor
 0 = the definition's fixed economic neutral point
+1 = strong evidence at or beyond the definition's upper anchor
```

For `POSITIVE_EVIDENCE` factors, higher means stronger evidence for the named
economic family. For `RISK_HIGHER_WORSE`, higher means more risk. The common range
does not imply common predictive power and does not authorize adding unrelated
factors into a stock score.

Missing is never represented by zero. Missing and invalid components have null
raw/normalized values and explicit status/reason rows.

### Fixed piecewise transform

For ordered knots `a < n < b`, where `n` is economically neutral:

```text
PW(x; a,n,b) = -1                                  when x <= a
                (x-n)/(n-a)                        when a < x < n
                0                                  when x = n
                (x-n)/(b-n)                        when n < x < b
                +1                                 when x >= b
```

All arithmetic uses the repository's 38-digit `ROUND_HALF_EVEN` Decimal policy.
Binary floats, NaN, infinity, data-dependent winsorization, and ordinary z-scores
are prohibited. Saturation at declared anchors is part of the versioned transform,
not an empirical tail edit.

For an ascending Phase 2.6 percentile `p`:

```text
PCTL(p) = 2p - 1, provided 0 <= p <= 1
```

Thus the peer median is neutral, the bottom of the peer distribution is negative,
and the top is positive. This mapping does not reinterpret the upstream percentile
or change its tie policy.

Estimate revision breadth is already in `[-1,+1]` and uses `IDENTITY` after a hard
domain check. Industry positive breadth begins in `[0,1]` and uses `PCTL`, so 50%
positive participation is neutral. For `max_dd_252`, the transform first computes
non-negative drawdown severity `-x`; a positive raw maximum drawdown is outside the
frozen semantic domain and is invalid rather than silently repaired.

### Why not z-scores or contemporaneous universe percentiles?

- Growth, revisions, margins, and volume-like variables are skewed, heavy-tailed,
  and can contain base effects. A mean/standard-deviation transform is fragile.
- A universe transform would make the same economic observation change merely
  because another security entered the cross-section and would introduce a new
  Phase 2.8-style ranking dependency.
- Fixed knots are auditable, work on one requested security, and give `0` an
  economic meaning.
- Peer percentiles remain valuable where business-model context matters, but they
  are an explicit 30% channel rather than a substitute for absolute strength.

The knots are neutral v1 priors, not discovered alpha. Future empirical evidence
may motivate a challenger normalization version, never an in-place edit.

### Raw plus peer-relative architecture

For company fundamentals and estimates, v1 generally assigns 70 expected weight
units to absolute evidence and 30 to formal peer-relative evidence. This answers
both questions:

```text
Is the economic value strong on an absolute basis?
Is it strong relative to genuinely comparable companies?
```

A company growing 2% at the 90th peer percentile retains weak/near-neutral absolute
growth and positive relative growth; the relative rank cannot turn 2% into 40%.
Conversely, a company growing 40% in an even faster group retains strong absolute
growth but may have weak relative evidence. Named subfactor contributions show the
difference.

The 70/30 split is an economic prior: absolute economics dominate while peer
context is material. It was not tuned on NVDA, CRDO, or any known winner. Market
Leadership instead uses trend/relative/momentum domains, and Industry Strength
uses equal economic-family domains.

### Chosen combination method

V1 uses a **hierarchical weighted arithmetic mean**. It is preferred over the
alternatives because it preserves sign, exposes additive contributions, supports
fixed channel weights, and is easy to hand-calculate.

For subfactor `g`, let `W_g` be expected group weight, `I_g` the included component
weight, `w_i` a component's definition weight, and `z_i` its normalized value:

```text
group_coverage_g = I_g / W_g
subfactor_value_g = sum(w_i * z_i for included i in g) / I_g
factor_value = sum((W_g / W_total) * subfactor_value_g for required g)
```

All required subfactors must be valid, so their fixed top-level weights always sum
to one. Missing optional components are reweighted **only inside their own semantic
group** after coverage gates pass. Missing gross margin therefore cannot cause the
peer or trend channel to take over the factor.

For each included component, persist:

```text
effective_weight_i = (W_g / W_total) * (w_i / I_g)
contribution_i = effective_weight_i * z_i
factor_value = sum(contribution_i)
```

This also guarantees a valid factor remains in `[-1,+1]`.

Weighted median was rejected because a component can switch the output abruptly
and additive attribution is weak. Geometric combination cannot naturally preserve
negative and neutral evidence. Rule-based conjunctions are valuable, but they are
signals rather than factor aggregation. A flat average was rejected because
families with more correlated inputs would receive accidental extra weight.

### Component direction metadata

Every definition component stores one of:

```text
HIGHER_IS_STRONGER_EVIDENCE
LOWER_IS_STRONGER_EVIDENCE
HIGHER_IS_MORE_RISK
LOWER_IS_MORE_RISK
DESCRIPTIVE_NON_DIRECTIONAL
```

Only the first four may have non-zero v1 factor weight. A
`DESCRIPTIVE_NON_DIRECTIONAL` feature such as absolute return magnitude, dollar
volume, or raw volume trend may be shown as context but cannot acquire desirability
through an undocumented sign choice. Factor definitions also store one of:

```text
POSITIVE_EVIDENCE
RISK_HIGHER_WORSE
DESCRIPTIVE
```

`DESCRIPTIVE` is reserved for future families; no v1 top-level factor uses it.

## G. Missing inputs, coverage, statuses, and reasons

### Status decision

Dedicated factor status remains exactly:

```text
VALID
MISSING
INVALID
```

Do not add `PARTIAL` as a fourth status. Partial coverage is orthogonal to whether
the result passed its semantic contract. Persist:

```text
coverage_state = FULL | REDUCED
component_count_coverage
weight_coverage
each subfactor's coverage
all excluded component decisions
```

A `VALID/REDUCED` factor is inspectable and has passed every required anchor,
subfactor, and total-coverage gate. A missing input is never converted to neutral.

### Exact coverage policy: `factor_coverage_v1`

For every factor:

1. Each required component marked `R` in section E must resolve to an admitted,
   exact-build, PIT-valid numeric source artifact.
2. Every named subfactor is required and must have weight coverage **at least
   `0.60`**. Exact `0.60` passes.
3. Total factor weight coverage must be **at least `0.70`**. Exact `0.70` passes.
4. Optional missing, stale, inadmissible, or upstream-invalid components remain
   relationally present as excluded decisions and contribute no weight.
5. Required upstream `INVALID` input makes the factor `INVALID`. A required
   `MISSING`, stale, unavailable, wrong-build, or unadmitted input makes it
   `MISSING` with the most specific reason.
6. An optional upstream `INVALID` value is excluded and reduces coverage. A row
   claiming `VALID` but containing a non-finite or out-of-domain value is an
   integrity failure and makes the factor `INVALID`, regardless of optionality.
7. `ESTIMATED` source status is not admitted in v1. It is excluded as
   `COMPONENT_STATUS_NOT_ADMITTED`; no factor silently treats it as valid.

Required anchors plus fixed subfactor weights prevent opportunistic factors from
changing into whichever metric happened to be available. Examples:

- Growth remains valid without EPS only when exact revenue and peer-revenue
  anchors pass; its missing EPS row and 90% total coverage remain visible.
- Profitability cannot become “gross margin only”; operating margin and its peer
  comparison are required, and total coverage still must reach 70%.
- Estimate Revision requires both EPS and revenue FQ1 magnitude plus their peer
  magnitude anchors. It cannot turn into an EPS-only factor.
- Market Leadership needs valid trend, relative-strength, and momentum groups.
  At least 70% total coverage is required even when each group individually passes.
- Market Fragility requires realized volatility plus at least one of drawdown or
  volatility regime; `40/100` alone is insufficient and `70/100` exactly passes.

### Fundamental period coherence

Within a factor snapshot, direct fundamental components must match the fiscal
identity and `period_end` of the factor's required fundamental anchor. A peer
percentile's authoritative target `FeatureValue` must match the corresponding
direct source period. V1 never combines Q2 revenue with Q1 EPS merely because each
is the latest row for its own feature. A mismatch is excluded as
`COMPONENT_PERIOD_MISMATCH`; a required mismatch makes the factor missing.

Estimate and market coherence is equally strict:

- Estimate features require exact T, the pinned provider/dataset, one FQ1 target
  resolved at T across EPS and revenue, and the corresponding one FY1 target.
- Market features require exact T/session and the same pinned `market_v1` build.
- Peer features require exact T and their authoritative result/group/build.
- All Industry Strength inputs require the same exact industry group snapshot,
  taxonomy release, node, T, and build.

No family carries an earlier snapshot forward. A newest missing, invalid, stale,
or incompatible artifact never permits fallback to an older favorable artifact.

### Stable reason codes and precedence

Component decisions may use:

```text
COMPONENT_MISSING
COMPONENT_STALE
COMPONENT_STATUS_NOT_ADMITTED
COMPONENT_PERIOD_MISMATCH
COMPONENT_TIMESTAMP_MISMATCH
BUILD_MISMATCH
QUALITY_NOT_ADMITTED
INVALID_COMPONENT
NONFINITE_COMPONENT
NORMALIZATION_UNDEFINED
```

Factor-level reasons are:

```text
NO_COMPONENTS
COMPONENT_MISSING
COMPONENT_STALE
COMPONENT_STATUS_NOT_ADMITTED
COMPONENT_PERIOD_MISMATCH
COMPONENT_TIMESTAMP_MISMATCH
BUILD_MISMATCH
QUALITY_NOT_ADMITTED
INVALID_COMPONENT
NONFINITE_COMPONENT
NORMALIZATION_UNDEFINED
INSUFFICIENT_SUBFACTOR_COVERAGE
INSUFFICIENT_COMPONENT_COVERAGE
```

Deterministic precedence is:

1. an invalid required component or source-integrity failure -> `INVALID` with
   the applicable invalid reason;
2. zero included components -> `MISSING/NO_COMPONENTS`;
3. a required component exclusion -> `MISSING` with its most specific reason in
   the order timestamp/period, build, stale, status, quality, then generic missing;
4. a subfactor below 60% -> `MISSING/INSUFFICIENT_SUBFACTOR_COVERAGE`;
5. total coverage below 70% -> `MISSING/INSUFFICIENT_COMPONENT_COVERAGE`;
6. otherwise -> `VALID`, with `FULL` at 100% and `REDUCED` below 100%.

An invalid manifest, unknown requested factor, unpinned definition, or internally
inconsistent definition is a request-level error. The requested batch rolls back;
it is not laundered into an ordinary missing factor row.

## H. Quality propagation

Quality ordering is reused exactly:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

For a factor value:

```text
factor quality = weakest(
    factor-specific replay admission,
    every included source artifact's admitted quality,
    every required peer/industry group and statistic admission used by it
)
```

The result can never upgrade an input. The component row stores its resolved
quality and complete admission. A source `FeatureValue` that lacks a trustworthy
embedded quality field must be re-admitted against its exact manifest provider,
coverage, row provenance, and family-specific policy; the runner must not assume
quality from the feature name. Failure is `QUALITY_NOT_ADMITTED`.

Excluded optional components affect coverage but do not lower the numeric result's
quality because they did not enter it. Their attempted admissions and reasons are
still persisted. A required excluded component prevents a valid result.

Admission is **factor-specific**, not manifest-wide contamination. A synthetic
estimate provider in a broad research manifest lowers `estimate_revision_v1` when
included, but it does not lower `market_fragility_v1`, which uses no estimate
evidence. Conversely, a Growth factor that includes a synthetic peer percentile is
`DEVELOPMENT`, even if its direct SEC inputs are stronger. One family never
contaminates an unrelated factor merely because both builds were available to the
same run.

Definition lifecycle (`CANDIDATE` versus later production selection) is separate
from data quality. A candidate can use `PIT_QUALIFIED` data and remain an unvalidated
candidate; a promoted definition can still yield `DEVELOPMENT` on synthetic data.

## I. Versioning, build identity, time stability, and PIT

### Definition identity

Every immutable definition stores:

```text
factor_id
factor_version
public registry key (for example growth_v1)
component mapping and source-version expectations
subfactor hierarchy
normalization_version
weight_version
coverage_policy_version
formula_version
orientation
canonical definition payload and SHA-256 hash
```

V1 common versions are:

```text
normalization_version = factor_fixed_piecewise_v1
coverage_policy_version = factor_coverage_v1
formula_version = hierarchical_weighted_mean_v1
runner_version = factor_runner_v1
```

Each family also has its own explicit weight version, such as
`growth_weights_v1`. Changing a component, source-version expectation, knot,
direction, weight, required/optional flag, subfactor boundary, coverage threshold,
missing policy, formula, or orientation requires a new factor version. Historical
meaning is never rewritten. A wording-only correction still appends registry
metadata or documentation; immutable published definition content is not updated
in place.

### Build identity

Reuse `ResearchBuildManifest` as the canonical content-addressed build identity.
Do not concatenate provider IDs, dataset IDs, and hashes into an opaque
`calculation_version` string. The manifest must pin:

- research timestamp, git commit, schema revision, dependency versions, and raw
  inventory hash;
- exact fundamental, market, estimate, classification, eligibility, identity,
  peer, and industry dataset/build IDs actually needed by the requested factors;
- provider and qualification versions and full required lookback coverage;
- source feature/metric registry and calculation versions;
- exact factor definition hashes, normalization/weight/coverage/formula versions,
  and `factor_runner_v1`;
- calendar/session, fiscal compatibility, peer policy, and source-selection
  configuration.

Phase 2.7 should persist the canonical manifest once in a shared
`research_build_manifests` content-addressed table keyed by `build_id`, then have
factor values reference it. This is a persistence adapter for the existing
`ResearchBuildManifest`, not a competing manifest format. Phase 2.6's existing
payload-in-provenance rows remain frozen and valid.

Only datasets transitively required by a factor enter its quality calculation,
but all source IDs used by the result must belong to the one pinned manifest.

### Research snapshot semantics

`research_timestamp` is the time the question is asked. `calculated_at` is the
later execution clock. A factor is an immutable as-of snapshot, never a permanent
“latest factor.” New filings, estimate events, prices, peer classifications,
lookback ageing, or taxonomy changes produce another snapshot identity.

At T:

- every input must satisfy its frozen `available_at <= T` rule;
- fundamental resolution uses the exact build, newest measured eligible period,
  no fallback, the Phase 2.6 135-day inclusive freshness rule, and the factor's
  period-coherence rule;
- estimates, market, peer, and industry inputs require exact T snapshots;
- future recalculations or rows in another build cannot enter the factor;
- the factor runner does not call an unpinned generic “latest” helper.

An exact rerun of the same definition, subject, T, and build must reuse the same
factor, subfactor, and component IDs, values, fingerprints, and creation times.
Changed content at that identity raises
`PUBLISHED_FACTOR_CONFLICT_USE_NEW_BUILD_OR_VERSION`.

### Candidate and production definitions

Definitions and calculated values are immutable. All v1 definitions initially have
candidate research status. Research runs name the candidate definition explicitly.
A later append-only `factor_promotions` record may select a definition as champion
from an effective timestamp, with approver, validation artifact, decision, and
rollback reference. Promotion does not alter or clone historical values. A
challenger version can run beside the champion; rejection and rollback are new
records, not changes to old definitions or factor snapshots.

No Phase 2.7 implementation may call a candidate “validated,” “calibrated,” or
“production alpha” merely because its deterministic tests pass.

## J. Storage model

### Decision: dedicated factor persistence

Use dedicated factor tables. Do **not** register factors as `ResearchFeature` rows
or persist them as ordinary `FeatureValue` rows.

| Consideration | Reuse `ResearchFeature` / `FeatureValue` | Dedicated factor model |
| --- | --- | --- |
| Semantic clarity | Conflates an atomic measured feature with a composite evidence family | Preserves the frozen feature/factor distinction |
| Versioning | `research_features.feature_key` is globally unique and is awkward for concurrent v1/v2 definitions | Natural `(factor_id, factor_version)` identity and coexistence |
| Definition metadata | No first-class subfactors, transforms, weights, orientation, or coverage contract | Explicit immutable definition and component rows |
| Status | Shared enum includes `ESTIMATED`; no factor coverage state | Exact factor status plus orthogonal coverage state |
| Quality | No dedicated quality/admission columns | Weakest-input quality is first-class |
| Provenance | One JSON field; no component FKs | Exact relational source links and contributions |
| Industry inputs | Industry metrics deliberately are not `FeatureValue` rows | Can link `IndustrySnapshotMetric` directly |
| Query/backtest use | Requires JSON parsing and semantic filtering | Indexed factor/version/T/status/subject queries |
| Migration complexity | Smaller initially but pushes complexity into opaque payloads | More tables, but constraints encode the actual domain |

The extra schema is justified by auditability and long-lived historical semantics.
There should be one authoritative factor store, not a second generic
`FeatureValue` projection that can disagree with it.

### Proposed tables

#### `research_build_manifests`

```text
build_id                 CHAR(64) PRIMARY KEY
canonical_payload        TEXT NOT NULL
created_at               TIMESTAMPTZ NOT NULL
```

The payload must round-trip through `ResearchBuildManifest.create` and hash back
to `build_id`. Rows are content-addressed and immutable.

#### `research_factors`

```text
id                       BIGINT PRIMARY KEY
factor_id                TEXT NOT NULL
factor_version           INTEGER NOT NULL CHECK > 0
registry_key             TEXT NOT NULL
name                     TEXT NOT NULL
description              TEXT NOT NULL
subject_scope            SECURITY | INDUSTRY_NODE
orientation              POSITIVE_EVIDENCE | RISK_HIGHER_WORSE | DESCRIPTIVE
normalization_version    TEXT NOT NULL
weight_version           TEXT NOT NULL
coverage_policy_version  TEXT NOT NULL
formula_version          TEXT NOT NULL
definition_payload       JSONB NOT NULL
definition_hash          CHAR(64) NOT NULL
created_at               TIMESTAMPTZ NOT NULL
UNIQUE (factor_id, factor_version)
UNIQUE (registry_key)
UNIQUE (definition_hash)
```

The canonical payload includes the exact ordered subfactors and components. A
PostgreSQL immutability guard rejects `UPDATE` and `DELETE`.

#### `research_factor_components`

```text
id                       BIGINT PRIMARY KEY
research_factor_id       BIGINT NOT NULL REFERENCES research_factors
component_key            TEXT NOT NULL
ordinal                  INTEGER NOT NULL
subfactor_key            TEXT NOT NULL
subfactor_weight_units   NUMERIC NOT NULL CHECK > 0
source_kind              FEATURE_VALUE | INDUSTRY_SNAPSHOT_METRIC
source_key               TEXT NOT NULL
source_registry_version  TEXT NOT NULL
expected_units           TEXT NOT NULL
expected_frequency       TEXT NOT NULL
required                 BOOLEAN NOT NULL
component_weight_units   NUMERIC NOT NULL CHECK > 0
direction                TEXT NOT NULL
normalization_method     TEXT NOT NULL
normalization_parameters JSONB NOT NULL
definition_fingerprint   CHAR(64) NOT NULL
UNIQUE (research_factor_id, component_key)
UNIQUE (research_factor_id, ordinal)
```

Database/service validation requires all rows for a definition to reproduce its
canonical payload and hash. Definitions are registered idempotently and conflicts
fail rather than being “repaired.”

#### `factor_promotions` (promotion hook; populated only after later validation)

```text
id                       BIGINT PRIMARY KEY
research_factor_id       BIGINT NOT NULL REFERENCES research_factors
decision                 PROMOTE | REJECT | RETIRE | ROLLBACK
effective_at             TIMESTAMPTZ NOT NULL
validation_artifact_id   TEXT NOT NULL
approved_by              TEXT NOT NULL
reason                   TEXT NOT NULL
supersedes_promotion_id  BIGINT NULL REFERENCES factor_promotions
created_at               TIMESTAMPTZ NOT NULL
UNIQUE (research_factor_id, decision, effective_at)
```

This append-only table selects a definition prospectively; it never changes the
definition or historical values. Phase 2.7 may create the hook but must not insert
a `PROMOTE` decision without the later champion/challenger validation gate.

#### `factor_values`

```text
id                       BIGINT PRIMARY KEY
research_factor_id       BIGINT NOT NULL REFERENCES research_factors
subject_scope            SECURITY | INDUSTRY_NODE
security_id              INTEGER NULL REFERENCES securities
industry_node_id         BIGINT NULL REFERENCES industry_nodes
peer_group_snapshot_id   BIGINT NULL REFERENCES peer_group_snapshots
research_timestamp       TIMESTAMPTZ NOT NULL
effective_date           DATE NOT NULL
build_id                 CHAR(64) NOT NULL REFERENCES research_build_manifests
snapshot_origin          LIVE | HISTORICAL_REPLAY | EDUCATIONAL
status                   VALID | MISSING | INVALID
reason                   TEXT NULL
numeric_value            NUMERIC NULL
expected_weight          NUMERIC NOT NULL
included_weight          NUMERIC NOT NULL
weight_coverage          NUMERIC NOT NULL CHECK BETWEEN 0 AND 1
component_count          INTEGER NOT NULL
included_component_count INTEGER NOT NULL
coverage_state           FULL | REDUCED | NOT_APPLICABLE
data_quality_level       TEXT NOT NULL
admission                JSONB NOT NULL
provenance               JSONB NOT NULL
fingerprint              CHAR(64) NOT NULL
calculated_at            TIMESTAMPTZ NOT NULL
created_at               TIMESTAMPTZ NOT NULL
```

Exactly one subject key is non-null and must match the definition's subject scope.
For `INDUSTRY_NODE`, `peer_group_snapshot_id` is also required and must identify
that exact node/build/T. Partial unique indexes enforce one immutable value for:

```text
(research_factor_id, security_id, research_timestamp, build_id)
(research_factor_id, industry_node_id, peer_group_snapshot_id,
 research_timestamp, build_id)
```

A `VALID` row has a finite non-null value and null reason. `MISSING`/`INVALID`
have a null value and non-null reason. Published rows reject update/delete.

#### `factor_subfactor_values`

```text
id                       BIGINT PRIMARY KEY
factor_value_id          BIGINT NOT NULL REFERENCES factor_values
subfactor_key            TEXT NOT NULL
ordinal                  INTEGER NOT NULL
status                   VALID | MISSING | INVALID
reason                   TEXT NULL
fixed_factor_weight      NUMERIC NOT NULL
expected_component_weight NUMERIC NOT NULL
included_component_weight NUMERIC NOT NULL
coverage                 NUMERIC NOT NULL CHECK BETWEEN 0 AND 1
numeric_value            NUMERIC NULL
contribution             NUMERIC NULL
data_quality_level       TEXT NULL
fingerprint              CHAR(64) NOT NULL
UNIQUE (factor_value_id, subfactor_key)
```

This table makes the absolute/relative or trend/relative/momentum decomposition a
queryable result rather than duplicated JSON.

#### `factor_value_components`

```text
id                       BIGINT PRIMARY KEY
factor_value_id          BIGINT NOT NULL REFERENCES factor_values
definition_component_id  BIGINT NOT NULL REFERENCES research_factor_components
feature_value_id         INTEGER NULL REFERENCES feature_values
industry_snapshot_metric_id BIGINT NULL REFERENCES industry_snapshot_metrics
peer_relative_result_id  BIGINT NULL REFERENCES peer_relative_results
included                 BOOLEAN NOT NULL
exclusion_reason         TEXT NULL
source_status            TEXT NOT NULL
source_reason            TEXT NULL
source_period_end        DATE NULL
source_available_at      TIMESTAMPTZ NULL
source_calculation_version TEXT NOT NULL
raw_value                NUMERIC NULL
raw_units                TEXT NULL
normalized_value         NUMERIC NULL
definition_weight        NUMERIC NOT NULL
effective_weight         NUMERIC NULL
contribution             NUMERIC NULL
data_quality_level       TEXT NULL
provider_admission       JSONB NULL
selection_provenance     JSONB NOT NULL
fingerprint              CHAR(64) NOT NULL
UNIQUE (factor_value_id, definition_component_id)
```

Exactly one of `feature_value_id` and `industry_snapshot_metric_id` is present for
an included v1 component. A `peer_v1` FeatureValue additionally requires its
authoritative `peer_relative_result_id`; database/service checks bind the projected
value to that result. Included rows require raw value, normalized value, effective
weight, contribution, quality, and null exclusion reason. Excluded rows require a
reason and null normalized/contribution fields. The table stores the exact source
artifact even when the source is missing/invalid, when such an artifact exists.

No v1 scoring component depends on multiple source artifacts, which keeps this
lineage one-to-one. A future genuinely multi-input component must add a normalized
`factor_component_inputs` junction rather than hiding an ID list in JSON.

### Publication behavior

- Register and validate definitions before calculation; extra or conflicting v1
  rows fail atomically.
- Resolve all inputs and compute all requested values before publication.
- Publish factor value, subfactors, and every expected component in one savepoint/
  caller transaction.
- Exact concurrent reruns converge on the same identity and compare all content.
- Changed content conflicts and requires a new build or factor version.
- Persist `MISSING` and `INVALID` results as research evidence; do not publish only
  winners.
- PostgreSQL guards reject mutation of definitions and published results.

## K. Provenance contract

Starting from one factor value, an auditor must reconstruct:

```text
subject security/company or exact historical industry node
research timestamp T, effective date D, snapshot origin, and execution time
factor_id, factor_version, definition hash, and lifecycle selection
normalization, weight, coverage, formula, and runner versions
canonical ResearchBuildManifest payload and build_id
all source dataset/build/provider qualifications and exact coverage
every expected component and its included/excluded decision
exact FeatureValue or IndustrySnapshotMetric IDs
authoritative PeerRelativeResult IDs for peer projections
raw values, units, statuses, reasons, period/target identity, and available_at
normalized values, definition/effective weights, and contributions
missing components and subfactor/total coverage
component, subfactor, and final quality/admission
final status, reason, value, fingerprint, and immutable creation identity
```

Compact JSON may repeat the canonical formula, manifest hash, and admission for
convenient export. It may not replace the relational source links. The factor
fingerprint covers the ordered definition, subject, T/D, build, all expected
component decisions, source IDs, raw and normalized values, weights,
contributions, coverage, quality, status/reason, and final value.

Example audit question:

> Why was `growth_v1` equal to `0.94` at T?

The answer is not “the model said so.” It is the exact two subfactor values, three
included component rows, one excluded EPS row, fixed definition, source IDs,
quality, and arithmetic shown in section L.

## L. Worked semiconductor example

Consider fictional **Asteria Semiconductor** at an aware research timestamp T.
This is an arithmetic illustration, not a historical case or an optimization
example. Ratios use decimal form (`0.40` means 40%).

Selected raw evidence:

| Evidence | Raw value | Source status / quality |
| --- | ---: | --- |
| `revenue_yoy_growth` | `0.40` | VALID / `PIT_QUALIFIED` |
| `operating_income_yoy_growth` | `0.55` | VALID / `PIT_QUALIFIED` |
| `eps_yoy_growth` | missing | MISSING |
| `peer_revenue_yoy_growth_percentile` | `0.90` | VALID / `DEVELOPMENT` synthetic peer evidence |
| `revenue_growth_acceleration` | `0.12` | VALID / `PIT_QUALIFIED` |
| `peer_revenue_growth_acceleration_percentile` | `0.80` | VALID / `DEVELOPMENT` |
| `gross_margin` | `0.65` | VALID / `PIT_QUALIFIED` |
| `operating_margin` | `0.28` | VALID / `PIT_QUALIFIED` |
| `gross_margin_yoy_change` | `0.03` | VALID / `PIT_QUALIFIED` |
| `operating_margin_yoy_change` | `0.04` | VALID / `PIT_QUALIFIED` |
| `eps_adjusted_diluted_consensus_change_scaled_30d_fq1` | `0.12` | VALID / `DEVELOPMENT` synthetic estimates |
| `eps_adjusted_diluted_revision_breadth_30d_fq1` | `0.75` | VALID / `DEVELOPMENT` |
| `rel_ret_63_mkt` | `0.18` | VALID / `DEVELOPMENT` market source |
| `price_vs_sma200` | `0.15` | VALID / `DEVELOPMENT` |
| `sma_slope_50` | `0.06` | VALID / `DEVELOPMENT` |
| `peer_return_63d_percentile` | `0.85` | VALID / `DEVELOPMENT` |

### Detailed `growth_v1` calculation

Normalize the absolute components:

```text
revenue growth:
  PW(0.40; -0.20, 0, 0.40) = +1.00

operating-income growth:
  PW(0.55; -0.30, 0, 0.50) = +1.00  (saturated at upper anchor)

EPS growth:
  MISSING -> null, never 0
```

The absolute group expected weight is 70; included weight is `50 + 10 = 60`:

```text
absolute group coverage = 60 / 70 = 0.857142857...
absolute group value = (50*1.00 + 10*1.00) / 60 = 1.00
```

Normalize the peer component:

```text
PCTL(0.90) = 2*0.90 - 1 = 0.80
peer group coverage = 30 / 30 = 1.00
peer group value = 0.80
```

Both required anchors are present, both groups exceed 60%, and total weight
coverage is:

```text
(60 + 30) / 100 = 0.90
```

The factor is therefore `VALID/REDUCED`:

```text
growth_v1 = 0.70*1.00 + 0.30*0.80 = 0.94
```

Component attribution after within-group reweighting is:

| Component | Effective weight | Normalized value | Contribution |
| --- | ---: | ---: | ---: |
| Revenue growth | `0.70*(50/60) = 0.583333...` | `1.00` | `0.583333...` |
| Operating-income growth | `0.70*(10/60) = 0.116666...` | `1.00` | `0.116666...` |
| EPS growth | null | null | null |
| Peer revenue-growth percentile | `0.30` | `0.80` | `0.24` |
| **Total** | **`1.00`** |  | **`0.94`** |

Quality is the weakest included evidence. Although the direct fundamental rows are
`PIT_QUALIFIED`, the included synthetic peer component is `DEVELOPMENT`, so the
factor quality is **`DEVELOPMENT`**. Missing EPS does not lower quality because it
did not enter the value; it lowers coverage and remains an excluded component row.

This `0.94` is strong Growth evidence under `growth_v1`. It is not a 94%
probability and says nothing by itself about valuation, fragility, or whether the
stock should be bought.

### Separate acceleration meaning

```text
PW(0.12; -0.20, 0, 0.20) = 0.60
PCTL(0.80) = 0.60
```

With EPS acceleration missing, the absolute group has `50/70 = 0.714285...`
coverage, total coverage is 80%, and both groups pass:

```text
growth_acceleration_v1 = 0.70*0.60 + 0.30*0.60 = 0.60
```

Asteria therefore has very strong current growth (`0.94`) but only moderately
positive acceleration (`0.60`). Keeping the factors separate preserves that
economically useful distinction.

### Selected component intuition in other families

```text
gross margin:
  PW(0.65; 0.10, 0.35, 0.70)
  = (0.65-0.35)/(0.70-0.35) = 0.857142857...

operating margin:
  PW(0.28; -0.10, 0, 0.30) = 0.933333333...

gross-margin change:
  PW(0.03; -0.08, 0, 0.08) = 0.375

operating-margin change:
  PW(0.04; -0.10, 0, 0.10) = 0.40

scaled EPS revision:
  PW(0.12; -0.20, 0, 0.20) = 0.60

EPS breadth:
  IDENTITY(0.75) = 0.75

market-relative return:
  PW(0.18; -0.20, 0, 0.20) = 0.90

price versus SMA200:
  PW(0.15; -0.30, 0, 0.30) = 0.50

SMA50 slope:
  PW(0.06; -0.10, 0, 0.10) = 0.60

peer return percentile:
  PCTL(0.85) = 0.70
```

Those are inspectable contributions, not an instruction to average the families
into one stock score. Missing required revenue-revision, peer-revision, trend, or
momentum inputs would keep the corresponding top-level factor missing even though
the displayed individual components are positive.

## M. Future test matrix

### Registry and formula contract

- Exact nine public IDs, logical IDs/versions, subject scopes, orientations, and
  candidate status; reject an extra v1 factor.
- Exact ordered component IDs, actual frozen source keys, source versions,
  subfactor assignments, required flags, weights, directions, transforms, and
  version strings.
- Definition registration is idempotent; tampered metadata, duplicate ordinals,
  unknown source keys, non-positive weights, or a changed hash fail atomically.
- Every transform hand-calculates at lower knot, neutral knot, upper knot,
  between knots, and beyond knots under exact Decimal arithmetic.
- `PCTL(0)= -1`, `PCTL(0.5)=0`, `PCTL(1)=1`; breadth endpoints and exact zero;
  reject values outside declared domains.
- `max_dd_252` direction is reversed exactly once; Market Fragility remains
  higher-is-worse and is never inverted into positive evidence.
- Weighted subfactor and component contributions sum exactly to the factor value
  under the declared Decimal policy and stay in `[-1,+1]`.

### Missingness and coverage

- All components present -> `VALID/FULL` with expected values and quality.
- Required component missing -> `MISSING` with exact reason and null value.
- Optional component missing above threshold -> `VALID/REDUCED`, no zero
  substitution, correct within-group reweighting.
- Exact 60% subfactor coverage passes; one exact Decimal unit below fails with
  `INSUFFICIENT_SUBFACTOR_COVERAGE`.
- Exact 70% total coverage passes; one unit below fails with
  `INSUFFICIENT_COMPONENT_COVERAGE`.
- A subfactor passes but total factor coverage fails, and vice versa.
- No usable components -> `MISSING/NO_COMPONENTS` with all decisions retained.
- Required stale input -> `MISSING/COMPONENT_STALE`; optional stale input reduces
  coverage only.
- Required upstream invalid -> `INVALID/INVALID_COMPONENT`; optional upstream
  invalid may be excluded when all gates still pass.
- A source marked VALID with NaN/infinity or impossible domain -> factor
  `INVALID/NONFINITE_COMPONENT` or `NORMALIZATION_UNDEFINED`.
- Valid zero and valid negative values remain numeric evidence and are not confused
  with missing.
- `ESTIMATED` source status is excluded; required and optional cases behave as
  specified.
- Missing CapEx propagates through FCF inputs and prevents a valid Cash Flow
  Quality factor; no fabricated FCF appears.
- Missing EPS can still produce valid Growth/Acceleration only at the specified
  anchor and coverage boundaries.

### PIT, build, and source isolation

- A component available one microsecond after T is excluded; exact T is eligible
  where its frozen family permits it.
- Wrong calculation version, dataset, manifest, provider, units, frequency,
  accounting basis, horizon, currency/share basis, peer policy, or taxonomy release
  cannot enter a value.
- Only another-build rows available -> `BUILD_MISMATCH`; no generic latest fallback.
- Fundamental components must share exact fiscal identity/period; Q1/Q2 mixing is
  excluded and tested.
- Estimate EPS/revenue FQ1 target coherence and FY1 target coherence are enforced
  through provenance.
- Market/estimate/peer inputs require exact T; earlier snapshots are not carried.
- Industry inputs from two groups, nodes, releases, timestamps, or builds cannot
  be combined.
- A later filing, correction, price, estimate event, peer reclassification, or
  recalculated feature cannot change an already published factor snapshot.
- Synthetic estimate input forces only affected included factors to `DEVELOPMENT`;
  an unrelated qualified factor is not contaminated.
- `QUALITY_NOT_ADMITTED` is fail-closed and includes the exact attempted admission.

### Family-specific economics

- Growth: revenue dominates; missing optional EPS; negative/zero/extreme growth;
  low absolute/high peer and high absolute/low peer cases.
- Acceleration: high level/decelerating and low level/accelerating cases remain
  distinct; no invented operating-income acceleration.
- Profitability: negative operating margin, high gross/low operating margin, and
  low absolute/high peer cases.
- Margin Expansion: contraction with margin improvement remains positive margin
  evidence but is not labeled a growth-led operating-leverage signal.
- Cash Flow Quality: negative/zero/extreme FCF margin/conversion, volatile cash
  growth, and missing conversion semantics.
- Estimate Revision: magnitude versus breadth disagreement, correction-driven
  consensus movement, FQ1 versus FY1 disagreement, no-directional-revision breadth
  missing, and counts never entering the score.
- Market Leadership: positive trend/weak relative strength, strong relative/weak
  absolute trend, momentum deceleration, and excluded directionless volume.
- Industry Strength: equal family weights despite unequal metric counts, median/
  breadth disagreement, and one shared node-scoped result for multiple companies.
- Market Fragility: low/high volatility, shallow/deep drawdown, rising/falling
  regime, and no automatic buy/sell inversion.

### Persistence and operations

- Complete relational walk from FactorValue through subfactors/components to every
  exact FeatureValue, PeerRelativeResult, IndustrySnapshotMetric, group, manifest,
  provider admission, and raw evidence chain.
- Included/excluded row constraints and exactly-one-source FK constraints.
- Exact rerun preserves every ID, value, reason, source link, fingerprint, and
  creation timestamp.
- Changed content at the same identity fails; a new build or factor version
  publishes separately without changing old rows.
- Candidate and production-selection histories remain isolated; a promotion does
  not rewrite values.
- Injected failure rolls back value, subfactors, and components together.
- Concurrent writers converge on one complete immutable publication; no reader
  sees partial components.
- PostgreSQL rejects update/delete of definitions, manifests, values, subfactors,
  and components.
- Fresh migration, downgrade/upgrade boundaries, metadata parity, and populated
  migration preflights pass when implementation begins.
- A 10,000-security requested snapshot uses bounded family-wise queries and bulk
  publication; assert query count does not grow per security and inspect plans/
  memory before the final audit.

No test may claim investment effectiveness. Formula regression, provider
qualification, historical predictive validation, and production readiness remain
separate gates.

## N. Scale, batching, caching, and materialization

The expected order of magnitude is 10,000 securities times nine factor families
at selected research timestamps. PostgreSQL remains appropriate. No distributed
compute, streaming platform, vector database, or daily Cartesian materialization
is justified.

Required batch API shape:

```text
calculate_factors(
    subjects,
    research_timestamp,
    research_build_manifest,
    factor_registry_keys,
) -> immutable factor values
```

Implementation should:

1. validate the manifest and requested definition set once;
2. resolve registry IDs once;
3. group source retrieval by family and fetch all requested securities in bounded
   set-based queries;
4. use window-ranked/bulk selection for exact fundamental periods and exact-T
   estimate/market/peer rows;
5. fetch peer results and industry metrics by group/build in bulk and reuse one
   industry calculation across all securities in a node;
6. normalize and combine with Decimal arithmetic without database round trips per
   security/component;
7. bulk-insert values, subfactors, and components in safe batches; and
8. resolve publication conflicts by immutable identity comparison.

The immutable identity itself is the cache key. On-demand calculation persists the
snapshot; exact later requests reuse it after content verification. Do not maintain
one mutable “latest” cache. Do not generate daily full-universe factor rows unless
a future backtest/run manifest explicitly requests those dates. If measured history
later justifies partitioning or incremental materialization, add it under a new
operational design without changing factor meaning.

Node-scoped Industry Strength avoids thousands of duplicate identical rows.
Company research snapshots reference the exact node factor. At full coverage, a
requested 10,000-security date produces at most roughly 80,000 security-scoped v1
factor rows plus the much smaller number of node-scoped industry rows, not a
security-by-industry-metric Cartesian product.

## O. Backtest compatibility and future diagnostics

Phase 2.7 makes later Phase 2.9 questions possible without performing them now:

```text
Does high growth_v1 precede delivered growth or future excess returns?
Does high estimate_revision_v1 precede further revisions or outperformance?
Does high market_leadership_v1 precede excess returns?
Does high market_fragility_v1 precede drawdown or adverse path outcomes?
```

Each historical experiment pins the factor definition, build, coverage/status,
eligible universe, prediction time, execution policy, label version, and outcome
build. Missing factor rows and exclusions stay in the cohort record. Outcome data
remain in the separate ledger and can never enter the factor manifest.

Phase 2.7 does not compute factor percentiles or ranks. A later cross-sectional
artifact must identify its PIT universe, tie rule, denominator, factor version,
and build. It must not overwrite the underlying factor value.

Future redundancy diagnostics, calculated outside the factor runner, should include:

- Pearson correlation of bounded factor values on paired valid observations;
- Spearman rank correlation at the same T and eligible cohort;
- overlap of later top/bottom quantiles and signal triggers;
- missingness and coverage co-occurrence;
- correlations by sector, size, time, provider build, and predeclared regime; and
- out-of-sample family ablations against future objective-specific labels.

Growth versus Acceleration and Profitability versus Margin Expansion are expected
to correlate sometimes; they remain separate because level and change are distinct.
Diagnostics determine whether a later composite should use both. Phase 2.7 does
not tune weights after inspecting those outcomes.

Historical experiments preserve candidate definitions, all trials, failures,
false positives, delisted names, and unseen chronological holdouts. NVDA, CRDO,
Helios, or another famous/fixture winner may illustrate mechanics but cannot select
formulas or weights. Promotion follows the existing champion/challenger contract.

## P. Future extensions

### Valuation

Defer a Valuation factor. The repository has no qualified PIT market-cap/share-
count/enterprise-value bridge, debt/cash alignment, forward valuation engine, or
scenario/intrinsic-value implementation. A shallow proxy would create false
precision. A later valuation workstream should define DCF/scenarios, FCF yield,
EV/Revenue, EV/EBITDA, economically valid P/E/PEG relationships, historical bands,
peer/growth adjustment, and margin of safety before registering a factor version.

### Business quality

Do not create a Business Quality factor from an LLM opinion. The documented
BusinessQualityResearch dossier, moat hypotheses, management execution, capital
allocation, customer/supplier concentration, competitive evidence, and ROIC
definitions remain future work. If qualified, they should first exist as cited,
versioned, scoped assertions with counter-evidence and outcomes. Any later numeric
factor needs its own reviewed rubric and validation.

### News, events, and geopolitics

Do not create news sentiment, catalyst, policy, or geopolitical factors. The event
layer is not production-qualified, and sentiment direction is not an economic fact.
Future event evidence must preserve occurrence/publication/availability, source
versions, company roles, mechanism, lifecycle, uncertainty, and neutral political
context before any factor proposal.

### Alternative data and professional research

Future qualified observations may cover web activity, jobs, supply chains, product
usage, options, short interest, ownership, or licensed professional research. A
proxy is not revenue and an analyst report is not automatically a consensus event.
Rights, PIT history, identity, coverage, common-origin dependence, and source-
specific validation are prerequisites.

### ML, calibration, and ranking

No ML is needed for v1. Later candidates may fit transforms or weights only on
chronologically prior, mature outcomes and must beat the simple frozen baseline
under the same cohorts. Calibrated probabilities name one outcome/horizon and are
stored separately. Universe ranking and detector combination remain inspectable
and versioned; no hidden model may replace the factor lineage.

The storage design permits future source kinds and factor versions, but v1 code
must reject unregistered components rather than treating an extension point as
permission to ingest them.

## Q. Open decisions

There are **no blocking Phase 2.7 architecture decisions left open**. The v1
registry, transforms, weights, coverage, quality, build, storage, provenance, and
phase boundary are exact enough to implement.

The following are deliberately deferred gates, not ambiguities in this design:

- whether empirical evidence eventually promotes any candidate or favors a v2
  transform/weight set;
- which external providers eventually qualify above `DEVELOPMENT`;
- the exact Phase 2.9 replay schedule and outcome cohorts;
- any future valuation, qualitative, event, alternative-data, ML, signal, or rank
  definitions.

Implementation checkpoint sequencing and migration revision names are engineering
planning details to define at Phase 2.7 implementation start; they cannot change
the semantic contract above.

## R. Recommendation

**SAFE TO IMPLEMENT PHASE 2.7**

Implementation must remain inside this factor boundary, preserve every frozen
upstream formula, begin all nine definitions as candidates, retain explicit
missingness and quality, and stop before Phase 2.8.
