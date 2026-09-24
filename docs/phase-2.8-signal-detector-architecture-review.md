# Phase 2.8 signal / detector architecture review

Review date: 2026-09-23

Approved starting checkpoint: `9eebee90d8a6c8eb77a0f2b75369244376ca5d88`
on `master`, equal to `origin/master` with a clean worktree before this review.
Alembic current/head is `p27001`. After the preserved local PostgreSQL container
was started, the baseline reproduced **736 passed, 8 existing warnings, zero
failures**.

This is an architecture review only. It adds no application code, model,
migration, signal row, prediction, rank, calibration, or Phase 2.9 behavior.
Foundation and Phases 2.1, 2.2, 2.4, 2.5, 2.6, and 2.7 remain frozen. In
particular, this design consumes the exact nine frozen Phase 2.7 factors and does
not change their formulas, source mappings, transforms, weights, coverage, or
quality semantics.

## A. Verdict

**CONDITIONAL PASS**

The architecture is exact enough to implement. The conditions are release and
evidence gates, not unresolved signal mechanics:

- Every v1 signal definition begins as `CANDIDATE`. Deterministic execution does
  not establish predictive value, alpha, or production fitness.
- The thresholds in this review are symmetric, simple economic priors on the
  frozen `[-1,+1]` factor scale. They were not fitted to outcomes or famous
  winners. Any later empirical change creates a new signal version.
- A signal consumes only exact, immutable Phase 2.7 `FactorValue` artifacts from
  one pinned factor build. It never recalculates a feature or factor and never
  substitutes a newer factor version.
- Missing factor evidence is never neutral. A signal result distinguishes a
  determinate non-fire from an indeterminate or invalid evaluation.
- Data quality is signal-local and cannot exceed the weakest factor in the
  deterministic decision proof. Synthetic estimate evidence remains
  `DEVELOPMENT` when it is required to establish a result.
- Opportunity, confirmation, context, warning, and contradiction signals coexist.
  No v1 rule converts their count into an overall score or silently cancels one
  with another.
- Phase 2.8 stops before cross-sectional ranking, probability calibration,
  outcome evaluation, portfolio construction, thesis generation, or AI judgment.

Subject to those conditions, section T recommends **SAFE TO IMPLEMENT PHASE
2.8**.

## B. Phase 2.8 scope

### In scope

- An immutable registry of nine focused signal definitions and two detector
  identities.
- A closed, typed rule tree over exact frozen factor values using only inclusive
  `>=`, inclusive `<=`, `ALL`, and `AT_LEAST(k)` operations.
- Boolean fired/not-fired decisions, deterministic evidence strength for fired
  results, severity, signal class, factor-family support, contradiction, warning,
  missingness, coverage, and quality.
- One security-scoped fundamental acceleration signal that does not require
  estimate or market data, plus a separately confirmed early-growth candidate.
- Node-scoped industry context calculated once and referenced by company research,
  rather than duplicated once per security.
- Immutable definitions, results, condition-level factor lineage, content
  fingerprints, exact factor-build pinning, atomic publication, and idempotent
  replay.
- Persistence of every requested result: fired, not fired, missing, and invalid.
  Publishing only positive detections is prohibited.
- A minimal deterministic signal artifact that a later research snapshot or
  prediction ledger can reference without treating the signal itself as a
  prediction.
- Bounded, set-based execution for requested universes, including explicit
  10,000-security scale verification during implementation.

### Out of scope

- Any change to a frozen feature or factor definition, value, quality rule, or
  runner.
- Direct use of raw features in v1 signal rules. Phase 2.8 begins at the frozen
  factor boundary.
- An overall stock score, buy/sell score, expected return, probability of
  success, recommendation, watchlist priority, or position size.
- Cross-sectional security ranking. Phase 2.6 peer percentiles embedded in
  factors remain source context; Phase 2.8 adds no universe-wide rank.
- A historical backtester, outcome labels, execution simulation, calibration,
  automatic threshold fitting, ML, or champion promotion.
- A prediction/outcome ledger, forward-return claim, catalyst deadline, or thesis
  adjudication. Compatibility is designed here; implementation belongs to the
  later ledger/backtest phase.
- Valuation, business quality, news, catalyst, macro, alternative-data, or AI
  signals. Qualified factors for those families do not yet exist.
- A mutable “latest signal” cache or unrequested daily full-universe
  materialization.

### Exact phase boundary

```text
frozen PIT features
  -> frozen Phase 2.7 factors
  -> Phase 2.8 deterministic signals / detectors
  -> Phase 2.9 historical replay and outcome ledger
  -> Phase 2.10 calibration, diagnostics, and ranking
  -> later research thesis and cited AI synthesis
```

## C. Definitions

| Term | Frozen Phase 2.8 meaning |
| --- | --- |
| **Feature** | One deterministic measurement under a frozen formula and source contract, such as `revenue_yoy_growth` or `return_126d`. A feature may be descriptive, positive, negative, missing, or invalid. |
| **Factor** | One versioned, interpretable evidence family that deterministically summarizes related frozen inputs, with a bounded value, orientation, coverage, quality, contributions, and lineage. A factor is not a decision. |
| **Signal** | One immutable, versioned interpretation of factor states for a named research pattern. A signal result records whether the rule fired, why, the exact supporting and contradicting evidence, coverage, quality, and, only when fired, evidence strength and severity. |
| **Detector** | A deterministic, versioned rule-set implementation that evaluates one or more signal definitions and emits a result for every requested definition and subject. A detector is code and registry metadata; it is not an additional investment conclusion. |
| **Rank** | A cross-sectional order or percentile over an explicitly pinned eligible universe at T, including denominator and tie policy. Phase 2.8 does not produce one. |
| **Prediction** | A prospectively or retrospectively originated, dated claim about a named future outcome, horizon, threshold, and adjudication policy. A signal becomes prediction evidence only when a later immutable ledger packet explicitly uses it. |
| **Thesis** | A structured, versioned explanation of a proposed mechanism, with claims, evidence, counter-evidence, catalysts, risks, horizons, and invalidation conditions. It may consume signals later but is not generated by the v1 detector. |

The words `score` and `confidence` are not signal-layer domain terms. The only
numeric signal output is `evidence_strength`, which is a deterministic rule-margin
measure defined in section G. It is not a probability, expected return, universe
rank, or recommendation.

## D. V1 signal registry

The smallest useful v1 registry contains nine definitions:

| Registry key | Display code | Scope | Class | Purpose |
| --- | --- | --- | --- | --- |
| `fundamental_acceleration_v1` | `FUNDAMENTAL_ACCELERATION` | `SECURITY` | `OPPORTUNITY` | Strong growth level and strong growth acceleration together. It requires neither estimates nor market evidence, although its frozen factors contain their existing peer-relative context. |
| `margin_inflection_v1` | `MARGIN_INFLECTION` | `SECURITY` | `OPPORTUNITY` | Strong margin-expansion evidence, kept distinct from current profitability level. |
| `estimate_confirmation_v1` | `ESTIMATE_CONFIRMATION` | `SECURITY` | `CONFIRMATION` | Strong positive same-target estimate-revision evidence. |
| `market_confirmation_v1` | `MARKET_CONFIRMATION` | `SECURITY` | `CONFIRMATION` | Strong trend, relative-strength, and momentum evidence. |
| `industry_tailwind_v1` | `INDUSTRY_TAILWIND` | `INDUSTRY_NODE` | `CONTEXT` | Strong positive node-scoped industry evidence. It is context, not company alpha. |
| `industry_headwind_v1` | `INDUSTRY_HEADWIND` | `INDUSTRY_NODE` | `CONTEXT` | Strong negative node-scoped industry evidence. It is not automatically a company sell signal. |
| `cash_flow_contradiction_v1` | `CASH_FLOW_CONTRADICTION` | `SECURITY` | `CONTRADICTION` | Strong growth accompanied by negative cash-flow-quality evidence. |
| `fragility_warning_v1` | `FRAGILITY_WARNING` | `SECURITY` | `WARNING` | Strong observed market fragility. Higher `market_fragility_v1` is worse. |
| `early_growth_candidate_v1` | `EARLY_GROWTH_CANDIDATE` | `SECURITY` | `OPPORTUNITY` | Strong growth and acceleration plus at least one independent confirmation family, with contradictions and warnings retained rather than netted away. |

All nine definitions begin in lifecycle state `CANDIDATE`.

### Deliberate omissions

- There is no `growth_signal_v1` or `growth_acceleration_signal_v1` that merely
  renames one factor. `fundamental_acceleration_v1` interprets their conjunction.
- There is no separate `cash_flow_confirmation_v1` in v1. Positive cash flow can
  confirm early growth inside the compound rule, while the more decision-relevant
  growth/cash disagreement receives its own contradiction signal.
- There is no one-for-one profitability signal. Profitability level is context in
  the early-growth rule; `margin_expansion_v1`, not `profitability_v1`, measures
  deterioration or improvement.
- There is no `GROWTH_ACCELERATION`, `ESTIMATE_REVISION_STRENGTH`, or
  `MARKET_LEADERSHIP_CONFIRMATION` alias beside the canonical IDs above. Duplicate
  names would create two historical meanings for the same rule.
- There is no omnibus “bullish,” “bearish,” or final recommendation signal.

## E. Detector registry

Two code-level detector identities are sufficient:

| Detector key | Outputs | Inputs |
| --- | --- | --- |
| `factor_pattern_detector_v1` | The first eight atomic/context signal definitions in section D | Exact Phase 2.7 factor values only |
| `early_growth_detector_v1` | `early_growth_candidate_v1` | Exact Phase 2.7 factor values and the security's exact node-scoped industry factor reference |

The common execution implementation is versioned as
`signal_detector_runner_v1`.

Definitions live in an explicit Python registry as frozen dataclasses containing
a small typed rule tree. This is declarative enough to canonicalize, hash, inspect,
and test, but it is not a general-purpose DSL: no database-authored rules, string
evaluation, arbitrary expressions, SQL fragments, or runtime-edited thresholds
are allowed. The database is an immutable mirror of the code-owned definition,
not a control plane that can change historical semantics.

V1 does **not** persist a separate `DetectorResult`. The detector is the rule-set
and execution identity; `SignalResult` is its authoritative domain output. A
second result abstraction would duplicate status, quality, lineage, and
idempotency without adding meaning. Detector ID/version and runner version are
stored in each signal definition, build, and result provenance.

The compound detector reads factor values directly rather than depending on
previous signal-result rows. Shared immutable condition constants prevent rule
drift, while the expanded canonical rule in each definition makes every signal
self-contained and independently replayable. This avoids evaluation-order and
partial-publication dependencies between atomic and compound signals.

## F. Rule definitions

### F.1 Shared factor-state vocabulary

The following buckets apply only inside Phase 2.8 definitions. They do not change
factor values:

| State | Exact interval |
| --- | --- |
| `STRONG_NEGATIVE` | `-1 <= x <= -0.50` |
| `NEGATIVE` | `-0.50 < x <= -0.10` |
| `NEUTRAL` | `-0.10 < x < +0.10` |
| `POSITIVE` | `+0.10 <= x < +0.50` |
| `STRONG_POSITIVE` | `+0.50 <= x <= +1` |

The cutoffs are symmetric around zero. `+/-0.10` creates a small neutral band;
`+/-0.50` requires evidence halfway from the factor's fixed neutral point to its
bounded extreme. These are transparent v1 priors, not empirically optimized
cutoffs. Every comparison below is inclusive at its displayed threshold and uses
the repository's exact Decimal policy.

For `market_fragility_v1`, the vocabulary describes risk evidence:
`STRONG_POSITIVE` means strongly elevated fragility, not a desirable state.

### F.2 Atomic and context rules

| Signal | Decision rule | Required / alternative | Non-gating context | Stops firing at a later snapshot when... |
| --- | --- | --- | --- | --- |
| `fundamental_acceleration_v1` | `growth_v1 >= +0.50 AND growth_acceleration_v1 >= +0.50` | Both factor conditions are required. | Separate margin, cash, industry, and fragility results remain visible. | Either required factor is below `+0.50`; missing/invalid evidence is indeterminate, not a negative value. |
| `margin_inflection_v1` | `margin_expansion_v1 >= +0.50` | Required. | `growth_v1 <= -0.10` is stored as a non-gating contradiction because margin improvement may be cost-led during contraction. | Margin Expansion falls below `+0.50`. |
| `estimate_confirmation_v1` | `estimate_revision_v1 >= +0.50` | Required. | Synthetic estimate inputs are allowed only with their existing admission and keep this result `DEVELOPMENT`. | Estimate Revision falls below `+0.50`. |
| `market_confirmation_v1` | `market_leadership_v1 >= +0.50` | Required. | `fragility_warning_v1` may fire simultaneously. | Market Leadership falls below `+0.50`. |
| `industry_tailwind_v1` | `industry_strength_v1 >= +0.50` | Required node factor. | No company desirability is inferred. | Industry Strength falls below `+0.50`. |
| `industry_headwind_v1` | `industry_strength_v1 <= -0.50` | Required node factor. | It may coexist with a company opportunity signal. | Industry Strength rises above `-0.50`. |
| `cash_flow_contradiction_v1` | `growth_v1 >= +0.50 AND cash_flow_quality_v1 <= -0.10` | Both are required. | This records disagreement; it does not erase Growth. | Growth is below `+0.50` or Cash Flow Quality is above `-0.10`. |
| `fragility_warning_v1` | `market_fragility_v1 >= +0.50` | Required. | It never subtracts from an opportunity strength. | Market Fragility falls below `+0.50`. |

“Stops firing” describes a new evaluation at a later T. A later non-fire never
mutates or invalidates the immutable earlier snapshot. Prediction/thesis
invalidation with a deadline remains a later ledger concern.

### F.3 Exact `early_growth_candidate_v1` rule

The detector separates the growth core from independent confirmation families:

```text
GROWTH_CORE = ALL(
    growth_v1 >= +0.50,
    growth_acceleration_v1 >= +0.50,
)

FUNDAMENTAL_ECONOMICS_CONFIRMATION = AT_LEAST(1,
    profitability_v1 >= +0.10,
    margin_expansion_v1 >= +0.10,
    cash_flow_quality_v1 >= +0.10,
)

INDEPENDENT_CONFIRMATION = AT_LEAST(1,
    EXPECTATIONS: estimate_revision_v1 >= +0.50,
    MARKET:       market_leadership_v1 >= +0.50,
    INDUSTRY:     industry_strength_v1 >= +0.50,
    FUNDAMENTAL_ECONOMICS: FUNDAMENTAL_ECONOMICS_CONFIRMATION,
)

EARLY_GROWTH_CANDIDATE = ALL(
    GROWTH_CORE,
    INDEPENDENT_CONFIRMATION,
)
```

`growth_v1` and `growth_acceleration_v1` are level and change, but they share
fundamental growth evidence and count together as the one required
`FUNDAMENTAL_GROWTH` family. The detector therefore requires at least two
independent families in total: `FUNDAMENTAL_GROWTH` plus one of
`EXPECTATIONS`, `MARKET`, `INDUSTRY`, or `FUNDAMENTAL_ECONOMICS`. Several
profitability/margin/cash conditions still count as one Fundamental Economics
family; correlated members cannot inflate the family count.

The exact node-scoped `industry_strength_v1` is resolved through the security's
pinned historical peer-group/node context at the same T, effective date, and
factor build. It is not copied into a fabricated security factor.

The following diagnostic predicates are evaluated and persisted but never veto
the signal:

```text
margin_expansion_v1 <= -0.10  -> MARGIN_DETERIORATION
cash_flow_quality_v1 <= -0.10 -> CASH_FLOW_WEAKNESS
profitability_v1 <= -0.50     -> WEAK_PROFITABILITY_LEVEL
industry_strength_v1 <= -0.50 -> INDUSTRY_HEADWIND
market_fragility_v1 >= +0.50  -> FRAGILITY_WARNING
```

`profitability_v1` is a level factor, so `WEAK_PROFITABILITY_LEVEL` must not be
described as profitability “collapsing.” Margin Expansion supplies the actual
change evidence.

There are no economic vetoes in v1. Strong growth can therefore coexist with
Cash Flow Contradiction, Industry Headwind, or Fragility Warning. Integrity,
missingness, build, and quality failures can make the evaluation unavailable;
that is not an economic veto.

### F.4 Estimate-absent companies

Analyst estimates are not required for either `fundamental_acceleration_v1` or
`early_growth_candidate_v1`:

- `FUNDAMENTAL_ACCELERATION` is the early, estimate/market-independent pattern.
- `EARLY_GROWTH_CANDIDATE` additionally requires one independent family, but that
  family may be Market, Industry, or Fundamental Economics rather than Estimates.
- If Estimate Revision is missing and another family passes, the candidate may
  fire with reduced evidence coverage. Estimate missingness remains explicit.
- If no family passes and one or more potentially decisive families are missing,
  the candidate is `MISSING/INSUFFICIENT_CONFIRMATION_COVERAGE`, not false and not
  neutral.

This avoids two overlapping `EARLY_GROWTH_CANDIDATE_CORE` and
`EARLY_GROWTH_CANDIDATE_CONFIRMED` labels. The fundamental signal plus separately
visible confirmation signals already express the research sequence.

## G. Evidence strength and severity

### G.1 Output contract

Every result has:

```text
evaluation_status = VALID | MISSING | INVALID
fired              = true | false | null
evidence_strength  = Decimal [0,1] | null
severity           = LOW | MEDIUM | HIGH | null
```

- `VALID` always has a Boolean `fired` value.
- Only `VALID/fired=true` has evidence strength and severity.
- `VALID/fired=false`, `MISSING`, and `INVALID` have null strength and severity.
- Zero is a legitimate just-at-threshold strength for a fired signal. Null means
  that strength is not applicable; it is not silently converted to zero.

A `[0,1]` range is chosen instead of `[-1,+1]` because direction is already
carried by the named signal, operator, and class. A larger value means the fired
rule lies farther beyond its thresholds. It never means probability or expected
return.

### G.2 Exact leaf margins

For a passed `x >= theta` condition:

```text
m_ge(x, theta) = (x - theta) / (1 - theta)
```

For a passed `x <= theta` condition:

```text
m_le(x, theta) = (theta - x) / (theta + 1)
```

The frozen factor domain and thresholds ensure denominators are positive and
each passed margin is in `[0,1]`. A condition exactly on its inclusive threshold
has margin zero; a condition at the relevant factor extreme has margin one.

### G.3 Rule-tree aggregation

```text
ALL(children)          strength = minimum child strength
AT_LEAST(k, children) strength = kth-largest passed child strength
```

For `AT_LEAST(1)`, this is the strongest passing witness. Ties are resolved by
canonical condition ordinal. The selected k witnesses form the deterministic
decision proof; additional passing children remain supplemental supporting
evidence and cannot lower strength. No averaging or hidden factor weights are
used.

For Early Growth, strength is therefore the weaker of:

1. the weaker Growth Core margin; and
2. the strongest eligible independent confirmation-family margin.

This “weakest required clause” interpretation is more faithful than averaging
strong evidence over a barely satisfied required condition. It also prevents an
extra, weak optional confirmation from reducing a result.

### G.4 Severity

Severity is derived only from evidence strength:

```text
LOW     when 3 * evidence_strength < 1
MEDIUM  when 1 <= 3 * evidence_strength < 2
HIGH    when 2 <= 3 * evidence_strength <= 3
```

This uses exact rational thirds without a rounded decimal constant. Severity is
the magnitude of the named signal, including warnings; it is not confidence or
stock quality. Changing the formula or boundaries requires a new signal version.

## H. Missingness and coverage

### H.1 Evaluation rules

Each decision leaf requires an exact Phase 2.7 factor definition and value with:

- the declared subject scope;
- the exact research timestamp and effective date;
- the one manifest-pinned upstream factor build;
- status `VALID` and a finite value in `[-1,+1]`;
- an admitted quality record; and
- an exact relational factor-definition/value identity.

A direct required condition behaves as follows:

| Factor state | Signal treatment |
| --- | --- |
| Exact valid/admitted value | Evaluate the operator and store pass/fail. |
| Exact factor row is `MISSING` or no exact row exists | `MISSING`; never substitute zero or another build. |
| Exact factor row is `INVALID` or has impossible/non-finite content | `INVALID`. |
| Only another-build/version row exists | `MISSING/BUILD_MISMATCH`. |
| Exact factor admission cannot be verified | `MISSING/QUALITY_NOT_ADMITTED`. |

For `AT_LEAST(k)` groups:

- k valid passes make the group true even if non-selected alternatives are
  unavailable; the result is `VALID/REDUCED` when the whole rule is determinate.
- Fewer than k passes with every alternative valid and evaluated makes the group
  false.
- Fewer than k passes when missing alternatives could still satisfy the group is
  `MISSING`.
- Fewer than k passes when an invalid alternative could still satisfy the group
  is `INVALID`.

Required `ALL` leaves are all evaluated. A missing or invalid required leaf makes
the result unavailable rather than claiming a complete non-fire from partial
evidence.

### H.2 Stable result reasons

V1 result reasons are:

```text
REQUIRED_FACTOR_MISSING
INSUFFICIENT_CONFIRMATION_COVERAGE
INDUSTRY_CONTEXT_MISSING
QUALITY_NOT_ADMITTED
BUILD_MISMATCH
FACTOR_DEFINITION_MISMATCH
FACTOR_TIMESTAMP_MISMATCH
FACTOR_SUBJECT_MISMATCH
FACTOR_INVALID
NONFINITE_FACTOR
```

Malformed manifests, unknown definitions, inconsistent rule trees, unpinned
factor definitions, and changed content at an already published identity are
request/publication errors. They roll back the requested batch rather than being
laundered into an ordinary missing signal.

When several unavailable leaves matter, precedence is integrity/invalid,
definition/subject/timestamp, build, quality, industry context, then generic
missing/insufficient confirmation. Every condition retains its own exact reason
even when the final result uses the higher-precedence reason.

### H.3 Coverage

Coverage is descriptive and never called confidence:

```text
condition_coverage =
  admitted distinct factor inputs / declared distinct factor inputs
```

Persist additionally:

- declared, admitted, selected-proof, missing, and invalid factor counts;
- every selected factor's upstream Phase 2.7 `weight_coverage` and
  `coverage_state`;
- `minimum_selected_factor_weight_coverage`; and
- `coverage_state = FULL | REDUCED | NOT_APPLICABLE`.

`FULL` requires all declared factor inputs to be admitted and every selected
proof factor to be `FULL` upstream. A determinate result is `REDUCED` when a
non-decisive alternative/diagnostic is unavailable or a selected factor is
`VALID/REDUCED`. Missing/invalid results use `NOT_APPLICABLE`.

There is no arbitrary signal-level percentage gate. The Boolean rule tree itself
states exactly which evidence is sufficient, so optional absence cannot silently
change the rule's meaning.

## I. Quality propagation

Reuse the frozen order exactly:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

For a fired result:

```text
signal quality = weakest(
    signal-specific build/admission,
    all required ALL-clause factor values in the decision proof,
    the deterministically selected k witnesses for each AT_LEAST(k) clause,
)
```

For a valid non-fire, the proof contains the valid factor conditions required to
establish that the rule failed. Missing/invalid results retain attempted input
qualities but make no valid signal-quality claim.

Supplemental support, contradictions, warnings, and unselected alternatives
retain their own factor qualities but do not contaminate the primary result.
This is semantic isolation, not quality shopping: the canonical rule and
strength/ordinal policy choose the proof before quality is propagated.

Consequences:

- `estimate_confirmation_v1` is `DEVELOPMENT` when it consumes the currently
  synthetic Estimate Revision factor.
- Early Growth is `DEVELOPMENT` if Estimate Revision is its selected decisive
  confirmation and that factor is `DEVELOPMENT`.
- Early Growth is not downgraded merely because an unused or supplemental
  Estimate Revision factor is `DEVELOPMENT`; it may be established by a stronger
  qualified Market, Industry, or Fundamental Economics witness.
- A Fundamental Acceleration signal does not inherit market or estimate quality
  because it consumes neither factor family.
- Industry signals inherit the node factor's quality, which may itself reflect
  estimate and market evidence under the frozen Industry Strength definition.

Every v1 quality level is admitted for candidate/development research and
propagated honestly. A missing or malformed admission is
`QUALITY_NOT_ADMITTED`; low quality is not silently upgraded or rewritten as
missing. Definition lifecycle (`CANDIDATE`) remains separate from data quality.

## J. Storage model

### J.1 Decision

Use dedicated signal persistence. Do not store signals as `ResearchFeature`,
`FeatureValue`, `ResearchFactor`, or `FactorValue`. Reuse only the existing
content-addressed `research_build_manifests` adapter.

V1 needs four authoritative tables. A separate detector-definition or
detector-result table is unnecessary for the reasons in section E.

### J.2 `research_signals`

One immutable definition row:

```text
id
signal_id
signal_version
registry_key
display_code
name
description
subject_scope                 SECURITY | INDUSTRY_NODE
signal_class                  OPPORTUNITY | CONFIRMATION | WARNING |
                              CONTRADICTION | CONTEXT
lifecycle_state               CANDIDATE
detector_id
detector_version
runner_version
threshold_policy_version
strength_policy_version
severity_policy_version
coverage_policy_version
quality_policy_version
rule_payload                  canonical JSONB typed rule tree
definition_hash              SHA-256
created_at
```

Unique identities are `(signal_id, signal_version)`, `registry_key`, and
`definition_hash`. PostgreSQL guards reject `UPDATE` and `DELETE`.

### J.3 `research_signal_conditions`

One immutable row for every leaf in the canonical decision or diagnostic tree:

```text
id
research_signal_id
condition_key
ordinal
rule_path
family_key
role                          DECISION | DIAGNOSTIC_CONTRADICTION |
                              DIAGNOSTIC_WARNING
research_factor_id            exact FK to the frozen factor definition
operator                      GTE | LTE
threshold
required_for_direct_rule
condition_fingerprint
created_at
```

The canonical JSON rule tree is authoritative for nested `ALL` and
`AT_LEAST(k)` structure; relational leaf rows make factor dependencies and
thresholds queryable. Registration must prove that the ordered rows reproduce
the canonical tree and definition hash.

### J.4 `signal_results`

One immutable evaluation for one definition, subject, T, and signal build:

```text
id
research_signal_id
subject_scope
security_id                   nullable by scope
industry_node_id              nullable by scope
peer_group_snapshot_id        required for INDUSTRY_NODE
research_timestamp
effective_date
build_id                      FK to signal ResearchBuildManifest
upstream_factor_build_id      exact pinned Phase 2.7 build
snapshot_origin               LIVE | HISTORICAL_REPLAY | EDUCATIONAL
evaluation_status             VALID | MISSING | INVALID
reason
fired
evidence_strength
severity
declared_factor_count
admitted_factor_count
selected_proof_factor_count
missing_factor_count
invalid_factor_count
condition_coverage
minimum_selected_factor_weight_coverage
coverage_state                FULL | REDUCED | NOT_APPLICABLE
data_quality_level
admission
provenance
fingerprint
calculated_at
created_at
```

Database checks enforce subject exclusivity; valid/null combinations; finite
`[0,1]` strength and coverage; fired/severity rules; exact quality values; and
immutable publication. Partial unique indexes enforce one result for:

```text
(research_signal_id, security_id, research_timestamp, build_id)
(research_signal_id, industry_node_id, peer_group_snapshot_id,
 research_timestamp, build_id)
```

Indexes support security/node lookup by T, definition/class/fired-state queries,
and later signal co-occurrence analysis.

### J.5 `signal_result_conditions`

One row for every declared condition, including unavailable and failed evidence:

```text
id
signal_result_id
definition_condition_id
factor_value_id               nullable only when no exact artifact exists
actual_factor_value
factor_status
factor_reason
factor_quality
factor_coverage_state
factor_weight_coverage
operator
threshold
evaluated
passed
selected_for_decision
evidence_relation             SUPPORTING | NON_SUPPORTING | CONTRADICTING |
                              WARNING | UNAVAILABLE
condition_strength
exclusion_reason
provider_admission
selection_provenance
fingerprint
created_at
```

The row snapshots operator, threshold, and actual value while retaining exact
FKs. Those redundant values must equal the immutable definition and factor rows;
they make exports self-explanatory but cannot replace relational lineage.

A future signal-to-signal composition need may add an explicit typed input
junction. V1 must not hide dependent signal IDs in JSON.

## K. Provenance and build pinning

Starting from one signal result, an auditor must reconstruct:

```text
subject security/company or exact historical industry node
research timestamp T, effective date, origin, calculated_at, and created_at
signal logical ID/version, display code, class, lifecycle, and definition hash
detector ID/version and signal runner version
threshold, strength, severity, coverage, and quality policy versions
canonical signal ResearchBuildManifest and build_id
exact upstream factor build_id and manifest
every exact factor definition ID/version/hash expected by the rule
every exact FactorValue ID, subject, T, status/reason, value, quality, and coverage
the security-to-industry node/group mapping when industry context is used
every operator and threshold as defined and evaluated
every passed, failed, unavailable, selected, supplemental, contradictory, and
warning condition
the deterministic proof set and rule-margin arithmetic
final status, reason, fired state, strength, severity, coverage, quality, and
fingerprint
```

The signal build uses the existing `ResearchBuildManifest` format with a new
signal configuration section. It pins at minimum:

- the exact upstream Phase 2.7 build ID and factor manifest hash;
- exact factor registry keys, logical versions, and definition hashes used by
  the requested signal registry;
- all signal definition hashes and the exact registry set;
- detector and runner versions;
- threshold, strength, severity, coverage, and quality policy versions;
- research timestamp, effective date, origin, Git commit, schema revision,
  dependencies, and configuration; and
- the exact industry-context mapping policy.

The manifest pins the factor build; condition rows pin the exact factor value IDs.
All input factor values for one signal must belong to the one pinned build. A
newer factor version/build can never silently replace an old source.

The result fingerprint covers the ordered definition, subject, T/effective date,
both build identities, every declared condition and factor-value ID, actual
values/statuses/quality/coverage, decisions and proof selection, rule arithmetic,
final output, admission, and provenance.

## L. Versioning and immutability

Changing any of the following requires a new signal version:

- factor definition/version dependency;
- threshold or comparison operator;
- required condition, alternative, family grouping, or confirmation count;
- diagnostic warning/contradiction predicate;
- any economic veto, if a future version introduces one;
- missingness or determinacy behavior;
- evidence-strength or severity logic;
- coverage or quality-proof policy;
- subject scope, class, or industry-context mapping semantics; or
- rule-tree structure.

Changing which signals a detector emits or their evaluation order/atomicity
contract requires a new detector version. An execution-only implementation change
requires a new runner version and build even when signal semantics remain the
same. Wording-only documentation corrections do not mutate the canonical semantic
payload.

Definitions, condition definitions, results, and result conditions are
append-only. Exact registration is idempotent; a conflicting registry identity
fails. Exact replay reuses all IDs, fingerprints, values, reasons, and creation
times. Changed content at the same identity raises:

```text
PUBLISHED_SIGNAL_CONFLICT_USE_NEW_BUILD_OR_VERSION
```

Adopting `growth_v2` or any other future factor version requires a new signal
definition even if the displayed threshold happens to be unchanged. Historical
signal meaning is never rewritten.

## M. Worked example

Assume fictional **Asteria Semiconductor** has the requested factor values at T.
All are exact `VALID/FULL` values from one build. For quality illustration, assume
all are `PIT_QUALIFIED` except `estimate_revision_v1`, which is `DEVELOPMENT`:

| Factor | Value |
| --- | ---: |
| `growth_v1` | `+0.75` |
| `growth_acceleration_v1` | `+0.82` |
| `profitability_v1` | `+0.35` |
| `margin_expansion_v1` | `+0.60` |
| `cash_flow_quality_v1` | `+0.10` |
| `estimate_revision_v1` | `+0.72` |
| `market_leadership_v1` | `+0.68` |
| `industry_strength_v1` | `+0.55` |
| `market_fragility_v1` | `+0.70` |

The industry value belongs to Asteria's exact pinned node; its industry signals
are stored once at node scope and referenced by the company packet.

### M.1 Atomic results

| Signal | Result | Strength | Severity | Explanation |
| --- | --- | ---: | --- | --- |
| `fundamental_acceleration_v1` | FIRED | `min((.75-.50)/.50, (.82-.50)/.50) = .50` | MEDIUM | Both Growth and Acceleration are strongly positive. |
| `margin_inflection_v1` | FIRED | `(.60-.50)/.50 = .20` | LOW | Margin Expansion is beyond the strong threshold. |
| `estimate_confirmation_v1` | FIRED | `(.72-.50)/.50 = .44` | MEDIUM | Strong positive revision evidence; quality is `DEVELOPMENT`. |
| `market_confirmation_v1` | FIRED | `(.68-.50)/.50 = .36` | MEDIUM | Strong market leadership. |
| `industry_tailwind_v1` | FIRED | `(.55-.50)/.50 = .10` | LOW | The node has strong positive industry evidence. |
| `industry_headwind_v1` | NOT FIRED | null | null | `+.55` is not `<= -.50`. |
| `cash_flow_contradiction_v1` | NOT FIRED | null | null | Cash Flow Quality `+.10` is not negative. |
| `fragility_warning_v1` | FIRED | `(.70-.50)/.50 = .40` | MEDIUM | Observed fragility is strongly elevated. |

### M.2 Early Growth calculation

The Growth Core strength is:

```text
min(0.50, 0.64) = 0.50
```

Fundamental Economics candidate margins are:

```text
profitability:   (0.35 - 0.10) / 0.90 = 0.277777...
margin expansion:(0.60 - 0.10) / 0.90 = 0.555555...
cash flow:       (0.10 - 0.10) / 0.90 = 0
family witness = 0.555555...
```

The other independent-family margins are Estimates `0.44`, Market `0.36`, and
Industry `0.10`. Fundamental Economics supplies the strongest witness, so:

```text
early_growth_candidate_v1 strength
  = min(core 0.50, confirmation 0.555555...)
  = 0.50
```

`EARLY_GROWTH_CANDIDATE` fires at **MEDIUM** severity. Its decision proof uses
Growth, Growth Acceleration, and Margin Expansion. Under the stated quality
assumption it remains `PIT_QUALIFIED`; the separate Estimate Confirmation remains
`DEVELOPMENT` and is retained as supplemental support. `FRAGILITY_WARNING` also
fires. Nothing subtracts the warning from the opportunity signal.

### M.3 Change one factor: estimates become missing

If `estimate_revision_v1` becomes `MISSING` while all other factors remain the
same:

- `estimate_confirmation_v1` becomes `MISSING`, not neutral or not-fired;
- Early Growth still fires through Fundamental Economics (and also has valid
  Market/Industry support);
- its strength remains `0.50` because the decisive proof is unchanged; and
- its condition coverage becomes reduced and the missing estimate condition and
  reason remain visible.

### M.4 Change one factor: cash flow contradicts growth

If only `cash_flow_quality_v1` changes from `+0.10` to `-0.45`:

```text
cash weakness margin = (-0.10 - (-0.45)) / 0.90 = 0.388888...
cash_flow_contradiction strength = min(0.50, 0.388888...) = 0.388888...
```

Cash Flow Contradiction now fires at MEDIUM severity. Early Growth still fires
because Growth Core and Margin Expansion confirmation remain intact. The packet
therefore says, explicitly, “early-growth pattern present, cash flow contradicts
it, and market fragility is elevated.”

### M.5 Change one factor: acceleration weakens

If `growth_acceleration_v1` instead falls from `+0.82` to `+0.20`, with other
original values unchanged, Fundamental Acceleration and Early Growth are valid
non-fires because the required core condition fails. Margin Inflection, Estimate
Confirmation, Market Confirmation, Industry Tailwind, and Fragility Warning can
still fire. The system preserves the vector rather than forcing a replacement
label.

## N. Test matrix

### Registry and rule contract

- Exact nine registry keys, logical IDs/versions, display codes, classes, scopes,
  lifecycle, detector IDs/versions, and rejection of extras.
- Exact factor-definition FKs, condition keys/order/paths, operators, thresholds,
  family groupings, required/alternative structure, and diagnostic roles.
- Exact canonical rule payload/hash registration, idempotency, and failure on
  tampering, duplicate ordinals, unknown factors, or invalid trees.
- Proof that no definition references a direct feature or a factor version other
  than the exact frozen v1 dependency.

### Thresholds, Boolean rules, and strength

- Every `+0.50`, `-0.50`, `+0.10`, and `-0.10` boundary one exact Decimal unit
  below, exactly at, and one unit above.
- All five factor-state buckets, including exact zero and both endpoints.
- Inclusive `GTE`/`LTE`, `ALL`, `AT_LEAST(1)`, a generic `AT_LEAST(k)` reference
  test, nested groups, canonical tie-breaking, and independence-family counting.
- Exact leaf margins at threshold/interior/extreme; `ALL` minimum;
  `AT_LEAST(k)` kth-largest; no optional-evidence dilution.
- Severity boundaries tested as exact `3*s` comparisons.
- Non-fired/missing/invalid results have null strength and severity; a fired
  just-at-threshold result has numeric strength zero.

### Missingness, coverage, and quality

- Required factor missing/invalid; only wrong-build value; wrong T/scope/version;
  missing industry context; unverified admission; non-finite corruption.
- Missing estimate plus valid Market/Industry/Economics confirmation still fires
  Early Growth with reduced coverage.
- No confirmation pass plus a potentially decisive missing family produces
  `INSUFFICIENT_CONFIRMATION_COVERAGE`; all valid failures produce a valid
  non-fire.
- Selected-proof, supplemental, contradicting, warning, and unavailable evidence
  relations are exact.
- Full/reduced/not-applicable coverage; distinct input counts; upstream
  `VALID/REDUCED` factor propagation; no missing-to-zero conversion.
- Weakest selected-proof quality; synthetic estimate isolation; node-factor
  quality; unselected low-quality alternatives do not contaminate unrelated
  signals.

### Economic behavior

- Strong Growth without Acceleration does not fire Fundamental Acceleration or
  Early Growth.
- Strong Acceleration with weak Growth behaves likewise.
- Fundamental Acceleration works without estimate and market factors.
- Each of the four independent confirmation families can establish Early Growth
  alone; multiple Fundamental Economics members still count as one family.
- Margin Inflection during negative Growth fires with explicit cost-led
  contradiction context.
- Tailwind and Headwind boundaries and mutual exclusivity.
- Market Confirmation and Fragility Warning coexist.
- Early Growth with Cash Flow Contradiction, Industry Headwind, Margin
  Deterioration, or Fragility Warning retains both sides of the evidence.
- The complete Asteria example and all three changes in section M are golden
  hand calculations, not threshold-fitting fixtures.

### PIT, persistence, and operations

- Exact upstream factor build/T/effective-date resolution and no newer-build or
  older-snapshot fallback.
- Exact node/group mapping at T; no present-day classification substitution.
- Complete relational lineage from signal result to every factor value and its
  existing component chain.
- Persist fired, not-fired, missing, and invalid results in one atomic requested
  batch.
- Exact rerun preserves every ID/value/reason/fingerprint/timestamp; changed
  content conflicts; new signal version or build publishes separately.
- PostgreSQL rejects update/delete; FK/subject/status/value/strength/quality
  constraints; injected rollback; two-writer concurrent publication; no reader
  sees partial condition rows.
- Fresh migration lifecycle, metadata parity, focused signal tests, frozen factor
  regression, and full repository suite.
- 1,000/5,000/10,000-security set-based scale proof with SQL statement count,
  runtime, peak RSS, PostgreSQL temp I/O, plans/index use, exact cardinalities,
  and no per-security query or industry Cartesian multiplication.

No test may claim investment effectiveness, probability calibration, provider
qualification, or production readiness.

## O. Historical backtesting compatibility

This design permits a later historical replay to ask exactly:

> What signal results would have existed at T using only the factor artifacts
> and industry mapping eligible in the pinned build at T?

Compatibility comes from immutable signal definitions, exact upstream factor
build/value IDs, exact T and subject mapping, persisted non-fires and unavailable
results, explicit origin, and deterministic fingerprints. A historical replay
created later is labeled `HISTORICAL_REPLAY`; it is never represented as a live
prediction issued at T.

Phase 2.8 performs no outcome join. Phase 2.9 may measure future delivered growth,
margin, revisions, excess return, drawdown, decay, and regime behavior for each
stable signal version. Missing results and excluded companies remain in cohort
accounting. Future outcome data can never enter a signal build or alter a signal
artifact.

Stable signal IDs and result rows also support later overlap diagnostics: pairwise
co-occurrence, conditional outcome tables, incremental family ablations, and
coverage/quality-stratified comparisons. Correlation is not a reason to merge
signals before those tests.

## P. Prediction-ledger compatibility

Phase 2.8 does not implement a prediction ledger. A later immutable research
packet can reference or copy:

- signal result IDs and fingerprints for the complete registered set;
- fired and not-fired decisions;
- missing/invalid results and reasons;
- signal/detector/runner/policy versions;
- exact factor build and value lineage;
- support, contradiction, and warning condition rows;
- evidence strength, severity, coverage, and quality; and
- the canonical signal build manifest.

The packet must add what a signal deliberately lacks: origin as an actual or
reconstructed prediction, objective family, named future claim, horizon/deadline,
benchmark/execution policy, thesis assertions, and outcome definition. Merely
having `EARLY_GROWTH_CANDIDATE` at T is not a claim of future success.

The exact signal registry set is pinned in the build and published atomically, so
a later packet can prove that absent positive signals were genuine non-fires or
unavailable evaluations rather than rows omitted after the outcome was known.

## Q. Scale strategy

At full coverage, 10,000 securities produce approximately 70,000
security-scoped results for the seven security signals plus two results per
requested industry node, not 90,000 security copies of node context. The expected
condition-row volume remains modest relative to the frozen factor component store.

Implementation should:

1. validate the signal manifest, exact registry, definitions, and upstream factor
   build once;
2. load exact factor definitions once;
3. retrieve all required security FactorValues in bounded set-based batches;
4. retrieve and map node FactorValues/group context once per distinct node;
5. evaluate immutable Decimal rule trees in memory without database round trips
   per security or condition;
6. bulk-insert results and all condition decisions in bounded batches inside one
   caller transaction;
7. resolve exact reruns by immutable identity/content comparison; and
8. return caller order without retaining large JSON payloads unnecessarily.

The public API should remain repository-compatible with:

```text
calculate_signals(
    subjects,                    # explicit SECURITY and INDUSTRY_NODE identities
    research_timestamp,
    research_build_manifest,     # signal build pinning one upstream factor build
    signal_registry_keys,
) -> immutable signal results
```

PostgreSQL remains sufficient. Reuse the measured bounded-batch pattern from the
factor runner, but choose actual batch sizes from Phase 2.8 measurement rather
than copying `250` by assumption. The 8 GB development host provides headroom,
not permission for full-universe object graphs, N+1 reads, or daily Cartesian
materialization.

The immutable semantic identity is the cache key. Later operational optimization
may consider staging/COPY or partitioning only after measured need and without
changing rule meaning, atomicity, lineage, or idempotency.

## R. Future calibration path

V1 thresholds and strength are frozen candidate priors. Phase 2.8 has no
self-calibration and no outcome-driven updates.

Phase 2.9 should replay the fixed definitions on qualified historical evidence and
record versioned outcomes separately. Phase 2.10 should pre-register objective,
horizon, eligible cohort, sample/maturity requirements, downside and coverage
limits, and a hypothesis budget before comparing:

- base rates and no-signal baselines;
- each atomic signal alone;
- Early Growth versus Fundamental Acceleration;
- confirmation-family additions and ablations;
- warning/contradiction interactions;
- evidence-strength and severity buckets; and
- sectors, sizes, regimes, providers, coverage states, and quality levels.

Training, validation, untouched chronological holdout, overlap controls, and
prospective shadow evidence follow the existing champion/challenger contract.
Calibration, if eventually supported, names one outcome and horizon and is stored
as a separate versioned artifact. It does not rename evidence strength as a
probability.

Evidence may later justify `signal_v2`, a challenger detector, retirement, or an
additional warning. It may never rewrite v1 thresholds or historical results.

## S. Open decisions

There are **no blocking Phase 2.8 architecture decisions left open**.

The following are genuine deferred evidence/release decisions, not ambiguities in
the v1 contract:

- whether historical and forward evidence eventually promotes any candidate or
  supports different v2 thresholds/family requirements;
- when external estimate, market, classification, and other providers qualify
  above their current levels;
- the exact Phase 2.9 outcome, execution, universe, and prediction-ledger schemas;
  and
- whether future qualified valuation, business-quality, event, or macro factors
  justify new signal definitions after their own architecture reviews.

Migration revision names and checkpoint sequencing are implementation planning
details. They may not change the semantics in this document.

## T. Recommendation

**SAFE TO IMPLEMENT PHASE 2.8**

Implementation must preserve the exact candidate registry, rule thresholds,
proof-based strength and quality, explicit missingness, warning coexistence,
factor-build lineage, immutable storage, and phase boundary above. It must stop
before ranking, prediction/outcome evaluation, calibration, portfolio action, or
Phase 2.9.
