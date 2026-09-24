# Phase 2.8 — Signal and Detector Implementation Contract

Status: **ARCHITECTURE APPROVED; IMPLEMENTATION SPEC FROZEN**.

This is the concise, authoritative implementation contract for Phase 2.8. It
preserves every decision in
[`phase-2.8-signal-detector-architecture-review.md`](phase-2.8-signal-detector-architecture-review.md).
If a detail is not repeated here, that architecture review remains authoritative.
Foundation and Phases 2.2, 2.4, 2.5, 2.6, and 2.7 remain frozen. Phase 2.8 reads
their immutable artifacts and never recalculates or changes a feature or factor.

The implementation starts from approved checkpoint
`84066fa05cc6bc4e3dcb49089755c1779f225b05`, with Alembic head `p27001` and the
reproduced baseline of 736 passing tests and 8 existing warnings.

## 1. Scope and phase boundary

Phase 2.8 v1 registers and evaluates exactly these nine candidate signals:

| Registry key | Logical ID / version | Scope | Class | Detector |
| --- | --- | --- | --- | --- |
| `fundamental_acceleration_v1` | `fundamental_acceleration` / `1` | `SECURITY` | `OPPORTUNITY` | `factor_pattern_detector_v1` |
| `margin_inflection_v1` | `margin_inflection` / `1` | `SECURITY` | `OPPORTUNITY` | `factor_pattern_detector_v1` |
| `estimate_confirmation_v1` | `estimate_confirmation` / `1` | `SECURITY` | `CONFIRMATION` | `factor_pattern_detector_v1` |
| `market_confirmation_v1` | `market_confirmation` / `1` | `SECURITY` | `CONFIRMATION` | `factor_pattern_detector_v1` |
| `industry_tailwind_v1` | `industry_tailwind` / `1` | `INDUSTRY_NODE` | `CONTEXT` | `factor_pattern_detector_v1` |
| `industry_headwind_v1` | `industry_headwind` / `1` | `INDUSTRY_NODE` | `CONTEXT` | `factor_pattern_detector_v1` |
| `cash_flow_contradiction_v1` | `cash_flow_contradiction` / `1` | `SECURITY` | `CONTRADICTION` | `factor_pattern_detector_v1` |
| `fragility_warning_v1` | `fragility_warning` / `1` | `SECURITY` | `WARNING` | `factor_pattern_detector_v1` |
| `early_growth_candidate_v1` | `early_growth_candidate` / `1` | `SECURITY` | `OPPORTUNITY` | `early_growth_detector_v1` |

Every signal starts in lifecycle state `CANDIDATE`. The two detector identities
are exactly `factor_pattern_detector_v1` and `early_growth_detector_v1`; their
logical IDs/versions are the key without `_v1` and integer `1`. The common runner
version is `signal_detector_runner_v1`.

Features measure. Factors summarize evidence. Signals interpret frozen factor
patterns. Signals are not probabilities, recommendations, rankings, portfolio
decisions, or AI judgments.

Phase 2.8 does not implement a buy/sell signal, overall score, stock rank,
probability of success, expected return, portfolio weight, valuation/news/AI
signal, prediction ledger, outcome ledger, backtester, calibration, or Phase 2.9
behavior.

## 2. Exact frozen factor boundary

The only v1 inputs are these exact Phase 2.7 registry keys:

```text
growth_v1
growth_acceleration_v1
profitability_v1
margin_expansion_v1
cash_flow_quality_v1
estimate_revision_v1
market_leadership_v1
industry_strength_v1
market_fragility_v1
```

The first eight are security-scoped except `industry_strength_v1`, which is
`INDUSTRY_NODE` scoped. `market_fragility_v1` keeps its frozen orientation:
`+1` means more observed fragility. A signal condition references the exact
immutable `ResearchFactor` definition and `FactorValue` artifact; it never copies
a factor formula, reads a raw feature directly, or substitutes a newer factor
version/build.

All inputs to one result belong to one manifest-pinned upstream factor build.
Security use of Industry Strength must resolve the security's exact historical
node and peer-group snapshot at the same research timestamp, effective date, and
factor build. Node context is never generalized or copied into a fabricated
security factor.

## 3. Versioned policies

The exact v1 policy identities are:

```text
threshold_policy_version = signal_thresholds_v1
strength_policy_version  = signal_rule_margin_v1
severity_policy_version  = signal_severity_thirds_v1
coverage_policy_version  = signal_coverage_v1
quality_policy_version   = signal_selected_proof_quality_v1
runner_version           = signal_detector_runner_v1
```

Changing any policy's semantics requires a new signal version. Changing the
detector output set or evaluation-order/atomicity contract requires a new detector
version. An execution-only change requires a new runner version and build.

All numeric rule arithmetic uses finite exact `Decimal` values and the repository's
38-digit `ROUND_HALF_EVEN` context. Binary floats, NaN, infinity, clipping,
fitted thresholds, and dynamic expressions are prohibited.

## 4. Typed rule tree

The code-owned immutable tree has only four node types:

```text
Comparison(factor_registry_key, GTE | LTE, threshold)
All(children)
AtLeast(required_count, children)
Diagnostic(comparison, CONTRADICTION | WARNING)
```

`Diagnostic` is a leaf role in the definition, not a decision operator. The
decision root contains only `Comparison`, `All`, and `AtLeast`. `AtLeast(1, ...)`
provides the only v1 any-of behavior. V1 has no general expression language,
database-authored logic, `eval`, dynamic Python, arbitrary SQL, negation, or
user-provided code.

Every child and leaf has a canonical zero-based ordinal. Canonical payloads
preserve node kind, grouping, ordinals, family keys, roles, required/direct-rule
metadata, exact factor ID/version/hash, operator, and Decimal threshold.

The shared factor-state cutoffs are inclusive as shown:

| State | Interval |
| --- | --- |
| `STRONG_NEGATIVE` | `-1 <= x <= -0.50` |
| `NEGATIVE` | `-0.50 < x <= -0.10` |
| `NEUTRAL` | `-0.10 < x < +0.10` |
| `POSITIVE` | `+0.10 <= x < +0.50` |
| `STRONG_POSITIVE` | `+0.50 <= x <= +1` |

## 5. Exact v1 definitions

### 5.1 Atomic and context definitions

```text
fundamental_acceleration_v1 = ALL(
    growth_v1 >= +0.50,
    growth_acceleration_v1 >= +0.50,
)

margin_inflection_v1 = margin_expansion_v1 >= +0.50
diagnostic contradiction: growth_v1 <= -0.10

estimate_confirmation_v1 = estimate_revision_v1 >= +0.50

market_confirmation_v1 = market_leadership_v1 >= +0.50

industry_tailwind_v1 = industry_strength_v1 >= +0.50

industry_headwind_v1 = industry_strength_v1 <= -0.50

cash_flow_contradiction_v1 = ALL(
    growth_v1 >= +0.50,
    cash_flow_quality_v1 <= -0.10,
)

fragility_warning_v1 = market_fragility_v1 >= +0.50
```

Tailwind and headwind cannot both pass for the same exact factor state. Margin
Inflection's negative-growth diagnostic is non-gating. No separate signal's
result is an input to another signal.

### 5.2 Early Growth

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
    MARKET: market_leadership_v1 >= +0.50,
    INDUSTRY: industry_strength_v1 >= +0.50,
    FUNDAMENTAL_ECONOMICS: FUNDAMENTAL_ECONOMICS_CONFIRMATION,
)

early_growth_candidate_v1 = ALL(
    GROWTH_CORE,
    INDEPENDENT_CONFIRMATION,
)
```

The three Fundamental Economics comparisons count as one confirmation family.
Growth and Acceleration together count as the one required Fundamental Growth
family. Early Growth therefore requires Fundamental Growth plus at least one of
Expectations, Market, Industry, or Fundamental Economics.

The following Early Growth diagnostics are always attempted and persisted but
never veto the decision:

```text
margin_expansion_v1 <= -0.10  -> MARGIN_DETERIORATION       (CONTRADICTION)
cash_flow_quality_v1 <= -0.10 -> CASH_FLOW_WEAKNESS         (CONTRADICTION)
profitability_v1 <= -0.50     -> WEAK_PROFITABILITY_LEVEL   (CONTRADICTION)
industry_strength_v1 <= -0.50 -> INDUSTRY_HEADWIND          (CONTRADICTION)
market_fragility_v1 >= +0.50  -> FRAGILITY_WARNING          (WARNING)
```

There are no economic vetoes in v1. Early Growth may coexist with cash-flow
contradiction, margin deterioration, industry headwind, weak profitability, or
fragility warning.

## 6. Evaluation status and determinacy

Every result has exactly:

```text
evaluation_status = VALID | MISSING | INVALID
fired              = true | false | null
evidence_strength  = Decimal [0,1] | null
severity           = LOW | MEDIUM | HIGH | null
```

`VALID` always has a Boolean `fired`. Only `VALID/fired=true` has strength and
severity. A fired condition exactly at threshold has numeric strength zero.
`VALID/fired=false`, `MISSING`, and `INVALID` have null strength and severity.

Every decision leaf is evaluated. A direct required leaf is:

- evaluated normally only from an exact admitted `VALID` factor value;
- `MISSING` for an absent or upstream `MISSING` exact artifact;
- `INVALID` for upstream `INVALID`, non-finite, or impossible content;
- `MISSING/BUILD_MISMATCH` when only another build is present; and
- `MISSING/QUALITY_NOT_ADMITTED` when exact admission cannot be verified.

For `ALL`, any missing or invalid required child makes the decision unavailable;
a partial valid failure is not enough to claim a complete non-fire. When every
child is valid, all must pass for the group to pass.

For `AT_LEAST(k)`:

- at least k valid passes make the group valid/true even when unselected
  alternatives are unavailable;
- fewer than k passes with every alternative valid makes it valid/false;
- fewer than k passes with potentially decisive missing alternatives is missing;
  and
- fewer than k passes with a potentially decisive invalid alternative is invalid.

Diagnostics never change decision determinacy. Their unavailable states remain
explicit and may reduce descriptive coverage.

Stable result reasons are:

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

When several unavailable conditions matter, precedence is invalid/integrity,
definition/subject/timestamp, build, quality, industry context, then generic
missing/insufficient confirmation. Each condition retains its own reason.
Malformed requests/manifests/trees and published-content conflicts are errors and
roll back the requested batch.

Missing Estimate Revision does not block Early Growth when another independent
family passes. If no family passes and a missing family could still satisfy the
rule, the result is `MISSING/INSUFFICIENT_CONFIRMATION_COVERAGE`, never a false
result based on a fabricated zero.

## 7. Deterministic proof selection

All leaves are persisted, but only the canonical decision proof controls strength
and signal quality:

- a passing `Comparison` selects itself;
- a passing `ALL` selects every child proof;
- a passing `AT_LEAST(k)` orders passing children by descending child strength,
  then canonical child ordinal, and selects exactly the first k proofs;
- a valid false `ALL`/`AT_LEAST` retains the valid evaluated conditions needed to
  establish the failure; and
- unavailable and diagnostic conditions are never selected into a fired proof.

Nested selection is recursive. Thus Fundamental Economics selects its strongest
passing member; Independent Confirmation then compares that family witness with
Expectations, Market, and Industry. Database row order can never select a proof.
Additional passing branches remain supplemental supporting evidence.

Changing this selection or tie policy requires a new signal version.

## 8. Evidence strength and severity

For a passed `x >= theta` leaf:

```text
m_ge(x, theta) = (x - theta) / (1 - theta)
```

For a passed `x <= theta` leaf:

```text
m_le(x, theta) = (theta - x) / (theta + 1)
```

At the threshold the margin is zero; at the relevant `-1` or `+1` extreme it is
one. Failed/unavailable leaves have no strength.

```text
ALL(children)          strength = minimum child strength
AT_LEAST(k, children) strength = kth-largest passed child strength
```

For Early Growth, strength is the weaker of the Growth Core's weaker margin and
the strongest eligible independent-family witness. No averaging, hidden factor
weights, or optional-evidence dilution is allowed.

Severity uses exact rational thirds:

```text
LOW     when 3 * evidence_strength < 1
MEDIUM  when 1 <= 3 * evidence_strength < 2
HIGH    when 2 <= 3 * evidence_strength <= 3
```

Evidence strength is rule-margin magnitude, not confidence, probability, expected
return, or stock quality.

## 9. Coverage and quality

```text
condition_coverage = admitted distinct factor inputs / declared distinct factor inputs
```

Persist declared, admitted, selected-proof, missing, and invalid distinct-factor
counts; each selected factor's upstream coverage state and weight coverage; and
the minimum selected factor weight coverage.

Signal coverage is `FULL | REDUCED | NOT_APPLICABLE`. `FULL` requires every
declared input admitted and every selected proof factor upstream `FULL`.
A determinate result is `REDUCED` when a non-decisive alternative/diagnostic is
unavailable or a selected factor is upstream `VALID/REDUCED`. Missing/invalid
results are `NOT_APPLICABLE`. There is no extra percentage admission gate.

Reuse this exact quality order:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

For a fired result, signal quality is the weakest of the signal build/admission,
every required `ALL` proof factor, and the selected k witnesses for every
`AT_LEAST(k)`. For a valid non-fire, it is the weakest valid factor evidence
required to establish failure. Missing/invalid results retain attempted condition
qualities but make no signal-quality claim.

Unselected alternatives, supplemental support, diagnostics, contradictions, and
warnings retain their own quality but do not contaminate the result. Proof is
chosen by strength and ordinal before quality is propagated; quality cannot choose
the most favorable branch. A selected synthetic Estimate Revision makes Early
Growth `DEVELOPMENT`; an unselected synthetic estimate does not downgrade a
Market-, Industry-, or Fundamental-Economics-established result.

## 10. Dedicated relational storage

Use exactly four authoritative Phase 2.8 tables. Do not store signals in
`ResearchFeature`, `FeatureValue`, `ResearchFactor`, or `FactorValue`. Do not add
a `DetectorResult` table.

### `research_signals`

One immutable definition with logical ID/version, registry key, display code,
name/description, scope, class, `CANDIDATE` lifecycle, detector ID/version, every
policy/runner version, canonical JSONB rule payload, SHA-256 definition hash, and
creation time. Unique identities are `(signal_id, signal_version)`, registry key,
and definition hash.

### `research_signal_conditions`

One immutable row per decision or diagnostic leaf with definition FK, condition
key, global ordinal, rule path, family key, role (`DECISION`,
`DIAGNOSTIC_CONTRADICTION`, or `DIAGNOSTIC_WARNING`), exact frozen
`ResearchFactor` FK, `GTE | LTE`, Decimal threshold, direct-required flag,
condition fingerprint, and creation time. Registration proves the ordered rows
reproduce the canonical tree/hash.

### `signal_results`

One immutable result per definition, scoped subject, research timestamp, and
signal build. It stores security or exact node/group identity, T/effective date,
signal build ID, upstream factor build ID, origin, evaluation status/reason,
nullable fired/strength/severity fields under the rules above, coverage counts and
state, nullable valid-result quality/admission, compact provenance, content
fingerprint, calculation time, and creation time.

Partial unique identities are:

```text
(research_signal_id, security_id, research_timestamp, build_id)
(research_signal_id, industry_node_id, peer_group_snapshot_id,
 research_timestamp, build_id)
```

### `signal_result_conditions`

One immutable row per declared condition, including failed and unavailable
conditions. It stores result and definition-condition FKs; nullable exact
`FactorValue` FK only when no exact artifact exists; actual value; factor
status/reason/quality/coverage; snapshotted operator/threshold; evaluated/passed;
selected-for-decision; evidence relation (`SUPPORTING`, `NON_SUPPORTING`,
`CONTRADICTING`, `WARNING`, or `UNAVAILABLE`); nullable passed-leaf strength;
exclusion reason; provider admission; selection/proof provenance; fingerprint;
and creation time.

Database checks and PostgreSQL triggers enforce scope, exact valid/null
combinations, finite ranges, FK consistency, complete lineage, and rejection of
`UPDATE`/`DELETE` on all four tables.

## 11. Build pinning, provenance, and fingerprints

Reuse the canonical content-addressed `ResearchBuildManifest`. Its signal
configuration pins at least:

- exact upstream factor build ID and factor manifest hash;
- exact factor registry keys, logical versions, and definition hashes;
- exact signal registry set and definition hashes;
- detector IDs/versions and runner version;
- all policy versions in section 3;
- T, effective date, snapshot origin, Git commit, schema revision, dependencies,
  datasets, providers, and configuration;
- signal-specific quality admission; and
- exact security-to-industry context mapping policy.

Condition rows pin exact FactorValue IDs. A result must reconstruct its subject,
T and build identities; signal/detector/policy versions; full canonical definition;
all expected factors and exact actual artifacts; node/group mapping; every passed,
failed, unavailable, selected, supplemental, contradictory, and warning condition;
proof arithmetic; coverage and quality; and final output.

The result fingerprint covers the ordered definition, subject, T/effective date,
both builds, every condition and factor artifact/value/status/quality/coverage,
proof selection and arithmetic, final result, admission, and provenance.

## 12. PIT, publication, and immutability

All factor inputs must match exact T, effective date, definition/version, subject,
scope, and upstream build. There is no future-value, older-snapshot, other-build,
or present-day-classification fallback.

Resolve and evaluate the whole requested registry before publication. Publish
fired, not-fired, missing, and invalid results plus every condition row in one
savepoint/caller transaction. A failure rolls back the requested batch. A reader
must never observe a result without its complete condition graph.

Exact replay reuses every result/condition ID, fingerprint, value, reason, proof,
and creation/calculation time. Changed content at the same identity raises:

```text
PUBLISHED_SIGNAL_CONFLICT_USE_NEW_BUILD_OR_VERSION
```

A different signal version, detector version, signal build, or upstream factor
build creates a distinct artifact and never mutates history. Concurrent writers
converge on one complete graph.

## 13. Runner and scale contract

The repository-compatible API is:

```text
calculate_signals(
    subjects,
    research_timestamp,
    research_build_manifest,
    signal_registry_keys,
) -> immutable signal results
```

The runner validates manifests/definitions once, loads exact factor definitions
once, batch-retrieves factor values, resolves node context once per distinct node,
evaluates Decimal trees in memory, and bulk-publishes bounded result/condition
batches. It never invokes `FactorRunner` or any upstream calculator.

At 10,000 securities the seven security-scoped definitions produce about 70,000
security results; the two node-scoped context definitions are calculated once per
requested node, not copied per security. SQL statements must grow by bounded
batches/source families rather than by security or condition. Measure 1k/5k/10k
runtime, SQL, RSS, PostgreSQL temp I/O, and representative plans before choosing
any optimization.

## 14. Mandatory verification

Implementation tests cover at least:

- exact nine signals/two detectors, metadata, definitions, ordered conditions,
  rules, factor FKs, hashes, idempotent registration, and rejection of extras;
- every `+0.50`, `-0.50`, `+0.10`, and `-0.10` threshold one Decimal unit below,
  at, and above; inclusive operators and tailwind/headwind mutual consistency;
- `ALL`, `AT_LEAST(1)`, generic `AT_LEAST(k)`, nesting, deterministic strength/
  ordinal proof selection, exact margins, and exact severity boundaries;
- required/optional missingness, invalidity, false-versus-missing distinction,
  wrong factor build/version/T/scope, future factors, and missing industry context;
- all four Early Growth confirmation families, missing estimate with alternate
  confirmation, no confirmation, required-core absence, and family counting;
- selected high-quality and selected development proofs, unselected development
  isolation, selected upstream reduced coverage, and unavailable diagnostics;
- warning/contradiction coexistence with opportunity signals and no economic veto;
- complete relational lineage, fingerprints, immutable definitions/results/
  conditions, new version/build isolation, exact replay, conflict, atomic rollback,
  concurrent one-winner publication, and no partial reads;
- a production-factor-artifact T1/T2 synthetic lifecycle without manual signal
  injection; and
- fresh migration/downgrade/upgrade/metadata verification, full regression, and
  the 10,000-security structural scale proof.

No test may claim alpha, prediction quality, probability calibration, provider
qualification, or production readiness.

## 15. Release limitations and checkpoint boundary

Provider qualification, licensing, broad historical coverage, complete identity/
delisting history, empirical validation, calibration, promotion, and production
readiness remain external or later-phase gates. They cannot weaken PIT,
missingness, quality, provenance, proof selection, or any frozen definition.

The implementation sequence is checkpoints 2.8.1 through 2.8.9 in the approved
architecture handoff. Each coherent checkpoint requires focused and regression
tests, a `PROJECT_STATE.md` update, commit, push, `HEAD == origin/master`, and a
clean worktree. Phase 2.8 stops after its internal audit. Phase 2.9 must not begin.
