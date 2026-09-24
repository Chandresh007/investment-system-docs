# Phase 2.7 — Factor Implementation Contract

Status: **ARCHITECTURE APPROVED; IMPLEMENTATION SPEC FROZEN**.

This document is the concise, authoritative implementation contract for Phase
2.7. It preserves the decisions in
[`phase-2.7-factor-architecture-review.md`](phase-2.7-factor-architecture-review.md).
If a detail is not repeated here, that architecture review remains authoritative.
Foundation and Phases 2.2, 2.4, 2.5, and 2.6 remain frozen. Phase 2.7 consumes
their immutable artifacts and does not recompute or reinterpret their formulas.

All source keys below were checked against the implemented registries at the
approved starting checkpoint `f51f1741488a74ce68db7ce3336382b4df03eed8`.

## 1. Scope and phase boundary

Phase 2.7 v1 registers and calculates exactly these nine candidate factors:

| Registry key | Logical ID / version | Subject | Orientation |
| --- | --- | --- | --- |
| `growth_v1` | `growth` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `growth_acceleration_v1` | `growth_acceleration` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `profitability_v1` | `profitability` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `margin_expansion_v1` | `margin_expansion` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `cash_flow_quality_v1` | `cash_flow_quality` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `estimate_revision_v1` | `estimate_revision` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `market_leadership_v1` | `market_leadership` / `1` | `SECURITY` | `POSITIVE_EVIDENCE` |
| `industry_strength_v1` | `industry_strength` / `1` | `INDUSTRY_NODE` | `POSITIVE_EVIDENCE` |
| `market_fragility_v1` | `market_fragility` / `1` | `SECURITY` | `RISK_HIGHER_WORSE` |

Every v1 definition starts in lifecycle state `CANDIDATE`. Deterministic tests do
not make a candidate validated, calibrated, or production alpha.

Phase 2.7 does not add `peer_leadership_v1`, `operating_leverage_v1`,
`valuation_v1`, `business_quality_v1`, `news_sentiment_v1`, an omnibus score,
buy/sell recommendation, rank, signal, expected return, probability, portfolio,
or position size. Phase 2.8 does not begin in this work.

The semantic boundary is:

```text
raw facts
  -> frozen deterministic features and industry metrics
  -> Phase 2.7 factor evidence summaries
  -> future Phase 2.8 signals / detectors
  -> later calibration, ranking, and research synthesis
```

A factor is a deterministic, interpretable, versioned, PIT-safe, quality-aware
aggregation of related evidence. It is not a probability, signal, rank, thesis,
recommendation, or opaque learned score.

## 2. Canonical value range and orientation

Every valid v1 factor value is a finite Decimal in `[-1,+1]`.

```text
-1 = strong evidence at the definition's negative/lower anchor
 0 = the definition's fixed economic neutral point
+1 = strong evidence at the definition's positive/upper anchor
```

For `POSITIVE_EVIDENCE`, a larger value means stronger evidence for the named
economic family. For `market_fragility_v1`, orientation is
`RISK_HIGHER_WORSE`: **+1 means more observed market fragility**. It is never
silently inverted into desirability. The common range does not imply equal
predictive power and does not authorize combining factors into a stock score.

Missing is never zero. A missing or invalid component has null raw and normalized
values plus an explicit exclusion decision.

## 3. Frozen source and build contract

| Source family | Registry/version | Exact build identity required |
| --- | --- | --- |
| Fundamentals | `ResearchFeature.version = v1` | Manifest-pinned `fundamental_v1` build selected under the frozen 135-day quarterly policy |
| Market | `market_v1` | Exact manifest-pinned `market_v1` snapshot/session build |
| Estimates | `estimates_v1` | Exact `estimates_v1:<provider_profile_id>:<dataset_id>` identity pinned by the manifest |
| Peer relative | `peer_v1` | Exact manifest/build-qualified peer result under `classification_sub_industry_v1:1` and `peer_stats_v1` |
| Industry snapshot | `industry_v1` | Exact `IndustrySnapshotMetric`, source statistic/group snapshot, node, taxonomy release, T, and build |

Direct peer inputs use the immutable projected `peer_v1` `FeatureValue` and must
also link its authoritative `PeerRelativeResult`. Industry inputs link the
authoritative `IndustrySnapshotMetric` directly. No source formula is duplicated
inside the factor implementation.

One hash-verified `ResearchBuildManifest` pins every source identity actually used
by a factor request and all factor definition/version identities. The canonical
payload is persisted once in `research_build_manifests`, keyed by its SHA-256
`build_id`. A wall clock, mutable “latest” selector, or opaque concatenated version
string is not semantic build identity.

The manifest must pin, as applicable:

- research timestamp, Git commit, schema revision, dependency versions, and raw
  inventory hash;
- exact fundamental, market, estimate, classification, eligibility, identity,
  peer, and industry dataset/build IDs;
- provider/qualification versions and required lookback coverage;
- source registry/calculation versions and compatibility policies;
- exact factor definition hashes and normalization, weight, coverage, formula,
  and runner versions; and
- fiscal, calendar/session, peer, taxonomy, and source-selection configuration.

## 4. Exact v1 factor definitions

`R` is a required anchor. `O` is optional subject to the coverage rules. Every
listed subfactor is required. Component weights are expected weight units in the
whole factor. Subfactor labels show their fixed factor weight.

Normalization codes are defined in section 5. Directions are persisted as exact
metadata values:

```text
HIGHER_IS_STRONGER_EVIDENCE
LOWER_IS_STRONGER_EVIDENCE
HIGHER_IS_MORE_RISK
LOWER_IS_MORE_RISK
DESCRIPTIVE_NON_DIRECTIONAL
```

Only the first four may carry non-zero v1 weight.

### 4.1 `growth_v1`

Weight version: `growth_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_growth` (70) | `revenue_yoy_growth` | R | 50 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.40)` |
| `absolute_growth` (70) | `operating_income_yoy_growth` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.50)` |
| `absolute_growth` (70) | `eps_yoy_growth` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.50)` |
| `peer_growth` (30) | `peer_revenue_yoy_growth_percentile` | R | 30 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

Revenue is the anchor and dominant evidence. `revenue_qoq_growth`, acceleration,
and FCF growth do not enter this factor. Frozen source handling of sign changes
and invalid bases is preserved.

### 4.2 `growth_acceleration_v1`

Weight version: `growth_acceleration_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_acceleration` (70) | `revenue_growth_acceleration` | R | 50 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `absolute_acceleration` (70) | `eps_growth_acceleration` | O | 20 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.30)` |
| `peer_acceleration` (30) | `peer_revenue_growth_acceleration_percentile` | R | 30 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

This measures the change in growth, never the growth level. No operating-income
acceleration feature is invented.

### 4.3 `profitability_v1`

Weight version: `profitability_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_profitability` (70) | `operating_margin` | R | 45 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.30)` |
| `absolute_profitability` (70) | `gross_margin` | O | 25 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(0.10,0.35,0.70)` |
| `peer_profitability` (30) | `peer_operating_margin_percentile` | R | 18 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `peer_profitability` (30) | `peer_gross_margin_percentile` | O | 12 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

FCF margin and conversion remain exclusively in Cash Flow Quality. Profitability
does not make a business-quality judgment.

### 4.4 `margin_expansion_v1`

Weight version: `margin_expansion_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_margin_change` (70) | `operating_margin_yoy_change` | R | 45 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.10)` |
| `absolute_margin_change` (70) | `gross_margin_yoy_change` | O | 25 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.08,0,0.08)` |
| `peer_margin_change` (30) | `peer_operating_margin_yoy_change_percentile` | R | 30 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

This measures margin change, not margin level or a growth-led operating-leverage
signal.

### 4.5 `cash_flow_quality_v1`

Weight version: `cash_flow_quality_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_cash_confirmation` (70) | `free_cash_flow_margin` | R | 25 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.25)` |
| `absolute_cash_confirmation` (70) | `free_cash_flow_conversion` | R | 20 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(0,0.80,1.20)` |
| `absolute_cash_confirmation` (70) | `op_cash_flow_yoy_growth` | O | 15 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.50)` |
| `absolute_cash_confirmation` (70) | `free_cash_flow_yoy_growth` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.50)` |
| `peer_cash_generation` (30) | `peer_free_cash_flow_margin_percentile` | R | 30 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

The exact source ID is `op_cash_flow_yoy_growth`. `free_cash_flow` currency amount
is not scored. Missing CapEx remains missing upstream FCF evidence and can never be
replaced with zero.

### 4.6 `estimate_revision_v1`

Weight version: `estimate_revision_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `absolute_revisions` (70) | `eps_adjusted_diluted_consensus_change_scaled_30d_fq1` | R | 20 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `absolute_revisions` (70) | `eps_adjusted_diluted_revision_breadth_30d_fq1` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `IDENTITY` |
| `absolute_revisions` (70) | `eps_adjusted_diluted_consensus_change_scaled_30d_fy1` | O | 5 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `absolute_revisions` (70) | `revenue_reported_consensus_change_pct_30d_fq1` | R | 20 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.10)` |
| `absolute_revisions` (70) | `revenue_reported_revision_breadth_30d_fq1` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `IDENTITY` |
| `absolute_revisions` (70) | `revenue_reported_consensus_change_pct_30d_fy1` | O | 5 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.10)` |
| `peer_revisions` (30) | `peer_eps_adjusted_diluted_consensus_change_scaled_30d_fq1_percentile` | R | 12 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `peer_revisions` (30) | `peer_eps_adjusted_diluted_revision_breadth_30d_fq1_percentile` | O | 6 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `peer_revisions` (30) | `peer_revenue_reported_consensus_change_pct_30d_fq1_percentile` | R | 12 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

Only the frozen 30-calendar-day horizon is scored. Revision counts are diagnostics,
not directional evidence, and do not enter the factor. Phase 2.7 never reconstructs
estimate events or consensus.

### 4.7 `market_leadership_v1`

Weight version: `market_leadership_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `trend` (35) | `price_vs_sma200` | R | 15 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.30)` |
| `trend` (35) | `sma_slope_50` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.10)` |
| `trend` (35) | `dist_52w_high` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.40,-0.10,0)` |
| `relative_strength` (40) | `rel_ret_63_mkt` | R | 15 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `relative_strength` (40) | `rel_ret_63_sec` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `relative_strength` (40) | `peer_return_63d_percentile` | R | 15 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `momentum` (25) | `return_126d` | R | 15 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.30,0,0.40)` |
| `momentum` (25) | `mom_accel_63` | O | 10 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |

Phase 2.7 does not recalculate returns, moving averages, or trend. `rel_ret_63_ind`,
directionless volume measures, absolute-return magnitudes, and risk features are
not scored here.

### 4.8 `industry_strength_v1`

Weight version: `industry_strength_weights_v1`. Each domain has fixed factor
weight `1/3` and expected component weight `4`; the overall expected weight is 12.

| Subfactor | `industry_v1` metric key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `industry_fundamentals` (4/12) | `median_revenue_yoy_growth` | O | 1 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.40)` |
| `industry_fundamentals` (4/12) | `median_revenue_growth_acceleration` | R | 1 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `industry_fundamentals` (4/12) | `median_operating_margin_yoy_change` | O | 1 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.10,0,0.10)` |
| `industry_fundamentals` (4/12) | `breadth_positive_revenue_acceleration` | O | 1 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `industry_estimates` (4/12) | `median_eps_revision_scaled_30d_fq1` | R | 2 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `industry_estimates` (4/12) | `breadth_positive_eps_revision_30d_fq1` | R | 2 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |
| `industry_market` (4/12) | `median_return_63d` | R | 2 | `HIGHER_IS_STRONGER_EVIDENCE` | `PW(-0.20,0,0.20)` |
| `industry_market` (4/12) | `breadth_above_sma200` | R | 2 | `HIGHER_IS_STRONGER_EVIDENCE` | `PCTL` |

This factor is calculated once for an exact historical sub-industry node and
snapshot. Company research later references that node-scoped result; Phase 2.7
does not duplicate it per security.

### 4.9 `market_fragility_v1`

Weight version: `market_fragility_weights_v1`.

| Subfactor | Source key | Role | Weight | Direction | Transform |
| --- | --- | --- | ---: | --- | --- |
| `observed_market_risk` (100) | `realized_vol_63` | R | 40 | `HIGHER_IS_MORE_RISK` | `PW(0.10,0.30,0.70)` |
| `observed_market_risk` (100) | `max_dd_252` | O | 30 | `LOWER_IS_MORE_RISK` | `NEGATE_THEN_PW(0.10,0.30,0.60)` |
| `observed_market_risk` (100) | `vol_regime` | O | 30 | `HIGHER_IS_MORE_RISK` | `PW(0.75,1.00,1.75)` |

For `max_dd_252`, first compute severity `-x`; a positive raw drawdown is outside
the frozen semantic domain and invalid. `vol_trend_21` is not scored. This factor
does not claim to measure leverage, liquidity, concentration, geopolitical risk,
or business durability.

## 5. Versioned normalization

Common versions are frozen as:

```text
normalization_version   = factor_fixed_piecewise_v1
coverage_policy_version = factor_coverage_v1
formula_version         = hierarchical_weighted_mean_v1
runner_version          = factor_runner_v1
```

All arithmetic uses the repository's 38-digit `ROUND_HALF_EVEN` Decimal policy.
Binary floats, NaN, infinity, learned thresholds, z-scores, data-dependent
winsorization, and retrospective threshold optimization are prohibited.

For exact Decimal knots `a < n < b`:

```text
PW(x; a,n,b) = -1               when x <= a
                (x-n)/(n-a)     when a < x < n
                0               when x = n
                (x-n)/(b-n)     when n < x < b
               +1               when x >= b
```

Saturation at `a` and `b` is part of the definition. It is not post-hoc clipping.

For an ascending frozen peer/industry percentile or breadth `p`:

```text
PCTL(p) = 2p - 1, provided 0 <= p <= 1
```

There is no extra clipping. Values outside `[0,1]` are invalid.

`IDENTITY(x)` accepts a finite Decimal only when `-1 <= x <= 1`. It is used for
already-bounded estimate revision breadth. `NEGATE_THEN_PW` validates the raw
drawdown domain `x <= 0`, computes `-x`, and then applies the declared `PW` knots.

## 6. Hierarchical combination and coverage

The v1 formula is a hierarchical weighted arithmetic mean. It must not be
flattened into one component average.

For subfactor `g`, let `W_g` be expected group weight, `I_g` included component
weight, `w_i` component weight, and `z_i` normalized value:

```text
group_coverage_g = I_g / W_g
subfactor_value_g = sum(w_i * z_i for included i in g) / I_g
factor_value = sum((W_g / W_total) * subfactor_value_g for required g)

effective_weight_i = (W_g / W_total) * (w_i / I_g)
contribution_i = effective_weight_i * z_i
factor_value = sum(contribution_i)
```

The only missing-component reweighting occurs **inside the same semantic
subfactor**, and only after all gates pass. Every named subfactor retains its fixed
top-level share.

Exact `factor_coverage_v1` policy:

1. Every `R` component must resolve to an admitted, exact-build, PIT-valid finite
   numeric source artifact.
2. Every subfactor must have weighted coverage at least `0.60`. Exactly `0.60`
   passes.
3. Total included component weight divided by total expected weight must be at
   least `0.70`. Exactly `0.70` passes.
4. Optional missing, stale, inadmissible, or upstream-invalid components are
   persisted as excluded decisions and contribute no value or weight.
5. Required upstream `INVALID` makes the factor `INVALID`. Required missing,
   stale, status-inadmissible, wrong-build, or quality-inadmissible evidence makes
   it `MISSING` with the most specific reason.
6. Optional upstream `INVALID` is normally excluded. A row claiming `VALID` with
   non-finite or out-of-domain content is a source-integrity failure and makes the
   factor `INVALID`, regardless of optionality.
7. `FeatureStatus.ESTIMATED` is not admitted in v1 and is excluded as
   `COMPONENT_STATUS_NOT_ADMITTED`.

Valid result coverage is `FULL` only at exact total weight coverage `1`; otherwise
it is `REDUCED`. Missing and invalid factor rows use `NOT_APPLICABLE`. Persist both
component-count coverage and exact weighted coverage, plus every subfactor's
expected/included weights and coverage.

## 7. Statuses, exclusion reasons, and coherence

Factor and subfactor status is exactly:

```text
VALID | MISSING | INVALID
```

There is no `PARTIAL` status. Reduced coverage is orthogonal to validity.

Stable component reasons are:

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

Factor-level reasons are the same where applicable, plus:

```text
NO_COMPONENTS
INSUFFICIENT_SUBFACTOR_COVERAGE
INSUFFICIENT_COMPONENT_COVERAGE
```

Deterministic precedence is:

1. invalid required component or source-integrity failure -> `INVALID` with the
   applicable invalid reason;
2. zero included components -> `MISSING/NO_COMPONENTS`;
3. required exclusion -> `MISSING`, choosing the most specific reason in the
   order timestamp/period, build, stale, status, quality, generic missing;
4. any subfactor below 60% ->
   `MISSING/INSUFFICIENT_SUBFACTOR_COVERAGE`;
5. total factor coverage below 70% ->
   `MISSING/INSUFFICIENT_COMPONENT_COVERAGE`;
6. otherwise -> `VALID/FULL` or `VALID/REDUCED`.

An invalid manifest, unknown factor key, unpinned definition, or internally
inconsistent definition is a request-level error that rolls back the requested
batch rather than publishing a missing factor.

Source coherence is mandatory:

- Direct fundamental components share the required anchor's fiscal identity and
  `period_end`. The authoritative target `FeatureValue` behind a peer percentile
  matches the corresponding direct period. Q2/Q1 mixing is excluded.
- Estimate components require exact T, pinned profile/dataset/build, one FQ1
  target across EPS and revenue, and the corresponding one FY1 target.
- Market components require exact T/session and one pinned `market_v1` build.
- Peer components require exact T and authoritative result/group/build.
- Industry components share one exact node, taxonomy release, peer group
  snapshot, research timestamp, source build, and factor build.

No family carries an earlier snapshot forward. The newest required source being
missing, invalid, stale, or incompatible never permits fallback to an older row.

## 8. Quality policy and isolation

Reuse the frozen ordering exactly:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

For each valid factor:

```text
quality = weakest(
    factor-specific replay/provider admission,
    every included source artifact admission,
    every required peer/industry group and statistic admission used
)
```

The result never upgrades an input. Each included component stores its resolved
quality and admission. A `FeatureValue` without trustworthy embedded quality must
be re-admitted against its exact manifest provider, coverage, row provenance, and
family policy. Failure is `QUALITY_NOT_ADMITTED`.

Quality isolation is per factor family. Synthetic estimates can make
`estimate_revision_v1` `DEVELOPMENT` and synthetic peer evidence can make the
specific factors using it `DEVELOPMENT`; neither automatically downgrades an
unrelated factor that did not consume those artifacts. A development market input
downgrades only factors that include that market evidence. Excluded optional
components reduce coverage but do not lower the quality of the numeric result.

Lifecycle state (`CANDIDATE`) and data quality are independent.

## 9. Definition identity and immutability

Each immutable definition stores:

```text
factor_id
factor_version
registry_key
name and description
subject_scope
orientation
ordered subfactors and components
source-version expectations
normalization_version
weight_version
coverage_policy_version
formula_version
canonical definition payload
SHA-256 definition_hash
created_at
```

Changing any component set, source expectation, required anchor, weight,
subfactor assignment/boundary, normalization method/knot, coverage threshold,
missing policy, direction, orientation, quality policy, or formula requires a new
factor version. Published definitions are never updated or deleted. Registration
is idempotent only for byte-for-byte/canonically equivalent content; a collision
fails atomically instead of repairing the row.

An append-only `factor_promotions` hook may be created for future `PROMOTE`,
`REJECT`, `RETIRE`, or `ROLLBACK` decisions. Phase 2.7 inserts no promotion and
implements no champion/challenger behavior beyond preserving version identity.

## 10. Dedicated relational storage

Factors do not reuse `ResearchFeature` or `FeatureValue` as their authoritative
store. The implementation uses repository-compatible forms of these tables:

### `research_build_manifests`

- primary key `build_id CHAR(64)`;
- canonical payload text that round-trips through `ResearchBuildManifest.create`
  and hashes back to the key; and
- immutable `created_at`.

### `research_factors`

- unique `(factor_id, factor_version)`, `registry_key`, and `definition_hash`;
- subject scope, orientation, version fields, canonical JSONB definition payload,
  hash, description, lifecycle state, and creation time; and
- PostgreSQL guards rejecting `UPDATE` and `DELETE`.

### `research_factor_components`

- one ordered row for every expected component;
- FK to the definition; unique component key and ordinal per definition;
- subfactor key and expected subfactor weight;
- source kind/key/registry version, expected units/frequency;
- required flag, component weight, direction, normalization method/parameters;
- canonical component fingerprint; and
- immutable guards and definition-payload/hash consistency validation.

V1 source kinds are `FEATURE_VALUE` and `INDUSTRY_SNAPSHOT_METRIC`. Peer inputs
are `FEATURE_VALUE` rows with an additional authoritative
`peer_relative_result_id` on the result lineage.

### `factor_values`

- FK to definition and build manifest;
- exactly one subject: `security_id` or `industry_node_id`, matching scope;
- an industry result also requires its exact `peer_group_snapshot_id`;
- T, effective date, execution clock, and `LIVE | HISTORICAL_REPLAY |
  EDUCATIONAL` snapshot origin;
- status/reason/value, expected/included weight, weight and count coverage,
  component counts, coverage state, quality, admission, compact provenance,
  fingerprint, and immutable creation time;
- partial unique identities for security and industry-node snapshots; and
- PostgreSQL checks for subject exclusivity, valid status/value combinations,
  finite bounded values, coverage ranges, and update/delete rejection.

### `factor_subfactor_values`

- one ordered row per expected subfactor and factor value;
- status/reason/value, fixed factor weight, expected/included component weight,
  coverage, contribution, quality, and fingerprint;
- uniqueness on `(factor_value_id, subfactor_key)`; and
- immutable guards and valid/null field checks.

### `factor_value_components`

- one row per expected definition component, including excluded decisions;
- FK to factor value and definition component;
- exact `feature_value_id` or `industry_snapshot_metric_id`; included rows have
  exactly one source; peer inputs also require authoritative
  `peer_relative_result_id`;
- source status/reason, period/end availability, calculation version, raw value
  and units, normalized value, definition/effective weight, contribution,
  quality/admission, selection provenance, and fingerprint;
- uniqueness on `(factor_value_id, definition_component_id)`;
- included rows require source/value/normalization/effective-weight/contribution/
  quality and a null exclusion reason;
- excluded rows require an exclusion reason and null normalized/effective-weight/
  contribution; and
- relational consistency and immutable update/delete guards.

Large JSON blobs may repeat compact formula/admission context for export, but may
not replace relational source FKs. A future truly multi-source component requires
an explicit junction table, not a hidden ID array.

## 11. PIT snapshot and publication semantics

`research_timestamp` T is when the research question is asked. `calculated_at` is
the later execution clock. At T, every input must satisfy the frozen source
family's `available_at <= T`, exact-build, freshness, status, provider, and
compatibility contract.

- Fundamentals select the exact-build newest measured eligible period, with no
  fallback and the inclusive 135-day policy.
- Estimates, market, peer, and industry sources require exact-T snapshots.
- Future corrections, recalculations, datasets, classifications, and taxonomy
  releases cannot enter an older factor snapshot.
- A published source `FeatureValue` remains immutable; factor code never updates
  or deletes it.

Resolve and calculate the entire requested batch before publication. Publish a
factor value, all expected subfactors, and all expected component decisions in one
savepoint/caller transaction. Persist valid, missing, and invalid results; do not
publish only favorable rows.

An exact rerun of definition, subject, T, build, and inputs reuses every factor,
subfactor, and component ID, fingerprint, value, reason, source link, and creation
time. Different content at that immutable identity raises:

```text
PUBLISHED_FACTOR_CONFLICT_USE_NEW_BUILD_OR_VERSION
```

A changed source build or new factor version creates a distinct artifact without
mutating prior history. Concurrent writers must converge on one complete result;
no reader may observe partial lineage.

## 12. Required provenance

Starting from one factor value, an auditor must reconstruct:

```text
subject security/company or exact historical industry node
T, effective date, snapshot origin, calculated_at, and created_at
factor logical ID/version, registry key, definition hash, and candidate state
normalization, weight, coverage, formula, and runner versions
canonical ResearchBuildManifest payload and build_id
exact source datasets/builds/providers/qualification coverage
every expected component and included/excluded decision
exact FeatureValue or IndustrySnapshotMetric IDs
authoritative PeerRelativeResult IDs for peer projections
raw values, units, source status/reason, period/target, and available_at
normalized values, definition/effective weights, and contributions
every subfactor value, coverage, quality, and contribution
missing components and total count/weight coverage
component and final provider admission/quality
final status/reason/value/fingerprint and immutable identity
```

The factor fingerprint covers the ordered definition, subject, T/effective date,
build, every expected component decision/source ID/raw and normalized value,
weights, contributions, coverage, quality, status/reason, and final value.

This lineage must answer, using arithmetic and exact source rows, questions such
as “Why was `market_leadership_v1 = 0.64` at T?”

## 13. Runner and batching contract

The explicit Phase 2.7 API is repository-compatible with:

```text
calculate_factors(
    subjects,
    research_timestamp,
    research_build_manifest,
    factor_registry_keys,
) -> immutable factor values
```

`FactorRunner` responsibilities are limited to:

1. validate the canonical manifest and exact requested definition set once;
2. resolve immutable definitions once;
3. group source retrieval by family and batch all requested subjects;
4. select exact PIT/build-qualified inputs without a generic latest fallback;
5. normalize components and calculate the fixed hierarchy using Decimal;
6. propagate component-specific coverage, quality, status, and reasons;
7. calculate each node-scoped industry factor once and reuse its reference;
8. persist factor values, subfactors, every component decision, manifest linkage,
   quality, and provenance in bounded batches; and
9. resolve exact reruns through immutable identity comparison.

The runner is isolated from the frozen fundamental, market, estimate, and peer
runners. It does not activate factor definitions in `ResearchFeature`, insert
factor outputs into `FeatureValue`, or alter any upstream runner scan.

At approximately 10,000 securities, query count must be bounded by source family,
not grow per security. Avoid unbounded loads, N+1 reads, Cartesian expansion, and
daily full-universe materialization. PostgreSQL remains the storage engine; a
distributed system is not justified for v1.

## 14. Mandatory verification contract

The implementation test matrix includes at least:

- exact nine registry IDs, logical versions, scopes, orientations, lifecycle
  state, ordered subfactors/components, weights, anchors, directions, transforms,
  source versions, and rejection of extras/tampering;
- piecewise lower/neutral/upper knots, interior values, saturation, negative and
  zero input, Decimal precision, non-finite rejection, identity domain, drawdown
  domain, and `PCTL(0)=-1`, `PCTL(0.5)=0`, `PCTL(1)=1`;
- all components present; required anchor missing; optional missing; valid zero;
  upstream invalid; status not admitted; exactly/below 60% group coverage;
  exactly/below 70% factor coverage; correct within-group reweighting; and no
  missing-to-zero conversion;
- weakest-input quality and factor-specific isolation, including synthetic
  estimates and development market/peer inputs;
- future source exclusion, wrong build/version exclusion, period/T mismatch,
  exact boundary availability, no older-source fallback, and no future mutation
  of a published factor;
- complete component/subfactor/source lineage, source `FeatureValue` immutability,
  factor-table FK integrity, valid finite bounded outputs, and database guards;
- exact rerun idempotency, changed-content conflict, new factor-version isolation,
  atomic rollback, and concurrent one-winner publication;
- all nine economic families, including the architecture review's deterministic
  semiconductor arithmetic without tuning the thresholds to the fixture;
- a production-path synthetic PostgreSQL lifecycle from frozen upstream features
  and peer/industry artifacts through factor persistence at multiple timestamps;
  and
- a representative 10,000-security run measuring SQL statements, runtime, peak
  memory, plans/index use, and PostgreSQL temporary I/O.

Fresh PostgreSQL migration verification must cover base through the new Phase 2.7
head, every new downgrade/upgrade boundary, metadata parity, populated migration
preflights where applicable, immutable triggers, and focused tests. Existing
migrations through `p26008` are never rewritten.

No test may claim investment effectiveness, empirical validation, provider
qualification, or production readiness.

## 15. Provider and release limitations

Existing external gates remain in force. Synthetic estimates, classifications,
peer inputs, or development market evidence force only the factors that consume
them to the weakest applicable quality, normally `DEVELOPMENT`. Phase 2.7 never
upgrades provider evidence or calls a deterministic candidate a production
factor.

Licensed historical classifications, qualified primary-equity/delisted coverage,
production estimate/market/fundamental providers, source rights, and strict
accounting provenance remain external dependencies. Missing provider quality is
not permission to weaken PIT, provenance, coverage, or frozen semantics.

## 16. Implementation checkpoint sequence

1. **2.7.0:** freeze this implementation contract.
2. **2.7.1:** factor definitions, manifest adapter, dedicated storage, migration,
   immutability, and relational-integrity tests.
3. **2.7.2:** versioned Decimal normalization engine and boundary tests.
4. **2.7.3:** required anchors, coverage, hierarchy, statuses, and quality
   propagation calculator.
5. **2.7.4:** Growth, Growth Acceleration, Profitability, Margin Expansion, and
   Cash Flow Quality source mappings.
6. **2.7.5:** Estimate Revision source mapping and estimate-specific quality/PIT
   tests.
7. **2.7.6:** Market Leadership, Industry Strength, and Market Fragility source
   mappings and orientation tests.
8. **2.7.7:** isolated registry-backed `FactorRunner`, batch retrieval,
   publication, and all-nine-ID tests.
9. **2.7.8:** full deterministic synthetic PostgreSQL lifecycle.
10. **2.7.9:** 10,000-security scale/performance sanity proof and structural fixes
    only where reproduced.
11. **2.7.10:** fresh internal audit, reproduced-defect fixes, README/state handoff,
    and readiness for independent Phase 2.7 review.

Every coherent checkpoint requires focused tests, the appropriate frozen-phase
regressions, a full suite, `PROJECT_STATE.md` update, commit, push, verification
that `HEAD == origin/master`, and a clean worktree. Phase 2.8 remains out of scope.
