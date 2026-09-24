# Phase 2.6 — Peer / Industry Implementation Contract

Status: **ARCHITECTURE APPROVED; IMPLEMENTATION SPEC FROZEN**.

This document is the concise implementation contract for Phase 2.6. The complete
design rationale remains in
[`phase-2.6-peer-industry-architecture-review.md`](phase-2.6-peer-industry-architecture-review.md).
If a detail is not repeated here, the architecture review remains authoritative.
Foundation and Phase 2.2, 2.4, and 2.5 temporal semantics, formulas, publication
contracts, and provider gates remain frozen.

## 1. Scope and release boundary

Phase 2.6 v1 implements only:

- provider-neutral, versioned four-level industry taxonomy structure;
- append-only PIT security-to-industry classification evidence;
- sealed historical classification and eligibility datasets;
- immutable, versioned classification peer policies;
- deterministic same-node peer resolution, with same-sub-industry canonical;
- separate membership and feature-observation eligibility;
- peer median, target-minus-peer-median, and ascending midrank percentile;
- ten approved peer-relative percentile features;
- eight focused industry snapshot metrics;
- relational provenance, provider admission, and weakest-input quality;
- synthetic PostgreSQL lifecycle, idempotency, and scale tests.

Phase 2.6 does **not** implement factors, signals, rankings, valuation, economic
moats, business-quality peers, AI-generated peers, news/event peers, portfolios,
backtesting, calibration, or Phase 2.7. It does not change the frozen
`rel_ret_63_ind` basket or recalculate Phase 2.2, 2.4, or 2.5 source metrics.

No proprietary GICS structure or assignments are stored in the repository.
Production GICS use requires a licensed, qualified historical source. Tests use
fictional `SYNTHETIC_GICS_STYLE_V1` evidence and always produce `DEVELOPMENT`
quality.

## 2. Reused frozen foundations

Phase 2.6 adds no competing temporal or publication framework.

| Existing foundation | Phase 2.6 use |
| --- | --- |
| `HistoricalUniverse` | Effective interval, knowledge time, receipt time, evidence key/version, source, timing policy, and raw lineage for both classification and primary-equity eligibility. |
| `SecurityIdentityHistory` | Historical permanent-security identity and lifecycle; never current `Security.is_active`. |
| `ResearchBuildManifest` | Pins datasets, provider qualifications, source feature builds, policies, code versions, configuration, and research timestamp. Its SHA-256 `build_id` is the build identity. |
| `ResearchFeature` / `FeatureValue` | Frozen source-feature inputs and optional immutable projection of the ten canonical percentile outputs. |
| Provider qualification/runtime gates | Fail-closed source admission, named development overrides, quality level, limitations, and survivorship qualification. |
| `publish_feature` semantics | Idempotent same-content publication; changed content requires another build/version and never overwrites history. |

Current `Company.sector`, `Company.industry`, `Company.sic`, ticker text, and
display names are not historical classification identity.

## 3. Canonical taxonomy and persistence

The internal schema is taxonomy-neutral and supports the GICS-style hierarchy:

```text
SECTOR → INDUSTRY_GROUP → INDUSTRY → SUB_INDUSTRY
```

Use stable internal IDs. Provider codes, names, and definitions are release
metadata, not identity. The implementation adds repository-compatible forms of:

| Table | Frozen responsibility |
| --- | --- |
| `industry_taxonomies` | Stable taxonomy family and rights/admission metadata; unique `taxonomy_key`. |
| `industry_taxonomy_releases` | Immutable logical release plus increasing evidence version, effective interval, announcement/publication/availability/receipt, source/profile, raw evidence, supersession, and fingerprint. |
| `industry_nodes` | Stable concept identity scoped to a taxonomy and fixed level. |
| `industry_node_versions` | Exact release-specific code, name, definition, and immediate parent. |
| `industry_node_lineage` | Explicit continuity, split, merge, or replacement evidence; never implicit peer membership. |
| `security_industry_classifications` | One-to-one extension of a `HistoricalUniverse` classification row to an exact release and leaf node. It contains no duplicate temporal columns. |
| `universe_evidence_datasets` | Immutable input corpus in `BUILDING`, `COMPLETE`, or `FAILED` state. |
| `universe_evidence_dataset_rows` | Explicit inclusion of immutable `HistoricalUniverse` rows; additions are forbidden after sealing. |
| `peer_policies` | Immutable key/version and canonical policy payload/fingerprint. |

Each v1 release is a strict acyclic four-level tree. Every non-sector node has one
parent at the immediately broader level, and a security classification points to
a sub-industry present in that exact release. A real concept split/merge/replacement
uses new stable node IDs and explicit lineage. Similar names never establish
continuity.

Taxonomy, release/node-version, classification-extension, policy, sealed-dataset,
snapshot, observation, result, and metric history is append-only. PostgreSQL must
reject destructive mutation of published/evidence rows. Corrections append a new
evidence version and preserve the original.

## 4. Classification and eligibility time

Classification uses:

```text
HistoricalUniverse.universe_key = classification:<taxonomy_key>
start_date                       = effective_from (inclusive)
end_date                         = effective_to (exclusive or null)
available_at                     = earliest justified knowledge time
retrieved_at                     = actual receipt time
evidence_key                     = one classification episode/segment
membership_version              = increasing correction version
```

Primary-security eligibility is separate evidence under
`peer_primary_common_equity_v1`. Classification never proves eligibility, and
eligibility never proves classification.

For effective date `D`, research timestamp `T`, and an exact sealed dataset:

1. restrict to evidence rows included in the pinned dataset;
2. restrict to the requested source/universe and `available_at <= T`;
3. group by `(source, universe_key, security_id, evidence_key)`;
4. select the greatest valid `membership_version` in each group;
5. only then apply `start_date <= D < end_date`;
6. reject conflicting heads, backwards knowledge, ambiguous branches, overlaps,
   or multiple active classifications;
7. join the exact classification extension and release-specific hierarchy.

The order is mandatory: select the latest evidence version known at `T` before
testing its effective interval. A June 1 announcement effective June 15 is known
but not active on June 10 and is active on June 15. A correction available July 10
cannot alter a query whose knowledge cutoff precedes July 10. A sealed earlier
dataset cannot acquire a later correction or backfill.

Taxonomy releases use the same effective/knowledge distinction. A future release
may be known before its effective date; membership under the active old release
must traverse only the old release hierarchy.

## 5. Peer policies and resolver

Canonical policy: `classification_sub_industry_v1`.

```text
kind: CLASSIFICATION
taxonomy level: SUB_INDUSTRY
eligibility universe: peer_primary_common_equity_v1
minimum non-target membership peers: 5
minimum non-target valid observations: 5
minimum valid-feature coverage: 0.60
one explicitly designated eligible security per company: required
hierarchical fallback: disabled
ordering: permanent security_id ascending
statistics version: peer_stats_v1
```

The generic resolver may also support explicitly requested, separately versioned
`classification_industry_v1`, `classification_industry_group_v1`, and
`classification_sector_v1`. They are different policies and may not publish the
canonical sub-industry feature IDs.

Conceptual API:

```text
PeerResolver.resolve(
    security_id,
    research_timestamp,
    research_build_manifest,
    peer_policy_key,
    peer_policy_version,
) -> PeerResolution
```

The resolver must:

1. require an aware UTC-compatible `T` and hash-verified manifest;
2. require exact provider/build admission across classification, universe,
   identity, and requested feature lookbacks;
3. derive effective session date `D` from the pinned calendar/session policy;
4. resolve target identity, eligibility, active taxonomy release, classification,
   leaf, and selected ancestor using only pinned PIT evidence;
5. resolve all exact-node classification candidates in a bounded set-based query;
6. intersect candidates with PIT primary-equity and lifecycle evidence;
7. enforce one explicitly evidenced primary security per company;
8. retain every rejected candidate with a stable reason;
9. sort by permanent `security_id` and exclude the target from peer observations;
10. return `MISSING/INSUFFICIENT_PEERS` below five non-target members, without
    automatic taxonomy fallback;
11. fingerprint the build, policy, T/D, release/path, candidates, decisions, and
    evidence IDs so exact reruns reuse the immutable snapshot.

A security that later delists remains eligible for an earlier date when the
pinned evidence says it was eligible then. Current-survivor flags never select a
historical peer set.

Membership exclusion codes include:

```text
NO_PIT_MEMBERSHIP
CLASSIFICATION_UNKNOWN
CLASSIFICATION_CONFLICT
TAXONOMY_RELEASE_MISMATCH
SECURITY_NOT_ELIGIBLE
SECURITY_IDENTITY_UNKNOWN
MULTIPLE_ELIGIBLE_SECURITIES
PROVIDER_NOT_QUALIFIED
DATASET_MISMATCH
```

## 6. Membership versus feature eligibility

Peer membership is target-independent group evidence. Feature eligibility is a
separate per-metric decision. A peer remains in the membership and coverage
denominator when its source feature is missing, invalid, stale, unqualified, or
semantically incompatible.

An observation contributes only when:

- exact source feature ID and manifest-pinned calculation/build version match;
- the newest applicable source artifact is legally visible at `T`;
- status is `VALID` (v1 does not silently admit `ESTIMATED`);
- family-specific freshness rules pass;
- unit, frequency, fiscal shape, accounting basis, provider methodology,
  currency, share basis, and scope satisfy the policy; and
- provider admission supports the requested claim.

Feature exclusion codes include:

```text
FEATURE_NOT_AVAILABLE_AT_T
FEATURE_MISSING
FEATURE_INVALID
FEATURE_ESTIMATED_NOT_ALLOWED
FEATURE_STALE
FEATURE_VERSION_MISMATCH
FEATURE_PERIOD_INCOMPARABLE
ACCOUNTING_BASIS_UNKNOWN
ACCOUNTING_BASIS_MISMATCH
CURRENCY_MISMATCH
UNIT_MISMATCH
SHARE_BASIS_MISMATCH
PROVIDER_NOT_QUALIFIED
DATASET_MISMATCH
```

Track independently:

```text
membership peer count
valid feature observation count
excluded feature observation count
coverage = valid feature observations / membership peers
```

A valid statistic requires a valid target, at least five membership peers, at
least five valid non-target observations, and coverage of at least 0.60.

## 7. PIT source-feature selection and staleness

All selected inputs must satisfy `available_at <= T`, exact manifest build/version
identity, and newest-applicable-row semantics. A missing/invalid/stale newest row
does not permit fallback to an older favorable row.

- **Quarterly fundamentals:** choose the greatest measured `period_end <= D` in
  the exact build, then the greatest `available_at <= T` for that period. Require
  consolidated standalone quarterly scope and policy accounting compatibility
  `SEC_US_GAAP_CONSOLIDATED_QUARTERLY`. Age is
  `T - FeatureValue.available_at`; exactly 135 days is valid, anything greater is
  `FEATURE_STALE`. Record availability and period-end ages.
- **Estimates:** require an exact snapshot for `T` under the Phase 2.5 pinned
  profile/dataset/build. Never carry an earlier snapshot forward.
- **Market:** require the exact Phase 2.4 market snapshot/session cutoff for `T`
  and preserve its frozen maximum-five-trading-session price freshness rule.
- **Classification/universe/identity:** no arbitrary age expiry while the
  evidenced effective interval is active, but the provider/dataset must assert
  coverage through `T`.

The source registry units must match exactly. V1 inputs are dimensionless ratios,
percentage-point changes, breadth, or log return; Phase 2.6 performs no currency
conversion, split repair, or underlying formula recalculation.

## 8. Canonical statistics

Use exact `Decimal` arithmetic under the repository's 38-digit
`ROUND_HALF_EVEN` policy. Never use float, silent clipping, or winsorization.

For target value `x` and `N` valid non-target peer values:

```text
peer_median = median(valid non-target peer values)
peer_median_difference = x - peer_median
L = count(peer value < x)
E = count(peer value == x)
peer_percentile = (L + 0.5 * E) / N
```

The percentile is ascending and descriptive. It is never inverted according to
whether a high raw value is economically attractive. Exact canonical Decimal
equality defines ties. For target 20 and peer values `[20, 20, 20, 40]`, the
percentile is `0.375`. If every peer equals the target, it is `0.5`.

Status precedence:

- target `MISSING` → result `MISSING` with source reason;
- target `INVALID` → result `INVALID` with source reason;
- fewer than five membership peers → `MISSING/INSUFFICIENT_PEERS`;
- fewer than five valid peer values →
  `MISSING/INSUFFICIENT_VALID_OBSERVATIONS`;
- coverage below 0.60 → `MISSING/INSUFFICIENT_FEATURE_COVERAGE`;
- evidence/build/taxonomy integrity conflict → `INVALID`.

Ordinary z-score, robust z-score, mean rank, desirability transforms, and
winsorization are not Phase 2.6 v1 statistics.

## 9. Exact peer feature registry

Registry version: `peer_v1`. Entries are numeric, inactive for legacy runners,
snapshot-frequency, unit `ratio`, ascending raw-value order, and
`investment_direction=NONE`. All use `classification_sub_industry_v1` and
`peer_stats_v1`.

| Canonical output feature | Frozen source feature |
| --- | --- |
| `peer_revenue_yoy_growth_percentile` | `revenue_yoy_growth` |
| `peer_revenue_growth_acceleration_percentile` | `revenue_growth_acceleration` |
| `peer_gross_margin_percentile` | `gross_margin` |
| `peer_operating_margin_percentile` | `operating_margin` |
| `peer_operating_margin_yoy_change_percentile` | `operating_margin_yoy_change` |
| `peer_free_cash_flow_margin_percentile` | `free_cash_flow_margin` |
| `peer_eps_adjusted_diluted_consensus_change_scaled_30d_fq1_percentile` | `eps_adjusted_diluted_consensus_change_scaled_30d_fq1` |
| `peer_eps_adjusted_diluted_revision_breadth_30d_fq1_percentile` | `eps_adjusted_diluted_revision_breadth_30d_fq1` |
| `peer_revenue_reported_consensus_change_pct_30d_fq1_percentile` | `revenue_reported_consensus_change_pct_30d_fq1` |
| `peer_return_63d_percentile` | `return_63d` |

There are exactly ten canonical Phase 2.6 `FeatureValue` outputs. Peer median and
target-minus-median live on the authoritative relative result; they are not extra
feature families.

## 10. Exact industry snapshot registry

An industry snapshot is an on-demand formal-node result, not a fabricated security
or generic `FeatureValue`. It includes all valid members rather than excluding one
target. Each metric requires five valid observations and 0.60 coverage.

| Metric key | Definition |
| --- | --- |
| `median_revenue_yoy_growth` | Median of valid latest quarterly `revenue_yoy_growth`. |
| `median_revenue_growth_acceleration` | Median of valid `revenue_growth_acceleration`. |
| `median_operating_margin_yoy_change` | Median of valid `operating_margin_yoy_change`. |
| `median_eps_revision_scaled_30d_fq1` | Median of exact-T `eps_adjusted_diluted_consensus_change_scaled_30d_fq1`. |
| `median_return_63d` | Median of exact-T `return_63d`; not an investable basket return. |
| `breadth_positive_revenue_acceleration` | Valid acceleration values `> 0` divided by valid observations. |
| `breadth_positive_eps_revision_30d_fq1` | Valid scaled EPS revisions `> 0` divided by valid observations. |
| `breadth_above_sma200` | Valid `price_vs_sma200 > 0` divided by valid observations. |

Missing values are not false. Breadth uses valid observations as its denominator;
the separate membership coverage exposes selection bias.

## 11. Snapshot and provenance persistence

Add repository-compatible relational forms of:

- `peer_group_snapshots` and `peer_group_members`;
- `peer_statistic_snapshots` and `peer_statistic_observations`;
- `peer_relative_results`;
- `industry_snapshot_metrics`.

One published result must reconstruct:

```text
target security/company and historical identity
T, effective D, calendar/session policy, and snapshot origin
manifest build ID and canonical payload
peer policy key/version/fingerprint and peer_stats_v1 formula
taxonomy family, exact release/evidence version, node, and full path
classification and eligibility evidence for target and every candidate
ordered candidates, included members, exclusions, and reason codes
source feature registry/build/version and every selected FeatureValue
period, available_at, age, unit/basis, status, and feature eligibility
L, E, N, median inputs, median, delta, percentile, counts, and coverage
provider admissions, limitations, override, weakest quality, and rights gate
calculation version, immutable fingerprint, and creation time
```

Use relational evidence instead of a giant opaque JSON document. Compact JSON may
repeat formulas and admission payloads but every evidence ID must resolve within
the pinned build.

Publication is atomic. Exact reruns reuse IDs, fingerprints, and timestamps.
Different content at the same identity raises
`PUBLISHED_PEER_SNAPSHOT_CONFLICT_USE_NEW_BUILD_OR_POLICY`; no published row is
updated or deleted. A projected canonical `FeatureValue` references the
authoritative `peer_relative_result` in provenance and uses a build-qualified
calculation identity.

## 12. Quality and survivorship

Quality ordering is:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

The result takes the weakest required and included taxonomy, classification,
universe, identity, source-feature, raw-evidence, and calculation admission.
Phase 2.6 never upgrades evidence quality. Synthetic evidence or any exercised
development override forces `DEVELOPMENT` and persists all limitations.

An excluded peer feature does not become a numeric input, but its reason and
coverage effect remain. A survivorship-safe request requires complete historical
universe/security, action, delisting, and lifecycle capability over the entire
lookback. Without it, fail closed with
`BACKTEST_DATA_NOT_SURVIVORSHIP_QUALIFIED`; an explicit named development override
may run only at `DEVELOPMENT` quality.

## 13. Scale and query contract

- Resolve dataset-pinned classification and eligibility heads set-wise, using
  PostgreSQL windowing or `DISTINCT ON`, not one query per security.
- Fetch one source feature for all group members in one bounded query. Fundamental
  selection may use a per-security window; estimate and market values require
  exact snapshots.
- Compute a group distribution once and derive requested target results without
  per-target database round trips.
- Persist only requested on-demand immutable snapshots; never build a daily
  security × feature × node Cartesian table.
- Index release/availability lookups, dataset membership, effective classification
  selection, feature cross-sections, group identity, observations, and results.
- Add query-count and representative PostgreSQL plan sanity coverage before final
  audit. Further infrastructure or partitioning requires measured need.

## 14. Synthetic acceptance contract

The PostgreSQL acceptance fixture contains a target plus at least six peers,
fictional two-release GICS-style hierarchy, a future-effective entrant, an exit,
a later-delisted historical peer, a future announcement, a PIT correction, a
re-entry where applicable, one stale feature, one missing feature, and dataset and
taxonomy-version changes.

It must prove exact boundaries for:

- evidence known versus membership effective;
- active taxonomy release and historical hierarchy;
- correction knowledge without rewriting earlier snapshots;
- sealed dataset isolation;
- target exclusion and deterministic peer order;
- five-peer and 60% thresholds with no fallback;
- membership retention despite metric missingness/staleness;
- exact 135-day freshness boundary;
- negative, zero, tie, median, delta, and midrank arithmetic;
- unit/basis/build/provider exclusions;
- weakest-input quality and survivorship failure/override;
- complete relational provenance;
- immutable run-twice idempotency and atomic rollback;
- bounded query counts and migration lifecycle.

The canonical June 10 worked case is target `0.30` and non-target peers
`[0.10, 0.20, 0.40, 0.50, 0.30]`: `L=2`, `E=1`, `N=5`, median `0.30`, delta `0`,
and percentile `0.5`.

## 15. Checkpoint sequence

1. **2.6.0:** freeze this contract.
2. **2.6.1:** taxonomy, release/hierarchy, classification extension, sealed
   datasets, peer policies, migration, and temporal/immutability tests.
3. **2.6.2:** deterministic PIT peer resolver and candidate provenance.
4. **2.6.3:** feature eligibility, staleness, statistics, exclusions, and quality.
5. **2.6.4:** exact ten-feature registry and isolated `PeerFeatureRunner`.
6. **2.6.5:** exact eight industry snapshot metrics.
7. **2.6.6:** full synthetic PostgreSQL lifecycle/integration scenario.
8. **2.6.7:** independent-minded implementation audit, defect regressions,
   migration/scale checks, and review handoff.

Each checkpoint requires focused tests, appropriate regression tests, an updated
`PROJECT_STATE.md`, commit, push, `HEAD == origin/master`, and a clean worktree.
Phase 2.7 may not begin during this work.
