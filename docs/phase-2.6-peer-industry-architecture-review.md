# Phase 2.6 peer and industry architecture review

Review date: 2026-09-22. Approved starting checkpoint:
`bdc5b8b6a69a356a7a723c378cddd734cc6a6fcb` on `master`, equal to
`origin/master` with a clean worktree before this documentation review. Baseline:
**431 passed, 8 existing warnings, zero failures**; Alembic head `fnd006`.

This is the final implementation specification for Phase 2.6. It does not add
application code, models, migrations, taxonomy data, providers, factors, signals,
rankings, or backtests. Foundation and Phases 2.2, 2.4, and 2.5 retain their frozen
semantics.

## A. Verdict

**CONDITIONAL PASS**

The architecture is implementable and safe to begin. The conditions apply to data
admission and production claims, not to the design:

- Official historical GICS structure and company assignments require a licensed,
  qualified source with effective time, knowledge time, corrections, delisted
  coverage, stable identity, and permitted derived use.
- No live provider currently qualifies a production or survivorship-safe peer
  cross-section. Synthetic fixtures may exercise the complete design, and an
  explicit development override may produce only `DEVELOPMENT` results.
- A Phase 2.6 implementation must preserve the existing fail-closed provider gate,
  immutable build/publication behavior, and frozen Phase 2.4 industry basket. It
  must not make current `Company.sector`, `Company.industry`, or display strings a
  historical identity.

Subject to those gates, the recommendation in section V is **SAFE TO IMPLEMENT
PHASE 2.6**.

## B. Phase 2.6 scope

### Terms

| Term | Exact meaning in this system |
| --- | --- |
| **Sector** | Level 1 of a named formal taxonomy release. It is the broadest GICS-style company classification and is identified by an internal node ID, never by its display name. |
| **Industry group** | Level 2 of the same formal taxonomy release, with exactly one sector parent in v1. |
| **Industry** | Level 3 of the same formal taxonomy release, with exactly one industry-group parent in v1. “Industry” by itself means this formal level, not any arbitrary peer set. |
| **Sub-industry** | Level 4 and the canonical leaf in the v1 GICS-style hierarchy. V1 security classifications point to a leaf and derive ancestors through the pinned release. |
| **Peer group** | The deterministic set selected for one target, timestamp, build, taxonomy release, and versioned peer policy. It may be based on a formal taxonomy now and a different evidenced policy later. A peer group is not synonymous with an industry node. |
| **Comparable company** | An issuer represented by one eligible security that is a member of the peer group **and** has a valid, fresh, semantically compatible observation for the particular statistic. Comparability is metric-specific; it is not permanent membership metadata. |
| **Historical universe** | The set of securities eligible under a named universe definition on an effective date, using only evidence versions known by the research cutoff. It includes failed, acquired, merged, bankrupt, and delisted names when the pinned provider/build covers them. It is distinct from classification. |

A classification relationship says where an issuer sits in a formal taxonomy. A
peer policy says which classifications and eligibility rules define a comparison.
A comparable observation says whether one peer can enter one statistic. These three
concepts must remain separate.

### In scope

- Provider-neutral taxonomy, release, node, hierarchy, and lineage identities.
- Append-only, bitemporal security classification evidence that reuses the frozen
  `HistoricalUniverse` effective/knowledge-time envelope.
- Sealed dataset membership for historical-universe and classification evidence.
- Immutable, versioned peer policies and a deterministic `PeerResolver`.
- Classification peers at sector, industry-group, industry, and sub-industry
  levels; same-sub-industry is the canonical v1 policy.
- Explicit membership eligibility, feature eligibility, exclusions, minimum-count,
  coverage, staleness, fiscal-period, unit, basis, and quality rules.
- Descriptive peer median, target-minus-peer-median, and ascending percentile.
- A focused registry of ten peer-relative percentile features.
- On-demand, reproducible peer and industry snapshots with relational provenance
  and optional immutable `FeatureValue` projection for the ten canonical outputs.
- Focused industry medians and breadth measures.
- Synthetic end-to-end acceptance coverage, including corrections, taxonomy
  changes, delisting, future-effective announcements, stale data, and missing data.

### Explicitly out of scope

- Factors, desirability transforms, composite scores, signals, stock ranking,
  portfolio construction, backtesting, calibration, or investment recommendations.
- AI/LLM-generated peers, inferred competitors, or semantic similarity as an
  authoritative v1 membership source.
- Fundamental, competitive, valuation, or business-quality peer selection in v1.
- Valuation calculations, market-cap/enterprise-value bridges, FX conversion,
  market-share claims, TAM estimates, news/event scoring, or causal attribution.
- Changes to frozen fundamental, market, estimate, universe, provider-admission,
  or `FeatureValue` semantics. In particular, Phase 2.6 does not replace or
  reinterpret `rel_ret_63_ind`.
- Daily materialization of every security × feature × group × date combination.
- Qualification of an external taxonomy, universe, security-master, market, or
  estimates provider merely because synthetic tests pass.

## C. Canonical taxonomy

### Decision

The canonical v1 **taxonomy family is official GICS, using its four formal levels
and an exact provider release**. The schema and algorithms are taxonomy-neutral;
no code path may assume particular numeric codes, names, node counts, or a current
release. A result identifies both `taxonomy_key=GICS` and the exact immutable
taxonomy release/evidence version.

GICS is the best v1 fit because it is company-oriented, global, already matches the
project's sector/industry investment vocabulary, and uses the required sector →
industry group → industry → sub-industry hierarchy. MSCI describes one company
classification at each of those four levels, based principally on business
activity and revenue, with earnings and market perception also considered
([MSCI GICS overview](https://www.msci.com/indexes/index-resources/gics)).

This choice is conditional on rights. GICS is the proprietary standard of S&P
Global and MSCI, and S&P's current methodology/disclaimer states that use and
distribution can require licensing
([S&P GICS methodology](https://www.spglobal.com/spdji/en/documents/methodologies/methodology-gics.pdf)).
S&P offers current and historical company classifications as a dataset
([S&P GICS dataset](https://www.marketplace.spglobal.com/en/datasets/gics-%2890%29)).
The project must verify the actual contract for storage, historical retention,
derived peer statistics, backups, AI processing, internal display, and any external
redistribution. This review is not legal advice and does not infer those rights.

Until a qualified licensed profile exists:

- production GICS membership is unavailable;
- tests use a clearly named `SYNTHETIC_GICS_STYLE_V1` taxonomy;
- repository fixtures use fictional codes, definitions, and assignments; and
- outputs remain `DEVELOPMENT` and cannot claim official GICS classification.

The existing Phase 2.4 GICS-sector-name-to-ETF dictionary remains frozen legacy
market logic. It is neither an official taxonomy dataset nor evidence of historical
classification rights. Phase 2.6 does not route Phase 2.4 through the new schema.

### Alternatives assessed

| Candidate | Appropriate use | Why it is not canonical v1 |
| --- | --- | --- |
| GICS | Investment-oriented global company comparison at four useful levels | Chosen, but exact structure/assignments and historical use require licensed qualification. |
| NAICS | Public economic statistics and future industry-denominator joins | NAICS classifies establishments by production process and has sector, subsector, industry group, industry, and national-industry levels; a multi-business public issuer does not map cleanly to one establishment code. It is North-America-focused rather than a global listed-equity peer standard ([U.S. Census NAICS structure](https://www.census.gov/programs-surveys/economic-census/year/2022/guidance/understanding-naics.html)). |
| SIC | SEC metadata, legacy crosswalks, and source diagnostics | It is older/coarser, current SEC SIC is not a historical peer record, and SEC uses it partly to assign filing-review responsibility ([SEC SIC list](https://www.sec.gov/search-filings/standard-industrial-classification-sic-code-list)). `Company.sic` is current metadata, not PIT evidence. |
| Provider taxonomy | Adapter input or a separately named taxonomy family | Vendor labels, revisions, history, coverage, and rights differ. They cannot be relabeled GICS or silently pooled. |
| Custom economic taxonomy | Future cross-industry economic, product, customer, or competitive group | It needs its own versioned definitions and evidence. It must not be an undocumented override of formal classification peers. |

### Taxonomy versioning rules

- `industry_taxonomies` identifies the family; `industry_taxonomy_releases`
  identifies one immutable published/corrected release version.
- A logical release key and monotonically increasing evidence version distinguish
  a later correction from the earlier artifact. Corrections append and supersede;
  no release, node, label, definition, or edge is updated in place.
- Stable internal `industry_node_id` identifies a continuing concept. Release-
  specific code, name, definition, and parent live in `industry_node_versions`.
- A split, merge, or genuine concept replacement creates new node IDs plus explicit
  lineage edges. Resolver code never assumes that similar names mean continuity.
- Each v1 release is a strict acyclic four-level tree. Every non-sector node has
  exactly one parent at the immediately broader level, and every classified
  security points to a sub-industry node present in that exact release.
- Cross-taxonomy mappings are optional, versioned evidence. They are never used
  implicitly by the v1 resolver.

## D. Historical membership model

### Reuse of the frozen foundation

`HistoricalUniverse` remains the temporal evidence envelope. Phase 2.6 must not
invent a second implementation of effective time, knowledge time, receipt time,
append-only correction versions, or stable `security_id` selection.

Classification evidence uses a `HistoricalUniverse` row with:

```text
universe_key      = classification:<taxonomy_key>
security_id       = permanent internal security identity
evidence_key      = one classification segment/episode
membership_version= monotonically increasing version of that segment
start_date        = effective_from, inclusive
end_date          = effective_to, exclusive or null
available_at      = earliest justified knowledge of this evidence version
retrieved_at      = actual receipt
source            = qualified provider/profile identity
timing_policy     = AS_OBSERVED | CERTIFIED_PUBLICATION | SYNTHETIC
source_raw_object_id = immutable raw evidence, except synthetic fixtures
```

An additive one-to-one `security_industry_classifications` extension links that
row to the exact taxonomy release and leaf node. It contains no competing temporal
columns. Existing `sector` and `industry` strings remain an optional legacy
projection for Phase 2.4 and are never Phase 2.6 identity.

An eligibility universe is separate evidence, for example
`universe_key=peer_primary_common_equity_v1`. Classification does not prove that a
security is an eligible primary common equity, and eligibility does not prove a
classification.

### Exact temporal semantics

For classification of security `S` on effective date `D`, known at timestamp `T`:

1. Restrict to evidence rows explicitly included in the pinned, sealed universe/
   classification dataset build.
2. Restrict to `available_at <= T` and the requested `universe_key` and source.
3. Group by `(source, universe_key, security_id, evidence_key)`.
4. Select the greatest `membership_version` in each group. Reject non-monotone
   knowledge time, conflicting heads, or ambiguous branches.
5. Only then filter `start_date <= D < end_date`, with null `end_date` open-ended.
6. Require at most one active classification segment for `S` in that taxonomy.
7. Join the selected extension row and exact release-specific node path. Missing,
   cross-release, or structurally invalid paths fail closed.

The order is mandatory: latest known version first, effective interval second,
classification filter last. It preserves the foundation rule that a later removal
or correction cannot resurrect an older segment.

For a June 1 announcement effective June 15, publication appends atomically:

- a new version of the old segment with `effective_to=June 15`, and
- a new segment with `effective_from=June 15` and the new leaf node.

At June 10 the change is known but the old classification is active. At June 16
the new classification is active. A future-effective segment must never be treated
as current merely because it is known.

### Corrections and reconstruction

A correction retains the affected `evidence_key`, increments
`membership_version`, has its own later `available_at` and raw evidence, and points
to the corrected leaf/release. Before that availability, the prior version remains
visible. After it, a query whose knowledge cutoff includes the correction may use
the corrected effective history. The original row remains queryable.

Dataset membership is as important as timestamp filtering. A later backfill or
newly certified historical artifact is included only in a new sealed dataset and
new `ResearchBuildManifest`. It cannot alter an existing peer snapshot. Certified
historical reconstruction may establish what the market could have known; it does
not impersonate what this installation actually received then. Snapshot provenance
records `HISTORICAL_REPLAY`, `LIVE`, or `EDUCATIONAL` origin separately.

Taxonomy structure follows the same distinction. A release may be announced and
known before its effective date. Membership active under an old release does not
traverse the future release's hierarchy. A corrected historical taxonomy release
is a new evidence version and a new build input.

## E. Peer types

| Peer type | V1 status | Definition |
| --- | --- | --- |
| Classification peer | **Implemented in Phase 2.6 v1 design** | Same node at a requested formal level under the exact taxonomy release at `D`, known at `T`, after historical-universe eligibility. |
| Fundamental peer | Future | Selected by versioned economic attributes such as scale, growth, margin structure, business model, and capital intensity. Metric selection cannot use future data or the outcome being evaluated without a declared policy. |
| Competitive peer | Future | Evidence that firms compete for the same product, customer, budget, or market. Formal taxonomy may be insufficient. |
| Valuation peer | Future | Companies for which a named multiple is comparable after enterprise-value, accounting, currency, growth, and profitability rules. |
| Business-quality/economic peer | Future | Evidence-backed product, end-market, moat, customer, or business-model similarity. AI may propose candidates but cannot make authoritative membership. |

Phase 2.6 v1 implements only deterministic classification peers. It deliberately
does not add metric-nearest-neighbor, clustering, hand-curated competitive, or
AI-generated peer authority. The policy and provenance interfaces leave room for
those future kinds without changing formal taxonomy history.

## F. V1 peer resolver

### Interface

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

The manifest, rather than a giant concatenated identifier, pins the taxonomy and
classification dataset, historical eligibility universe, security-master build,
provider/qualification versions, calendar policy, and relevant feature builds.
The manifest's existing SHA-256 `build_id` is the build identity.

`PeerResolution` returns:

```text
target security and company
research timestamp and effective session date
policy key/version/fingerprint
taxonomy family, release, leaf, and selected-level node path
peer-group definition identity and snapshot fingerprint
candidate securities in permanent-security-ID order
included peer security IDs (target excluded from this list)
membership and eligibility evidence IDs for every candidate
membership exclusions and stable reason codes
membership peer count and total group size
provider admission, limitations, and quality level
status/reason and complete provenance
```

### Canonical policy

`classification_sub_industry_v1` is the canonical feature policy:

```text
peer kind: CLASSIFICATION
taxonomy family: GICS (synthetic GICS-style in tests)
level: SUB_INDUSTRY
historical eligibility universe: peer_primary_common_equity_v1
minimum non-target membership peers: 5
minimum non-target valid feature observations: 5
minimum valid-feature coverage: 60% of non-target membership peers
one eligible security per company: required
hierarchical fallback: disabled
ordering: permanent security_id ascending
statistics: peer_stats_v1
```

The generic resolver also supports explicitly requested
`classification_industry_v1`, `classification_industry_group_v1`, and
`classification_sector_v1`. Those are distinct policies and peer-group identities.
They do not publish the canonical sub-industry feature IDs.

### Selection algorithm

1. Validate an aware UTC research timestamp and load the hash-verified manifest.
2. Admit every required source over the complete classification, universe, identity,
   and feature lookback. No source or build default is allowed.
3. Map `T` to effective date `D` using the pinned market-calendar/session policy.
4. Resolve the target's effective and known security identity and explicit primary-
   equity eligibility; never consult today's `Security.is_active`.
5. Resolve the target's taxonomy leaf and ancestor at the policy level from the
   pinned classification dataset.
6. Resolve all classification members of that exact node at `(D,T)` in one batch.
7. Intersect them with the pinned eligibility universe at `(D,T)` and the qualified
   security-master/lifecycle evidence. Retain every rejected candidate and reason.
8. Enforce one security per `company_id`. The eligibility dataset must name the
   primary security. Ambiguity is excluded as `MULTIPLE_ELIGIBLE_SECURITIES`; a
   ticker or lowest database ID is never an economic tiebreak.
9. Keep securities that later delisted if they were eligible at `D`. Exclude a
   security not yet effective, already outside the universe, or without evidenced
   historical identity at `D`.
10. Sort included peers by permanent `security_id`, exclude the target from the
    returned peer list, and count them.
11. If fewer than five peers remain, return `MISSING/INSUFFICIENT_PEERS`; do not
    broaden the taxonomy.
12. Hash canonical JSON containing the manifest build, policy, T/D, release/node,
    ordered candidate decisions, and evidence IDs. Exact reruns reuse the snapshot;
    conflicting content requires a new build or policy version.

The peer-group definition identity is stable for the policy/release/node. The
peer-group **snapshot** identity additionally includes T, build, resolved membership,
and evidence, because membership can change through time.

## G. Peer eligibility

Membership eligibility and feature eligibility are separate phases.

### Membership eligibility

A non-target security is a peer member only when all are true:

- it has an active classification in the exact taxonomy/node at `(D,T)`;
- it is active in the named historical eligibility universe at `(D,T)`;
- its permanent identity/lifecycle evidence is present in the pinned build;
- it is the explicitly designated primary eligible common equity for its company;
- its taxonomy, universe, and security providers/builds match the manifest; and
- no classification, version, overlap, branch, or dataset conflict exists.

Current ticker, current company active flag, current classification strings, feature
availability, price availability, or future survival are not membership criteria.
ADRs, preferred shares, funds, ETFs, warrants, and duplicate share classes are
excluded unless a future peer policy explicitly admits a historically evidenced
type. V1 does not infer security type from a ticker suffix.

### Feature eligibility

For one source feature, an already selected member contributes only when:

- the exact feature ID and manifest-pinned calculation/build version match;
- its selected value was available by `T` under section K;
- status is `VALID` (v1 does not silently admit `ESTIMATED`);
- it is within the family-specific staleness boundary;
- frequency, fiscal shape, unit, accounting/basis, provider methodology, currency,
  and share basis satisfy the feature policy; and
- its required provider admission is no stronger than the output claims.

A missing, stale, or incompatible observation does **not** remove the company from
the peer group. It produces a feature-observation exclusion and remains in the
coverage denominator. A statistic requires a valid target, at least five valid
non-target peers, and at least 60% valid coverage of all membership peers.

Stable membership exclusion reasons include:

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

Feature-observation reasons include:

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

These are machine-readable codes; explanations and underlying source reasons are
also preserved.

## H. Fallback policy

**V1 has no automatic hierarchical fallback.**

If the same-sub-industry policy has too few eligible peers, it returns
`MISSING/INSUFFICIENT_PEERS`. Automatically moving to industry or industry group
would change economic meaning precisely where group quality is weakest, and a user
could not compare the same feature consistently across companies.

A caller may make a second, explicit request using `classification_industry_v1` or
another named policy. That result has a different policy, group identity, and
provenance and cannot be stored under the canonical sub-industry feature ID. A
future validated fallback chain would require its own versioned policy and must
record the actual level used; it is not part of v1.

## I. Peer statistics

### Canonical v1 set

For each eligible target/source-feature pair, v1 computes exactly:

1. peer median, excluding the target;
2. target minus peer median, in the source feature's unit; and
3. ascending midrank percentile against non-target peers.

The snapshot also returns membership-peer count, valid-peer count, excluded-peer
count, feature coverage ratio, target age, peer ages, minimum/maximum peer values,
and all exclusions. Mean, ordinal rank, ordinary z-score, winsorization, and
MAD-based robust z-score are not canonical v1 outputs.

The median and difference are high-value explanatory fields, but they are not
multiplied into separate `FeatureValue` families. The ten registry features persist
the percentile; their authoritative peer-statistic snapshot contains the median
and difference.

### Median

Sort valid non-target peer values numerically. For odd `N`, select the middle
value. For even `N`, use the arithmetic mean of the two middle values. Calculations
use Decimal with the existing 38-digit, `ROUND_HALF_EVEN` policy. No float, clipping,
or winsorization is allowed.

```text
peer_median_difference = target_value - peer_median
```

### Percentile and ties

Percentile is always ascending in raw numeric value; it is never inverted to mean
“better.” Let:

```text
N = number of valid non-target peer observations
L = number of peer values strictly less than target x
E = number of peer values exactly equal to target x

peer_percentile = (L + 0.5 * E) / N
```

The output is a ratio in `[0,1]`. Exact Decimal equality after canonical unit
normalization defines a tie. Higher raw values have higher percentiles. Negative
values use ordinary numeric order. A valuation or volatility percentile, if added
later, remains descriptive; Phase 2.7 decides investment direction.

Worked revenue-growth example, with C as target:

```text
A 10%
B 20%
C 30%  <- target
D 40%
E 50%
```

C's peer reference is `[10,20,40,50]`: `L=2`, `E=0`, `N=4`, so C is
`0.50`, or the 50th percentile. Peer median is `(20+40)/2=30%`; median
difference is `0` percentage points. This small arithmetic example is below the
v1 publication threshold of five valid non-target peers, so a real v1 snapshot
with only these names returns `INSUFFICIENT_VALID_OBSERVATIONS` rather than false
precision.

Tie example for a target value of 20 against peer values:

```text
20
20
20
40
```

`L=0`, `E=3`, `N=4`; percentile is `(0 + 0.5*3)/4 = 0.375`. The peer median
is 20 and the difference is zero. If every peer equals the target, percentile is
exactly `0.5`.

### Status rules

- Target `MISSING` → result `MISSING` with the source reason.
- Target `INVALID` → result `INVALID`; an older favorable value is not substituted.
- Membership peers `<5` → `MISSING/INSUFFICIENT_PEERS`.
- Valid peer observations `<5` → `MISSING/INSUFFICIENT_VALID_OBSERVATIONS`.
- Valid peer coverage `<0.60` → `MISSING/INSUFFICIENT_FEATURE_COVERAGE`.
- An evidence/build/taxonomy integrity conflict → `INVALID`, not ordinary missingness.

## J. Staleness policy

Staleness is evaluated after membership and latest-PIT selection. Stale rows remain
visible in provenance and never cause a fallback to an older row.

| Family | V1 rule |
| --- | --- |
| Quarterly fundamentals | Select the latest PIT-known quarterly measured period. `T - feature.available_at <= 135 * 24 hours`; exactly 135 days is eligible, anything greater is `FEATURE_STALE`. Record both availability age and `T - period_end`. |
| Estimates | Require an exact snapshot calculated for T under the Phase 2.5 dataset/build and consensus-through-T orchestration. Never carry an earlier estimate `FeatureValue` forward. Existing event-history coverage rules remain authoritative. |
| Market | Require the exact market snapshot/session cutoff for T. Reuse Phase 2.4's frozen maximum-five-trading-session stale-price rule and provenance; Phase 2.6 does not replace it with calendar days. |
| Classification/universe/identity | No arbitrary age expiry while an evidenced effective interval remains active. The qualified dataset must assert coverage through T; a stale/current-only provider fails admission. |
| Future external industry observations | Each metric definition supplies cadence, publication-lag, and max-age policy. No generic carry-forward. |

The 135-day quarterly rule permits ordinary differences in fiscal calendars and
reporting dates but rejects skipped-quarter and nine-month-old comparisons. Every
snapshot exposes age distributions so later factors can distinguish a barely valid
cohort from a fresh one. A threshold change is a new peer-policy version.

## K. PIT retrieval, fiscal alignment, and build pinning

### Cross-sectional retrieval contract

At research timestamp `T`, every selected input must satisfy:

```text
input.available_at <= T
input belongs to the manifest-pinned dataset/build
input calculation/normalization/provider version exactly matches the manifest
the newest applicable row is selected even when it is MISSING or INVALID
```

There is no “latest production version” default and no mixing of builds within a
cross-section.

For quarterly fundamental features, selection is two-stage:

1. among values known by T in the exact build, choose the greatest measured
   `period_end` not after D; and
2. within that period choose the greatest `available_at <= T`.

This prevents a newly published restatement of an old quarter from displacing a
newer measured quarter. Fiscal identity and quarterly shape are validated through
the source provenance. If the latest measured period's newest value is missing,
invalid, stale, or ambiguous, do not fall back to an older period.

V1 deliberately compares each issuer's latest PIT-known comparable quarterly
feature, not identical fiscal labels across companies. `Company A FY2027 Q2` and
`Company B FY2026 Q4` may be compared when both are their latest fresh standalone
quarterly observations. The snapshot records fiscal year/period, period end,
availability, and age for every observation. Exact-quarter cohort analysis can be
a future policy; calendar-month inference is prohibited.

For estimates and market features, the source runner must publish the exact T
snapshot first. An earlier stored row is not evidence that nothing changed.

### Later calculations of historical T

`calculated_at` is operational metadata, not evidence availability. A feature may
be calculated today from qualified historical evidence that was available by T.
It is safe for a **reconstructed** historical result only when:

- the provider admits historical replay for the complete lookback;
- all raw/evidence versions and the calculation code are pinned in a new manifest;
- corrections with `available_at > T` are excluded;
- the snapshot origin is recorded as `HISTORICAL_REPLAY`, not a live prediction;
- the new artifact does not overwrite an older build or snapshot.

An as-seen-today backfill, current-only provider, uncertified historical timestamp,
or recalculation without an exact build is not eligible. A later bug fix or data
backfill creates a new calculation/build identity. Existing sealed results continue
to resolve their original manifest.

### Build contents

Reuse `ResearchBuildManifest`. Its nested fields pin at minimum:

```text
datasets:
  classification dataset/build
  historical eligibility-universe dataset/build
  security identity/lifecycle dataset/build
  fundamental feature build (when requested)
  market feature build (when requested)
  estimate provider dataset/build (when requested)
provider_versions and qualification versions
normalization_versions
feature_versions
calculation_versions:
  source feature versions
  peer_policy_version
  peer_stats_v1
configuration:
  calendar/session cutoff
  taxonomy family/release policy
  staleness and compatibility policy
research_timestamp
```

Peer tables and projected FeatureValues store the 64-character manifest build hash
plus relational dataset/evidence IDs. Do not encode every component into one giant
peer ID.

### Unit, currency, accounting, and share compatibility

V1 intentionally selects dimensionless source features: growth ratios, margin
ratios/changes, revision ratios/breadth, and log return. It does not compare
absolute revenue, FCF, EPS, market cap, or valuation multiples.

- Units must match the registry exactly (`ratio` or `percentage_points` as defined
  by the frozen source feature). A percentage displayed as 30 must not mix with a
  stored ratio of 0.30.
- Currency differences do not exclude a genuinely dimensionless ratio, but each
  company's numerator/denominator or old/new estimate must be internally currency-
  consistent. Absolute cross-currency features remain out of scope.
- Fundamental comparisons require the policy's declared accounting compatibility
  class and consolidated quarterly scope. Canonical v1 is
  `SEC_US_GAAP_CONSOLIDATED_QUARTERLY`; unknown or mixed frameworks are excluded,
  not silently treated as equivalent. A future IFRS or mixed-framework policy must
  be separately versioned and validated.
- Estimate features retain Phase 2.5's provider, methodology, reported/adjusted,
  diluted share type, share-basis, scope, currency, unit, and absolute-target
  constraints. Dimensionless cross-company comparison does not repair an invalid
  within-company series.
- The historical eligibility policy prevents duplicate ordinary/ADR/share-class
  representation. No current share structure is projected backward.

## L. Industry aggregates

`IndustrySnapshot` is an on-demand formal-node snapshot, not a security
`FeatureValue`. It uses the same taxonomy release, historical membership,
eligibility, build, PIT retrieval, staleness, compatibility, count, coverage, and
quality contracts as peer statistics. Unlike a target peer comparison, an industry
median includes every valid member.

The focused v1 metric registry is:

| Industry snapshot metric | Definition |
| --- | --- |
| `median_revenue_yoy_growth` | Median of valid latest quarterly `revenue_yoy_growth`. |
| `median_revenue_growth_acceleration` | Median of valid `revenue_growth_acceleration`. |
| `median_operating_margin_yoy_change` | Median of valid `operating_margin_yoy_change`. |
| `median_eps_revision_scaled_30d_fq1` | Median of exact-T `eps_adjusted_diluted_consensus_change_scaled_30d_fq1`. |
| `median_return_63d` | Median of exact-T `return_63d`; descriptive cross-section, not an investable basket return. |
| `breadth_positive_revenue_acceleration` | Count of valid acceleration values `>0` divided by valid count. Zero is not positive. |
| `breadth_positive_eps_revision_30d_fq1` | Count of valid scaled EPS changes `>0` divided by valid count. |
| `breadth_above_sma200` | Count of valid `price_vs_sma200 >0` divided by valid count. |

Every metric returns total members, valid/missing/invalid/stale counts, coverage
ratio, value, status/reason, ages, build, and evidence. A metric is valid only with
at least five observations and at least 60% coverage. Missing values are not false;
breadth denominator is valid observations, while the separate coverage ratio makes
selection bias visible.

Phase 2.4 already supplies frozen stock-versus-industry return via a daily
mean-log-return basket. Phase 2.6 neither duplicates nor relabels that as an
investable portfolio. `median_return_63d` and market breadth answer different
cross-sectional questions.

## M. Peer-relative feature registry

Canonical registry version: `peer_v1`. All entries are explicit, inactive for
legacy fundamental/market scans, numeric, snapshot-frequency, percentile unit
`ratio`, ascending raw-value order, and `investment_direction=NONE`.

| Canonical Phase 2.6 feature ID | Frozen source feature ID |
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

All ten use `classification_sub_industry_v1`, `peer_stats_v1`, and exact source
feature/build metadata in registry logic and provenance. The estimate-derived
entries can exist in the schema while remaining unavailable for production because
the current estimate provider is synthetic-only.

V1 deliberately omits:

- estimate revision-count percentile, because contributor coverage intensity can
  dominate economic meaning;
- absolute EPS change, FCF, revenue, and dollar-volume percentiles because units,
  currency, scale, and share basis need further policies;
- EPS growth percentile because sign-change/one-time-item behavior adds little to
  the focused set beside revenue and estimate evidence;
- market permutations already represented by frozen `rel_ret_63_ind`; and
- z-scores, robust z-scores, desirability scores, and automatically generated
  feature × peer-level combinations.

The peer median and median difference for each registered source metric are direct
fields of `PeerStatisticSnapshot`, not additional feature IDs. Industry aggregates
remain direct `IndustrySnapshot` outputs because `FeatureValue` is security-keyed.

## N. Data-quality propagation

Quality is an ordered lower-bound lattice:

```text
DEVELOPMENT
< PIT_PARTIAL
< PIT_QUALIFIED
< SURVIVORSHIP_QUALIFIED
< PRODUCTION_RESEARCH
```

The result quality is the weakest admission among every **required and included**
taxonomy, classification, universe, security-identity, source-feature, raw-evidence,
and calculation input. Phase 2.6 never averages quality, chooses the strongest
input, or upgrades synthetic/development evidence.

Exact policy:

- Missing provider/build admission is a hard error, not an assumed quality.
- A development override must use the existing named requester/reason/reference
  contract; any admitted limitation forces the result to `DEVELOPMENT` and is
  persisted.
- Synthetic evidence always forces `DEVELOPMENT`.
- An excluded feature observation does not become an input and therefore does not
  lower the numeric result's quality, but its exclusion and coverage impact remain
  explicit. A low-coverage cohort fails rather than presenting a high-quality rank.
- Target missing/invalid status propagates as specified in section I. Peer missing/
  invalid statuses remain exclusions unless they push count/coverage below policy.
- A production label additionally requires verified rights. A survivorship label
  additionally requires complete historical universe/security coverage, PIT
  actions, delistings, lifecycle, and `SURVIVORSHIP_SAFE` admission for the whole
  requested interval.
- Without delisted/failed-company coverage, a historical peer calculation fails a
  survivorship-safe request. It may run only under an explicit development override,
  returns `DEVELOPMENT`, and records `BACKTEST_DATA_NOT_SURVIVORSHIP_QUALIFIED`.

Phase 2.6 should extend, not weaken, provider capabilities with explicit taxonomy
structure and historical-classification coverage declarations. Brand name or a
current GICS code is never a qualification.

## O. Provenance and reconstruction contract

To answer “Why was NVDA at this percentile on this date?”, one immutable result
must reconstruct:

```text
target security/company and historical identity
research T, effective date D, timezone, calendar, and session cutoff
snapshot origin (live, historical replay, educational)
ResearchBuildManifest hash and canonical payload location
peer policy key/version/fingerprint and statistic version/formula
taxonomy family, exact release/evidence version, selected node and full path
target classification and eligibility evidence
all classification candidates in deterministic order
every inclusion/exclusion decision and reason
ordered included peers and membership evidence/raw references
source feature ID, registry metadata, exact calculation/build version
target and every peer FeatureValue ID/value/unit/status/period/available_at/age
feature eligibility decisions and reasons
L, E, N, median inputs, median, delta, percentile, count and coverage
provider admissions, qualification versions, limitations, overrides, quality
calculation code/version, output fingerprint, created_at
```

Relational rows, not a large opaque JSON blob, preserve membership and feature
observations. Compact JSON may repeat canonical formulas and admission payloads for
portable inspection. Every JSON evidence ID must resolve within the selected build.

Publication is atomic and append-only. Exact reruns return the same IDs and
fingerprints. Changed content at the same identity raises
`PUBLISHED_PEER_SNAPSHOT_CONFLICT_USE_NEW_BUILD_OR_POLICY`; no UPDATE/DELETE or
silent overwrite path is permitted for published results.

## P. Database design

No migration is created by this review. The implementation should add the following
PostgreSQL structures using BIGINT internal keys, immutable rows, explicit checks,
and exact NUMERIC values.

### Taxonomy and evidence

| Proposed table | Purpose and essential keys |
| --- | --- |
| `industry_taxonomies` | Stable taxonomy family: unique `taxonomy_key`, owner, description, rights/entitlement reference, active flag for admission only. Display name is not identity. |
| `industry_taxonomy_releases` | Immutable release evidence: taxonomy FK, logical release key, evidence version, effective interval, announced/published/available/retrieved timestamps, source/profile, raw FK, supersedes release, content fingerprint. Unique taxonomy + logical key + version. |
| `industry_nodes` | Stable concept identity scoped to taxonomy: taxonomy FK, immutable internal node key, level. Unique taxonomy + node key. |
| `industry_node_versions` | Release-specific provider code, display name, definition and parent node. Composite unique release + node and release + provider code; same-release composite FK enforces the parent version. |
| `industry_node_lineage` | Explicit CONTINUES/SPLIT_FROM/MERGED_FROM/REPLACED_BY relations between stable nodes/releases; never implicit peer membership. |
| `security_industry_classifications` | One-to-one extension keyed by `historical_universe_id`; exact taxonomy release and leaf node, source code, mapping method/version, content fingerprint. No duplicate effective/knowledge timestamps. |
| `universe_evidence_datasets` | Sealed BUILDING/COMPLETE/FAILED corpus with provider/profile, dataset and normalization version, retrieval watermark, manifest hash and completion time. Used for classification and eligibility builds. |
| `universe_evidence_dataset_rows` | Many-to-many inclusion of immutable `HistoricalUniverse` evidence rows in a dataset. Unique dataset + row; no additions after sealing. |

Taxonomy release corrections, node versions, classification extensions, and
dataset memberships must reject UPDATE/DELETE operationally. `HistoricalUniverse`
retains its frozen database semantics; Phase 2.6 uses its existing append-only
version API and an operational role that cannot delete evidence. Related old-
segment closure and new-segment insertion publish in one transaction.

### Policies and on-demand snapshots

| Proposed table | Purpose and essential keys |
| --- | --- |
| `peer_policies` | Immutable key/version plus canonical JSON for type, taxonomy level, universe, security-type/primary rules, count/coverage, fallback, compatibility, staleness, ordering and stats version; unique policy key + version and fingerprint. |
| `peer_group_snapshots` | One formal group at T/build/policy/release/node: effective date, manifest hash, status/reason, counts, admission/quality and canonical fingerprint. Unique build + T + policy + release + node. |
| `peer_group_members` | All target-independent candidates: snapshot, security/company, classification and eligibility evidence IDs, included flag, reason, deterministic ordinal. Unique snapshot + security. |
| `peer_statistic_snapshots` | Group + exact source feature/build/statistic version, full-group median/coverage/status/fingerprint. Unique group snapshot + source feature + calculation version + stats version. |
| `peer_statistic_observations` | One member observation: FeatureValue FK, value/unit/status, fiscal identity/period, available_at, age, accounting/basis metadata, included flag and exclusion reason. Unique statistic snapshot + security. |
| `peer_relative_results` | Target-specific leave-one-out peer median, delta, L/E/N, percentile, counts, status/reason, quality and fingerprint. Unique statistic snapshot + target security. |
| `industry_snapshot_metrics` | Focused median/breadth outputs keyed to the same group snapshot, metric definition/version, numerator/denominator/counts/coverage/status. No fake industry security. |

The canonical ten percentile results may be projected on demand to existing
`FeatureValue`, with `calculation_version=peer_v1:<manifest_build_id>:<policy_id>`
or another unambiguous build-qualified identity and provenance referencing
`peer_relative_result_id`. The specialized row is authoritative. Exact projection
is immutable under the existing feature publication contract.

### Required indexes

- Taxonomy releases: `(taxonomy_id, effective_from, effective_to, available_at)`
  and unique `(taxonomy_id, logical_release_key, evidence_version)`.
- Node versions: `(release_id, level, parent_node_id)` and unique release/provider
  code.
- Classification extension: `(taxonomy_release_id, leaf_node_id,
  historical_universe_id)`.
- Dataset rows: `(dataset_id, historical_universe_id)` plus reverse row lookup.
- Historical evidence batch selection: composite support for
  `(universe_key, security_id, source, evidence_key, available_at,
  membership_version)` in addition to frozen existing indexes.
- Feature cross-sections: `(feature_id, calculation_version, security_id,
  period_end DESC, available_at DESC)`; benchmark measured query plans before
  finalizing included/partial indexes.
- Group snapshots: `(research_timestamp, taxonomy_release_id, industry_node_id,
  peer_policy_id, build_manifest_hash)` and unique fingerprint.
- Observations/results: `(peer_statistic_snapshot_id, included, security_id)` and
  `(target_security_id, peer_statistic_snapshot_id)`.

No display-name index establishes identity. Foreign keys prevent orphan evidence;
checks enforce aware timestamps, valid intervals, hierarchy levels, status/value
combinations, nonfinite-value rejection, and count consistency.

### Storage decision

Compute peer and industry snapshots on demand, then cache/persist only requested or
published artifacts. Do not precompute every daily permutation. Membership changes,
new FeatureValues, explicit research requests, or backtest timestamps are natural
events. Repeated requests reuse the immutable content fingerprint.

## Q. Synthetic acceptance scenario

### Fixture

Taxonomy `SYNTHETIC_GICS_STYLE_V1` contains:

```text
R1: Technology
    └── Semiconductor & Equipment
        └── Semiconductors
            ├── Semiconductor Infrastructure
            └── Semiconductor Devices

R2: Technology
    └── Semiconductor & Equipment
        ├── Semiconductors
        │   └── Semiconductor Devices
        └── Semiconductor Infrastructure & Networking
            └── Semiconductor Infrastructure  (same stable leaf, new parent)
```

R1 is known 2025-12-01 and effective 2026-01-01. R2 is announced/available
2026-06-01 and effective 2026-06-15; it reparents the stable Semiconductor
Infrastructure leaf under the new industry. Queries before June 15 use the R1
path even though R2 is known. Queries from June 15 use R2.

Fictional companies/security IDs A–F are primary common equities:

- A is the target and continuously belongs to Semiconductor Infrastructure.
- B is continuous, then has a change to Semiconductor Devices announced August 1
  and effective September 1 (future membership announcement).
- C leaves for Semiconductor Devices effective June 15, announced June 1.
- D later delists effective July 1, announced June 25. Its permanent security ID
  and earlier membership remain queryable after delisting.
- E is initially recorded in Infrastructure. A provider correction available July
  10 says its correct Devices membership was effective May 1. Both evidence versions
  remain stored.
- F is a new Infrastructure entrant announced May 20 and effective June 1.

At June 10, revenue-growth values and fresh availability are:

```text
A 30%  target
B 10%
C 20%
D 40%
E 50%
F 30%
```

A has five non-target membership peers. Against `[10,20,40,50,30]`, A has
`L=2`, `E=1`, `N=5`, percentile `0.5`, peer median `30%`, and delta `0`.

For gross margin on the same date, D's latest result is 135 days plus one
microsecond old and therefore `FEATURE_STALE`; E has `FEATURE_MISSING`. Membership
still contains B–F, but only three non-target observations remain valid, so the
statistic is `MISSING/INSUFFICIENT_VALID_OBSERVATIONS`. Both exclusions appear in
the statistic snapshot; neither company disappears from the peer group.

### Required timestamp matrix

| Query | Expected classification/membership result | Expected statistic result |
| --- | --- | --- |
| May 19 | F announcement not known; A peers B–E (4) | `INSUFFICIENT_PEERS` |
| May 25 | F change known but not effective | Same four peers; future membership excluded |
| June 1 exact effective boundary | F enters; A peers B–F (5) | Revenue percentile valid at 0.5 once exact-T features are published |
| June 10 | R2 and C exit known but not effective; R1 hierarchy active | A peers B–F; revenue result above remains valid |
| June 15 exact boundary | R2 hierarchy becomes active and C leaves | A peers B,D,E,F (4); `INSUFFICIENT_PEERS`; provenance names R2 and actual node path, with no fallback |
| June 30 | D is still active because delisting is future-effective | D remains included despite today's eventual `Security.is_active=False` |
| July 1 exact boundary | D is no longer eligible | D excluded with lifecycle/universe evidence; earlier snapshots unchanged |
| July 9 | E correction unavailable | E's old known classification remains visible |
| July 10 exact correction boundary | E correction becomes known and is effective from May 1 | A's current peer set excludes E; the original evidence remains queryable before the correction cutoff |
| August 15 | B's September exit is known but not effective | B remains included |
| September 1 exact boundary | B exit effective | B excluded; no current-classification backfill into old snapshots |

### Acceptance assertions

1. Resolve evidence version before effective interval and taxonomy hierarchy.
2. Confirm exact microsecond/date boundaries for announcement, effective change,
   correction, and delisting.
3. Re-run June 10 after every later event: the same pinned build/result fingerprint
   remains unchanged and includes the later-delisted D.
4. Seal a second dataset containing E's correction; it cannot mutate the first
   dataset. At a knowledge cutoff before July 10 it still selects the old version;
   at/after July 10 it selects the correction.
5. Verify no automatic fallback on June 15. An explicit industry-level policy is a
   separate result and policy identity.
6. Verify peer membership is identical for revenue and gross-margin calculations,
   while feature eligibility/counts differ.
7. Verify the 135-day boundary, missing versus stale reasons, 60% coverage, target
   missing/invalid propagation, negative values, and exact ties.
8. Verify the worked midrank calculations with Decimal and deterministic peer order.
9. Verify accounting/unit/currency/share-basis conflicts are feature exclusions,
   not silent conversions.
10. Verify a current-only classification source and incomplete survivorship source
    fail closed; a named development override persists limitations and produces
    only `DEVELOPMENT` quality.
11. Walk one projected FeatureValue through relative result, statistic snapshot,
    every peer observation, membership/classification evidence, taxonomy release,
    source FeatureValues, manifest, provider admissions, and raw hashes.
12. Exact reruns reuse all published IDs/timestamps/fingerprints; injected failure
    rolls back group, observations, result, industry metrics, and FeatureValue
    projection atomically. Concurrent publication produces one winner.
13. Assert batch query counts to catch N+1 membership and feature retrieval.
14. Verify frozen Phase 2.4 `rel_ret_63_ind`, fundamental formulas, and Phase 2.5
    revision values remain byte-for-byte/numerically unchanged.

## R. Scale strategy

PostgreSQL remains sufficient for 10,000+ securities, multiple taxonomies,
historical membership, hundreds of eventual source features, and many requested
timestamps when access is set-based.

- Resolve a whole node's classification/eligibility heads with window functions or
  `DISTINCT ON` under exact dataset IDs, not one query per security.
- Retrieve one source feature for all member security IDs in one query, using a
  window partition by security for latest-PIT fundamental selection. Estimates and
  markets require exact snapshot keys.
- Compute the group distribution once and derive all targets' results in a batch.
  Leave-one-out medians can use sorted arrays/order-statistic logic without
  re-querying each target.
- Cache by immutable group/statistic fingerprint. A cache is an optimization, never
  evidence or authority.
- Seal dataset inputs before publication; use row/advisory locks and unique
  fingerprints only around publication, not whole-history serial loops.
- Measure `EXPLAIN (ANALYZE, BUFFERS)`, query count, rows scanned, wall time, and
  memory on realistic synthetic histories before adding indexes or infrastructure.
- Partitioning may be considered later from measured table size. No Spark, Kafka,
  Redis, Kubernetes, or separate analytical database is justified in v1.

## S. Backtesting compatibility

At each historical T, a future backtester will:

1. load the exact manifest and require admission over the complete universe,
   classification, identity, and feature lookbacks;
2. reconstruct the eligible historical universe, including later-failed/delisted
   names, from evidence known by T;
3. resolve the active taxonomy release and classifications using effective **and**
   knowledge time;
4. batch-select exact build-pinned FeatureValues available by T, applying latest-
   period and staleness rules;
5. resolve/publish peer and industry snapshots with candidates, exclusions,
   formulas, counts, coverage, and quality;
6. freeze the resulting peer features in the future research snapshot/prediction
   ledger before outcomes are read; and
7. reproduce the same fingerprints from pinned artifacts.

A correction, reclassification, new survivor, taxonomy revision, recalculated
feature, or provider backfill learned later cannot enter the sealed T build. A new
historical reconstruction uses a new build and is compared explicitly; it does not
rewrite the old prediction. Incomplete failed/delisted coverage prevents a
survivorship-safe label and, without an override, blocks that backtest purpose.

This supplies the later backtester with exact answers to: who belonged, what was
known, which values were eligible, why each company was included/excluded, and how
the percentile was computed.

## T. Future extensions

### Economic, competitive, and business-quality peers

Add new `peer_policy.kind` values and immutable evidence, not special cases inside
the classification resolver. Fundamental/economic peers need PIT scale/growth/
margin/capital-intensity/end-market attributes and a versioned distance/selection
formula. Competitive peers need product/customer/market evidence with effective
and knowledge time. Business-quality peers may use future dossiers and thesis
evidence. An LLM can suggest candidates for review but cannot publish authoritative
membership without cited evidence and a deterministic approved policy.

### Valuation peers

Reuse peer-group snapshots but add a separate valuation policy. It must name
enterprise/equity bridge, shares, fiscal/forward period, profitability domain,
currency/FX timestamp, accounting basis, negative-denominator handling, growth
adjustment, and stale-price rules. Phase 2.6 calculates no P/E, EV/Revenue,
EV/EBITDA, FCF yield, or growth-adjusted valuation.

### Relative growth and market share

`company revenue growth - peer median revenue growth` is a **relative-growth
percentage-point difference**. It may be consistent with share gain but does not
prove it. It can reflect acquisition, geography, mix, currency, accounting, or
peer-selection effects and must never be named `market_share_gain`.

Actual share requires a compatible market denominator with product, geography,
channel, period, unit/currency, methodology, coverage, publication availability,
revision vintage, and source rights. Future provider-neutral industry-observation
tables can key those vintages to taxonomy/economic-market nodes without hardwiring
one provider. Company numerator and market denominator scopes must reconcile.

### External industry growth

Future `industry_observations` can store qualified market size, units, price/volume,
capacity, utilization, or demand measures with period/effective/available/retrieved
time, source/build, units, geography, correction lineage, and raw provenance. A
versioned crosswalk links the observation's economic market to a taxonomy node;
absence of a crosswalk remains unknown.

### News and event aggregates

Future event intelligence can resolve each event's company to the classification
active at the event/research cutoff and aggregate diffusion or breadth by the same
historical node snapshot. Event occurrence, publication, correction, sentiment,
and membership times remain distinct. No news logic is part of v1.

### Robust statistics

MAD-based robust z-score may be useful after distribution and sample-size evidence.
It would require a versioned zero-MAD rule, minimum count, scaling constant, and
tail policy. V1 stops at median, median difference, and transparent percentile.

## U. Open decisions

Only external/provider decisions remain:

1. Select and contract an official historical GICS/classification source whose
   effective dates, publication/correction times, taxonomy-release history,
   delisted coverage, stable identifiers, and usage/retention/derived-data rights
   pass qualification.
2. Select and qualify the historical primary-common-equity universe/security-master
   source that can enforce one security per issuer and cover failed/delisted names.
3. Confirm that the selected fundamental corpus exposes sufficient accounting-
   framework and consolidation evidence for the strict v1 compatibility class;
   otherwise those observations remain explicitly unavailable.

These do not require changing the resolver, formulas, thresholds, fallback rule,
feature registry, or storage architecture. They block production data claims, not
synthetic implementation.

## V. Recommendation

**SAFE TO IMPLEMENT PHASE 2.6**

Implement the schema, sealed evidence builds, explicit registries, resolver,
statistics, on-demand persistence, and synthetic acceptance scenario in small
reviewable checkpoints. Keep all live outputs fail-closed until the open provider
and rights decisions are qualified. Do not implement factors, signals, rankings,
valuation, market-share claims, or AI-generated peers in Phase 2.6.
