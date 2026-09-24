## A. Verdict

**PASS — for the revised architecture specified below.**

The design is suitable for implementation without changing Phase 2.2 or Phase 2.4 calculations. Production historical coverage remains dependent on qualifying a provider’s actual data contract.

I reviewed the five requested documents, existing models, SEC normalization, raw storage, research resolvers, and feature runners.

Verified locally:

- Branch: `master`.
- `HEAD` and local `origin/master`: `cad71f308fd3403ddc85d9d01dfd21d27277229d`.
- Working tree clean.
- No files changed, migrations created, or commits made.
- Tests were not rerun for this read-only architecture task; the supplied 81/83 passing baseline remains the implementation checkpoint.

The principal decisions are:

1. Store immutable events against **absolute fiscal targets**.
2. Resolve relative horizons separately at research timestamp `T`.
3. Construct provider-specific consensus from identified contributors.
4. Preserve corrections as later knowledge, without treating them as new economic revisions.
5. Compare the **same absolute target** across lookbacks.
6. Keep consensus states authoritative and event-driven.
7. Apply PIT filtering to identity mappings, reporting evidence, and coverage metadata as well as estimate values.

## B. Final Architecture

```text
Provider connector
    ↓
Existing RawStorage / RawDataObject / IngestionRun
    ↓
Estimate normalizer and contract validation
    ↓
Immutable estimate observations
    ├── PIT contributor resolution
    ├── PIT fiscal-target mapping
    └── Exact metric/basis/currency/share-basis validation
    ↓
Contributor event replay
    ↓
Event-driven consensus states + relational membership
    ↓
Estimate revision calculator
    ↓
Existing ResearchFeature / FeatureValue
```

In parallel:

```text
Existing Filing + FinancialFact + archived reporting evidence
    ↓
ActualReportingResolver
    ↓
PIT reporting events + explicit fiscal sequence
    ↓
ForwardHorizonResolver(T)
    ↓
Absolute target ↔ relative horizon
```

Recommended components within the existing package structure:

| Component | Responsibility |
|---|---|
| `EstimateNormalizer` | Provider-specific parsing, timestamp interpretation, semantic validation and idempotency |
| `ContributorResolver` | PIT contributor identity and alias resolution |
| `EstimateTargetResolver` | Explicit fiscal identity and provider-period mapping |
| `ActualReportingResolver` | Reporting evidence adapter over existing filings/facts |
| `ForwardHorizonResolver` | FQ/FY classification at `T` |
| `EstimateEventResolver` | Corrections, withdrawals, contributor state and event ordering |
| `ConsensusCalculator` | Pure deterministic aggregation |
| `ConsensusStateWriter` | Atomic event-driven state and provenance persistence |
| `EstimateRevisionCalculator` | Magnitude, breadth and economic-event counts |
| `EstimateFeatureRunner` | `estimates_v1` registry and `FeatureValue` integration |

These extend the repository’s connector → raw → normalization → research pattern.

**Repository-specific constraints**

- `FinancialFact` has `filed_date`, fiscal fields and raw provenance, but no dedicated `available_at`.
- `Filing` has `acceptance_datetime`, filing/report dates and accession identity, but no explicit fiscal year/quarter.
- `Company.fiscal_year_end` and `Security.currency` are current metadata, not authoritative historical fiscal or reporting-currency histories.
- Existing `FeatureStatus` already supports `VALID`, `MISSING`, and `INVALID`; estimates do not need a new shared status enum.
- The fundamental runner selects every active registry entry without restricting the feature family. See Section J for a compatibility policy that avoids changing frozen code.

## C. Canonical Tables

The following are proposed tables, not migrations.

Use `BIGINT` identifiers for high-volume new tables, exact PostgreSQL `NUMERIC` for values, `DATE` for period boundaries, and UTC `TIMESTAMPTZ` for new temporal columns. Preserve raw decimal precision; use a fixed, versioned decimal context for arithmetic and output rounding.

### Reference and temporal-control tables

| Table | Important fields and constraints |
|---|---|
| `estimate_provider_profiles` | `id`, `provider`, `dataset_name`, `contract_version`, `methodology_class`, supported identities/events, ordering policy, timestamp policy, currency policy, share-basis policy, `source_raw_object_id`, `available_at`, `created_at`. Immutable profile versions. |
| `estimate_datasets` | `id`, profile references, `dataset_version`, normalization/calculation manifest hashes, input retrieval watermark, completion status, `created_at`. Identifies a reproducible imported corpus or published historical build. |
| `estimate_fiscal_periods` | `id`, `company_id`, `fiscal_calendar_id`, `target_fiscal_year`, `target_period_type`, `target_fiscal_period`. Immutable absolute period identity. Annual period is explicitly `FY`, never SQL `NULL`. |
| `estimate_fiscal_period_evidence` | `id`, `fiscal_period_id`, provider period identifier, `period_start`, `target_period_end`, predecessor/successor period references, mapping method/version, reporting currency, `available_at`, source references, `supersedes_evidence_id`. Append-only metadata and mappings. |
| `estimate_contributors` | `id`, `provider_profile_id`, canonical contributor key, identity kind, `created_at`. Provider-scoped canonical contributor. |
| `estimate_contributor_aliases` | `id`, `provider_profile_id`, provider analyst/broker identifier or approved fallback key, `contributor_id`, effective interval, `available_at`, evidence reference, `supersedes_alias_id`. Append-only identity evidence. |
| `estimate_coverage_assertions` | `id`, provider/profile, company/security scope, coverage kind, metric scope, covered start/end, completeness status, `available_at`, raw evidence. Distinguishes complete history, gaps and snapshot-only coverage. |

`fiscal_calendar_id` distinguishes explicitly documented calendar regimes, including fiscal-year transitions. It is not generated from calendar month.

### Observation table

`estimate_observations`:

```text
id
provider_profile_id
security_id
fiscal_period_id

provider_estimate_id
provider_event_id
provider_event_version
provider_sequence
logical_event_key
semantic_payload_hash
dedupe_key

contributor_alias_id
provider_analyst_id
analyst_name
broker
provider_broker_id

metric
metric_basis
revenue_basis
methodology_class
eps_share_type
share_basis_key
scope_key

estimate_value
currency
unit
source_unit_multiplier

event_type
provider_event_type
event_type_evidence
previous_economic_event_key
supersedes_observation_id
correction_action

provider_as_of
published_at
available_at
retrieved_at
timestamp_precision
timestamp_timezone
availability_basis

target_resolution_status
status
reason

source_raw_object_id
source_record_locator
normalization_version
created_at
```

Important constraints:

- Accepted value-bearing events require resolved target, contributor, finite value, currency, unit and semantic basis.
- A withdrawal has no numeric value.
- A correction explicitly identifies its affected logical event/version.
- Self-reference, correction cycles and incompatible supersession references are prohibited.
- Source linkage is mandatory.
- Unresolved records remain stored or quarantined with explicit reasons; they do not enter consensus.

### Review of every proposed observation field

| Proposed field | Final decision |
|---|---|
| `id` | Keep as storage identity; never economic ordering authority. |
| `company_id` | Remove from observation; derive through the target/security relationship and enforce company consistency. |
| `security_id` | Keep. Estimates remain security-specific, especially EPS/share-class semantics. |
| `provider` | Normalize into immutable `provider_profile_id`. |
| `provider_estimate_id` | Keep, nullable when unavailable; represents the provider’s estimate stream. |
| `provider_event_id` | Keep; namespace by provider/profile and record version. |
| `provider_sequence` | Keep when its ordering semantics are documented. |
| `contributor_id` | Resolve through the observation’s alias and PIT mapping; store the selected canonical ID in consensus membership. Do not retroactively rewrite observation identity. |
| `provider_analyst_id` | Keep as source identity evidence. |
| `analyst_name`, `broker` | Keep immutable source labels; neither is independently a reliable unique key. |
| `metric` | Keep: `EPS`, `REVENUE`. |
| `metric_basis`, `revenue_basis` | Keep with metric-dependent constraints and explicit `NOT_APPLICABLE`. |
| Fiscal target fields | Normalize into `estimate_fiscal_periods`; expose them through joins/views. |
| `target_period_end` | Move to versioned period evidence; retain the provider’s original representation in raw data. |
| `target_identity_confidence` | Replace a vague score with categorical resolution status and evidence IDs. |
| `estimate_value`, `currency`, `unit` | Keep; zero is valid, missing is not zero. |
| `event_type` | Keep canonical semantics, alongside original provider classification. |
| Four timestamps | Keep, with separate semantics below. |
| `timestamp_precision` | Keep; distinguish source precision from the conservative visibility boundary. |
| `source_raw_object_id` | Keep mandatory. |
| `supersedes_observation_id` | Keep for correction-version relationships, not ordinary economic succession. |
| `created_at` | Keep as system audit time; never use as historical publication time. |

Added fields prevent otherwise serious ambiguity: basic versus diluted EPS, split/share basis, provider methodology class, report scope, event-version identity, causal predecessor, availability evidence, and source record locator.

### Consensus and reporting tables

| Table | Important fields and constraints |
|---|---|
| `estimate_consensus_series` | `id`, profile, `security_id`, `fiscal_period_id`, metric/bases, methodology, currency/unit, EPS/share basis, scope. Unique exact comparison group. |
| `estimate_consensus_states` | `id`, dataset, `series_id`, `state_available_at`, `calculation_version`, state fingerprint, `mean`, `median`, `high`, `low`, `stddev`, `contributor_count`, `status`, `reason`, aggregation/minimum-contributor policy, build timestamp. |
| `estimate_consensus_members` | `(consensus_state_id, contributor_id)` primary key; selected observation/version, selected alias evidence, value. Exactly one member per contributor. |
| `estimate_consensus_exclusions` | State ID, reason, count, optional observation/identity references. Store relevant exclusion evidence without copying the entire event history. |
| `estimate_actual_reporting_events` | `id`, fiscal period, event action, evidence kind, `available_at`, timestamp precision, existing filing reference, raw reference, superseded reporting-event reference, resolver version. |
| `estimate_actual_reporting_facts` | Reporting-event ID and existing `FinancialFact` ID. Relational link to supporting fiscal/accounting evidence. |

A small `estimate_feature_evidence` relation links a `FeatureValue` to current/prior consensus states, reporting events, relevant observations and identity/coverage evidence. Compact calculation parameters can remain in existing `FeatureValue.provenance`.

## D. Absolute Fiscal Target Model

Separate **fiscal period**, **estimate target**, and **consensus series**.

**Fiscal period identity**

```text
company_id
fiscal_calendar_id
target_fiscal_year
target_period_type
target_fiscal_period
```

Allowed combinations:

```text
QUARTER     + Q1 | Q2 | Q3 | Q4
FISCAL_YEAR + FY
```

**Estimate target identity**

```text
security_id
absolute fiscal period identity
metric
metric_basis
revenue_basis
EPS share type / scope where applicable
```

**Consensus series identity**

Adds:

```text
provider profile
methodology class
currency
canonical unit
share basis
scope
```

Provider, currency and share basis distinguish comparable series; they do not change which fiscal quarter is being forecast.

**`target_period_end` is validation metadata, not identity.**

It can be corrected without renaming the target. A date conflict creates ambiguity until explicit evidence resolves it.

Fiscal mapping rules:

1. Accept explicit provider fiscal metadata with documented year-label conventions.
2. Map provider identifiers through an evidenced, versioned fiscal crosswalk.
3. Preserve SEC fiscal authority for actuals.
4. Never use month, calendar quarter, or approximate duration to invent fiscal identity.
5. A date-only provider target is insufficient unless an explicit provider/company crosswalk uniquely maps it.
6. Missing identity → `MISSING / TARGET_PERIOD_UNKNOWN`.
7. Conflicting identity → `MISSING / TARGET_PERIOD_AMBIGUOUS`.

Changing fiscal-year conventions requires a new calendar regime and explicit sequence links. Do not silently relabel historical targets.

## E. Forward Horizon Resolver

Interface:

```text
resolve(security_id, research_timestamp_T, horizon, dataset, policy)
    → absolute_period_id
      status/reason
      fiscal_evidence_ids
      reporting_event_ids
      coverage_evidence_ids
```

It can also classify an existing absolute target.

### Actual-reporting source of truth

Use an append-only reporting-event projection over:

- Existing `Filing` records.
- Existing `FinancialFact` fiscal metadata.
- Archived filing/release evidence establishing which period was actually reported.
- Qualified structured release metadata where an earnings release precedes the filing.

Do not treat any fact bearing `fy/fp` as proof that the corresponding period first became known. Comparative facts and amendments must be tied to the filing’s actual reporting context.

A filing date alone does not prove an intraday publication time. When available, use verified acceptance/dissemination evidence; otherwise apply the conservative date-only policy.

Q1’s existing `UNKNOWN` **duration shape** does not necessarily make its fiscal identity unknown. Explicit reporting-context evidence can establish Q1 reporting without deriving a standalone value from YTD data.

### Algorithm

1. Load only fiscal mappings and reporting evidence with `available_at <= T`.
2. Resolve the explicit company fiscal sequence.
3. Determine reporting status separately for quarterly and annual periods:
   - `REPORTED`
   - `NOT_REPORTED_AS_OF_T`, supported by adequate coverage
   - `UNKNOWN`
4. Establish a verified reported-period anchor.
5. Walk explicit successor links.
6. The first demonstrably unreported quarter is `FQ1`; its next three successors are `FQ2–FQ4`.
7. The first demonstrably unreported fiscal year is `FY1`; its successor is `FY2`.
8. Do not skip a missing estimate period. If Q2 has no estimates, Q3 remains FQ2.
9. A reporting gap, conflicting calendar mapping, or insufficient reporting coverage returns `FORWARD_HORIZON_UNRESOLVED`.
10. A reported period remains directly queryable as historical.

**Absence of a reporting row is not, by itself, proof that no report occurred.**

Only enough fiscal sequence evidence to reach the requested horizon is required. FQ1 does not require four populated estimate targets. FQ4 requires four known fiscal successors, but not four sets of estimates.

Quarterly and annual rollover are independent. Reporting Q4 does not automatically prove the annual period was reported, and an annual filing does not fabricate a standalone Q4 actual. A full-year reporting document can close both periods only when its context establishes both statuses.

### Worked rollover cases

| Case | Required behavior |
|---|---|
| Normal rollover | Q1 report becomes available May 20. Before that instant Q1 is FQ1; from that instant Q2 is FQ1. |
| January fiscal year-end | Provider explicitly identifies April 2026 as FY2027 Q1 and July 2026 as FY2027 Q2. Use those identities; April does not imply calendar Q2. |
| After-hours release | Release available at 16:05 New York time. A 16:00 snapshot retains the old horizon; 16:05 and later use the new horizon. Trading execution timing belongs to the future backtester. |
| Missing filing metadata | Do not infer the period from `report_date` month. Return unresolved horizon unless another qualified source supplies the reporting context. |
| Amended filing | A later amendment does not move the original first-known reporting boundary. Newly learned identity corrections apply only from their own availability. |
| FY rollover | Once FY2027 annual reporting is PIT-known, FY2028 becomes FY1. Quarter estimates remain on their own fiscal sequence. |

## F. Contributor Identity

**Consensus is provider-scoped in v1. Never pool vendors automatically.**

Identity hierarchy:

1. `provider + stable provider_analyst_id`, with documented scope.
2. Explicit provider replacement/alias mapping to a canonical contributor.
3. `provider + canonical broker identifier + conservatively normalized full analyst name`, only when uniqueness is evidenced.
4. Otherwise unresolved and excluded.

Normalization may standardize Unicode, whitespace and case. It must not use fuzzy name similarity to merge people.

| Situation | Policy |
|---|---|
| Analyst ID missing | Accept the approved broker/name fallback only with sufficient identity evidence. |
| Analyst changes broker | Stable analyst ID preserves identity; broker is affiliation metadata. Without an evidenced crosswalk, identity continuity is unresolved. |
| Broker spelling changes | Use explicit broker aliases. Do not create another contributor merely because display spelling changed. |
| Provider renumbers IDs | Append a PIT alias mapping. Earlier snapshots continue using earlier knowledge. |
| Same broker, multiple analysts | Distinct analysts count separately only when the provider identifies independent forecast contributors. |
| Multiple names on one house estimate | One forecast contributor, not one vote per named coauthor. |
| Unresolvable identity | Exclude with `CONTRIBUTOR_UNRESOLVED`. |
| Suspected ID reuse or collision | Quarantine the affected identity group; do not arbitrarily choose a person. |

Aliases have both effective dates and knowledge timestamps. An alias discovered at T2 must not silently merge contributors in a T1 reconstruction.

After a merger of aliases becomes known, select at most one current estimate for the resolved contributor. The deduplication may change consensus membership at T2; it is not an analyst revision.

## G. Consensus Algorithm

### Event semantics

| Event | Exact meaning |
|---|---|
| `NEW` | A contributor begins an active estimate episode for this exact target and semantic series. Re-entry after withdrawal is a new episode. |
| `REVISION` | An evidenced economic update to an existing active estimate. Only a nonzero comparable value change counts as a revision. |
| `WITHDRAWAL` | Explicitly terminates that contributor’s active estimate for the specified scope. |
| `CORRECTION` | Changes or retracts a provider representation of a referenced event. It is not a new economic estimate position. |

Do not infer a withdrawal from disappearance in a response unless the provider explicitly documents that the response is a complete replacement snapshot with that meaning.

A provider snapshot difference alone cannot prove whether a change was a revision, correction, or missed intermediate activity. Such feeds cannot supply canonical economic revision counts.

### Correction replay

Each economic event has a stable logical identity; corrections supply later versions of that event.

At T:

- Select only correction versions available by T.
- Apply corrections to their referenced event.
- Retain the event’s original economic position in the contributor sequence.
- A correction to an old event cannot overwrite a newer independent revision.
- A corrected withdrawal remains a withdrawal unless explicit correction semantics retract or replace it.
- A correction moving a record to another target removes it from the old group and introduces its corrected representation into the new group at the correction’s availability. It is not a new analyst initiation.

Unlinked corrections and conflicting correction branches are excluded pending resolution.

### Deterministic steps

For an exact series and T:

1. Pin dataset, provider profile, calculation version and policy.
2. Apply `available_at <= T` to observations and all supporting temporal evidence.
3. Resolve the absolute target.
4. Resolve PIT contributor identity.
5. Deduplicate provider transmissions using documented event identity/version and semantic payload hash.
6. Resolve correction chains.
7. Replay each contributor’s economic sequence, applying withdrawals and episode boundaries.
8. Select its latest active economic event.
9. Validate that selected event against exact metric, basis, methodology, currency, unit, share basis and scope.
10. Exclude invalid selected heads. **Do not resurrect an older estimate merely because the latest event is unusable.**
11. Require at least three resolved contributors.
12. Compute equally weighted statistics and persist relational members/exclusions.

Deduplicate before aggregation or event counting.

### Ordering

`available_at` controls visibility. It does not always express economic succession.

Within a contributor stream:

1. Explicit causal predecessor relationships.
2. Provider sequence, where its scope and semantics are guaranteed.
3. Documented publication/economic event ordering when sequence is unavailable.

A delayed older event must not replace an already known newer event.

For otherwise equivalent transmissions, stable provider keys/hashes give reproducible ordering. Conflicting events with identical temporal/sequence precedence produce `EVENT_ORDER_AMBIGUOUS`; an auto-increment database ID must not decide financial meaning.

Events sharing one availability timestamp are processed as one atomic boundary. Queries at that boundary see the completed state, not intermediate ingestion order.

### Aggregates

For selected values \(x_1,\ldots,x_n\):

\[
\bar{x}=\frac{\sum x_i}{n}
\]

\[
s=\sqrt{\frac{\sum(x_i-\bar{x})^2}{n-1}}
\]

Store mean, median, high, low, sample standard deviation and count.

`ddof = 1` is an explicit descriptive convention for contributor dispersion; it does not imply analysts form an independent random sample.

The three-contributor threshold is a minimum breadth guardrail. It prevents a single forecast or pair from masquerading as broad consensus. It is not a statistical confidence guarantee.

With fewer than three contributors:

- Preserve count and membership.
- Set standard consensus aggregates to null.
- Return `MISSING / INSUFFICIENT_CONTRIBUTORS`.

### Timestamp policy

| Timestamp | Meaning |
|---|---|
| `provider_as_of` | Provider’s stated observation/reference time; not automatically public availability |
| `published_at` | Publication/dissemination time supported by provider evidence |
| `available_at` | Earliest conservatively justified visibility time under the pinned availability policy |
| `retrieved_at` | When this system received the source payload |

Two explicit availability modes:

- **Certified historical event archive:** documented historical dissemination time can precede retrieval.
- **Uncertified history/current snapshot:** use first retrieval; never backdate from `provider_as_of`.

A vendor correction published at T2 remains T2 knowledge even if it refers to an old estimate date.

**Final date-only policy:** use the start of the following day in the documented source timezone, converted to UTC.

```text
source date D, timezone Z
available_at = midnight beginning D+1 in Z, converted to UTC
timestamp_precision = DATE_ONLY
```

This is a conservative visibility boundary, not a claimed publication instant.

- Use UTC only when the provider defines the date in UTC.
- Use exchange-local time only when the provider defines its dates that way.
- Do not use exchange close: estimates may be published after close.
- Unknown timezone with no usable timing contract falls back to retrieval.
- Do not universally assign `23:59:59 UTC`; it may expose US-local late-evening information too early and excludes fractional seconds at day-end.

## H. Revision Mathematics

### Lookbacks

Use **7, 30 and 90 calendar days**, defined as UTC timestamp subtraction:

```text
cutoff = T - timedelta(days=N)
```

Resolve the target at T once:

```text
P = ForwardHorizonResolver(T, requested_horizon)

current = latest state for P with state_available_at <= T
prior   = latest state for P with state_available_at <= cutoff
```

Never independently resolve FQ1 at both endpoints.

The prior state must be the latest state, including a missing/invalid state. Do not skip an insufficient-contributor state to find an older valid one.

Weekends and holidays need no special treatment. If both cutoffs select the same valid state, the change is zero.

### EPS: final v1 formulas

Always provide absolute change:

\[
\Delta_{\mathrm{EPS}}=new-old
\]

Use a bounded scale-normalized change:

\[
S_{\mathrm{EPS}}=
\begin{cases}
0, & old=0 \land new=0\\
\dfrac{new-old}{\max(|old|,|new|)}, & \text{otherwise}
\end{cases}
\]

This is the proposed max-denominator formula with **no positive economic floor**.

Properties:

- Preserves the sign of economic improvement/deterioration.
- Defined for zero and negative EPS.
- Lies in `[-2, 2]`.
- Invariant to common currency/unit scaling.
- Does not claim a percentage-growth interpretation.
- Does not require a currency-specific or split-sensitive `0.25` assumption.

A fixed `0.25` floor has no universal economic justification. A calibrated floor would need an explicit currency/share-normalization and empirical methodology outside this phase.

The floor-free metric can assign a large score to a small absolute change. Therefore absolute change is part of the canonical output; downstream ranking must not interpret the scaled value alone as economic materiality.

| Old → new | Absolute Δ | Naive `new/old−1` | Candidate floor `0.25` | Selected scaled change |
|---|---:|---:|---:|---:|
| `1.00 → 1.20` | `+0.20` | `+0.20` | `+0.166667` | `+0.166667` |
| `−1.00 → −0.50` | `+0.50` | `−0.50` | `+0.50` | `+0.50` |
| `−0.20 → +0.20` | `+0.40` | `−2.00` | `+1.60` | `+2.00` |
| `+0.20 → −0.20` | `−0.40` | `−2.00` | `−1.60` | `−2.00` |
| `0 → 0.50` | `+0.50` | Undefined | `+1.00` | `+1.00` |
| `0.01 → 0.10` | `+0.09` | `+9.00` | `+0.36` | `+0.90` |
| `0 → 0` | `0` | Undefined | `0` | `0` |

### Revenue

For `old > 0` and `new >= 0`:

\[
R_{\mathrm{revenue}}=\frac{new-old}{old}
\]

Store a ratio: `0.10` means 10%.

- `old = 0`: `INVALID / DENOMINATOR_UNSUPPORTED`.
- Negative canonical consolidated revenue estimate: `INVALID / INVALID_NUMERIC_VALUE` in v1.
- No arbitrary dollar floor.
- No hidden clipping; large revisions remain inspectable.

### Corrections and composition

**Consensus change is not identical to analyst revision momentum.**

A consensus mean can change because of:

- Economic revisions.
- Initiations.
- Withdrawals.
- Corrections.
- Contributor identity resolution.

The magnitude features measure **as-known consensus movement**, including those effects. Their provenance identifies the causes.

Economic momentum interpretation requires the separate revision breadth/count evidence. A correction-only move must never be labeled an analyst upgrade or downgrade signal.

### Currency, basis and corporate-action compatibility

- EPS basis enum: `GAAP`, `ADJUSTED`, `NORMALIZED`, `UNKNOWN`.
- EPS share type: `DILUTED`, `BASIC`, `UNKNOWN`.
- Revenue basis enum: `REPORTED`, `ORGANIC`, `CONSTANT_CURRENCY`, `SEGMENT`, `UNKNOWN`.
- For the irrelevant basis field, use `NOT_APPLICABLE`.

V1 canonical features use **ADJUSTED DILUTED EPS** and **REPORTED consolidated revenue**.

Other known EPS bases may have separate directly queryable consensus series. `UNKNOWN` records are retained but cannot form standard consensus. “Adjusted” records still require a compatible provider methodology class.

Revenue v1 excludes organic, constant-currency, segment and unknown bases from canonical consensus.

Aggregate only estimates in the PIT-evidenced reporting currency. Provider-normalized data is acceptable only when it is already expressed in that same currency and has documented historical normalization semantics.

No internal FX conversion. No cross-currency lookback.

EPS must share an explicit comparable share basis. V1 does not implement estimate split adjustment. Cross-split or ADR-ratio comparisons without comparable evidence return `SHARE_BASIS_MISMATCH`. Reuse existing corporate-action evidence to detect incompatibility without changing Phase 2.4.

## I. Breadth and Count

Use the interval:

```text
(T − N calendar days, T]
```

### Economic transition classification

For each documented economic update, compare:

```text
same contributor
same absolute target
same metric/basis/methodology
same currency/unit/share basis
same active episode
```

against its prior active estimate.

Classifications:

- `UP`
- `DOWN`
- `UNCHANGED`
- `NEW`
- `WITHDRAWAL`
- `CORRECTION`

Use exact normalized decimals; do not add an unexplained floating-point tolerance.

A same-value reaffirmation is `UNCHANGED`. An exact retransmission is deduplicated before classification.

Corrections update the baseline for subsequent economic events when they affect the active estimate. They do not themselves create an UP/DOWN transition.

For reproducibility, preserve the prior observation/version used when an economic transition became known. Later provider corrections do not silently rewrite previously observed revision-direction history. If a provider explicitly retracts the economic classification itself, record that separately; affected economic-activity results become unresolved until the event meaning is established.

### Canonical breadth

Use each contributor’s **latest comparable economic update within the window**, compared with its immediate prior active estimate.

Each contributor gets at most one breadth vote, even if it revised five times.

Let:

- `U`: contributors whose selected update was UP.
- `D`: contributors whose selected update was DOWN.
- `Z`: contributors whose selected update was UNCHANGED.

Canonical metric:

\[
Breadth=\frac{U-D}{U+D}
\]

- `+1`: all directional votes upward.
- `−1`: all directional votes downward.
- `0`: equal upward and downward votes.
- `U + D = 0`: `MISSING / NO_DIRECTIONAL_REVISIONS`.

Alternative:

\[
UpShare=\frac{U}{U+D+Z}
\]

This measures the share of comparable updates that were upward, but its interpretation changes with reaffirmation practices. Retain `U/D/Z` in evidence; do not add another canonical feature.

Initiations, withdrawals and corrections do not vote. A subsequent withdrawal does not erase an earlier economic revision from an activity window.

A contributor initiated within the window can vote if it later makes a genuine revision against that new baseline. The initiation itself remains excluded.

### Counts

```text
revision_count_30d =
number of distinct documented economic value-changing events
with T−30d < available_at <= T
```

This counts **events**, not unique contributors.

Exclude:

- Duplicate transmissions.
- Initiations.
- Same-value reaffirmations.
- Pure corrections.
- Withdrawals.
- Currency/unit/share-basis incompatibilities.

Return zero only when the event history is sufficiently complete to establish that no revisions occurred. Otherwise return `MISSING / INCOMPLETE_EVENT_HISTORY`.

`initiation_count` and `withdrawal_count` belong in the v1 calculation/evidence response, but not in the initial persisted feature registry.

## J. Feature Registry

Consensus states are the source of truth. Do **not** duplicate eight raw consensus means into `FeatureValue`.

Research snapshots, valuation components and analyst interfaces query:

```text
EstimateConsensusRepository.get_as_of(
    security, absolute_target or horizon, metric, basis, T, profile, dataset
)
```

The response includes statistics, status, members, reporting context and provenance.

Final `estimates_v1` registry: **15 features**.

Explicit basis in the IDs prevents ambiguity.

| Family | Canonical IDs |
|---|---|
| EPS absolute, FQ1 | `eps_adjusted_diluted_consensus_change_abs_7d_fq1` |
| | `eps_adjusted_diluted_consensus_change_abs_30d_fq1` |
| | `eps_adjusted_diluted_consensus_change_abs_90d_fq1` |
| EPS scaled, FQ1 | `eps_adjusted_diluted_consensus_change_scaled_7d_fq1` |
| | `eps_adjusted_diluted_consensus_change_scaled_30d_fq1` |
| | `eps_adjusted_diluted_consensus_change_scaled_90d_fq1` |
| EPS annual | `eps_adjusted_diluted_consensus_change_scaled_30d_fy1` |
| EPS activity | `eps_adjusted_diluted_revision_breadth_30d_fq1` |
| | `eps_adjusted_diluted_revision_count_30d_fq1` |
| Revenue, FQ1 | `revenue_reported_consensus_change_pct_7d_fq1` |
| | `revenue_reported_consensus_change_pct_30d_fq1` |
| | `revenue_reported_consensus_change_pct_90d_fq1` |
| Revenue annual | `revenue_reported_consensus_change_pct_30d_fy1` |
| Revenue activity | `revenue_reported_revision_breadth_30d_fq1` |
| | `revenue_reported_revision_count_30d_fq1` |

No FQ2/FY2 feature permutations in v1. Those consensus horizons remain directly queryable.

### Existing FeatureValue integration

- `ResearchFeature.version = estimates_v1`.
- Select estimate features by explicit family and allowlist.
- `FeatureValue.period_end = research date`, consistent with snapshot-oriented market features.
- Put absolute target identity in relational evidence, not in that date field.
- `FeatureValue.available_at = T` for an on-demand snapshot calculation.
- Store actual execution time in `calculated_at`.
- Keep non-null `period_end` to avoid nullable uniqueness behavior.
- Pin provider profile and dataset in the calculation identity. A full version identifier such as `estimates_v1:<profile>:<dataset>` prevents different data contracts from colliding under the existing unique key.
- Estimate retrieval filters the full calculation identity; it must not choose arbitrarily across providers/builds.

**Frozen-runner compatibility:** the existing [fundamental runner](../src/investment_research/research/runner.py) selects all active features.

Therefore estimate registry entries remain `active=False` for the legacy generic scan. The new estimate runner uses an explicit enabled-profile/feature allowlist and version filter, independent of that legacy flag. This must be documented and regression-tested; estimate features must not be temporarily activated during execution.

This is a narrow compatibility measure. It avoids changing frozen calculation code or allowing the fundamental runner to emit spurious estimate rows.

### Feature refresh boundaries

Rolling features can change without a new estimate:

- An old consensus state crosses `T−N`.
- A revision leaves the 30-day window.
- A reporting event changes the horizon.

Therefore compute features on demand at requested research timestamps. Persist requested snapshots only. Do not treat the last feature row as indefinitely valid merely because no new estimate arrived.

## K. Status / Reason Codes

`VALID` requires a value for numeric outputs; reason is null. `MISSING` and `INVALID` have null numeric outputs.

| Status | Canonical reasons |
|---|---|
| `MISSING` | `NO_ESTIMATES` |
| | `INSUFFICIENT_CONTRIBUTORS` |
| | `TARGET_PERIOD_UNKNOWN` |
| | `TARGET_PERIOD_AMBIGUOUS` |
| | `FORWARD_HORIZON_UNRESOLVED` |
| | `ACTUAL_REPORTING_STATUS_UNKNOWN` |
| | `FISCAL_SEQUENCE_UNRESOLVED` |
| | `NO_PRIOR_CONSENSUS` |
| | `NO_VALID_PRIOR_CONSENSUS` |
| | `CONTRIBUTOR_UNRESOLVED` |
| | `CONTRIBUTOR_IDENTITY_AMBIGUOUS` |
| | `METRIC_BASIS_AMBIGUOUS` |
| | `METRIC_BASIS_UNSUPPORTED` |
| | `REPORTING_CURRENCY_UNKNOWN` |
| | `CURRENCY_MISMATCH` |
| | `UNIT_MISMATCH` |
| | `SHARE_BASIS_MISMATCH` |
| | `WITHDRAWN` |
| | `NO_DIRECTIONAL_REVISIONS` |
| | `INCOMPLETE_EVENT_HISTORY` |
| | `EVENT_SEMANTICS_UNKNOWN` |
| | `AVAILABILITY_UNVERIFIED` |
| `INVALID` | `INVALID_NUMERIC_VALUE` |
| | `DENOMINATOR_UNSUPPORTED` |
| | `MALFORMED_TARGET_PERIOD` |
| | `MALFORMED_TIMESTAMP` |
| | `INVALID_UNIT` |
| | `EVENT_ORDER_AMBIGUOUS` |
| | `EVENT_ID_CONFLICT` |
| | `INVALID_SUPERSESSION` |
| | `SUPERSESSION_CYCLE` |
| | `INCONSISTENT_SECURITY_COMPANY` |

Distinction:

- A valid EUR observation cannot satisfy a USD query: `MISSING / CURRENCY_MISMATCH`.
- An unparsable currency/unit/value is malformed input: `INVALID`.
- An unsupported valid basis is missing coverage for the requested calculation.
- Zero estimates are valid values. Zero denominators are a calculation-domain issue.

A consensus with three valid contributors remains valid even if other records are excluded. Its exclusion ledger records those problems.

For deterministic primary reasons, evaluate request/identity validity first, then horizon, semantic compatibility, coverage, contributor sufficiency, prior state and formula domain. Preserve all secondary exclusions.

## L. Provenance

A result must reproduce from:

```text
research timestamp T
absolute target
provider profile / dataset
calculation and normalization versions
selected observation versions
contributor mapping evidence
fiscal mapping evidence
reporting events
coverage assertions
current/prior consensus state IDs
aggregation policy
lookback and formula parameters
```

Required guarantees:

1. Every selected observation traces to raw bytes and record location.
2. Every membership row identifies the exact contributor and observation version.
3. Every relative horizon traces to fiscal and reporting evidence.
4. Every revision transition identifies its prior and new observation versions.
5. Exclusions are explainable.
6. All temporal dependencies satisfy `available_at <=` their applicable research cutoff.
7. Original states remain available after correction.
8. Historical dataset builds are pinned; later backfill does not silently change an already published research run.

Certified late historical imports may add legitimate earlier knowledge, but they require a new dataset/build identity and explicit reconstruction. Uncertified backfills become visible at retrieval.

**Raw-storage integration detail:** the current raw writer uses second-resolution filenames. The estimate connector should provide a unique request identifier plus content hash in the existing `identifier` path, preventing two different estimate responses within one second from overwriting each other. Verify the stored content hash on replay. This requires no change to frozen research phases.

## M. Provider Contract

An eligible analyst-event connector must supply or deterministically establish:

- Security/company mapping.
- Stable event identity and version identity.
- Estimate-stream identity or equivalent.
- Explicit absolute fiscal target.
- Contributor identity and its scope.
- Metric and semantic basis.
- EPS basic/diluted and share-adjustment basis.
- Revenue consolidation/scope.
- Value, currency and unit scaling.
- Availability/publication evidence and timestamp precision/timezone.
- Event ordering semantics.
- Correction and withdrawal semantics.
- Historical coverage boundaries and known gaps.
- Raw evidence and normalization provenance.

If event type is absent, derivation is allowed only under a documented provider contract that distinguishes economic updates from corrections and duplicates.

Provider capability tiers:

| Tier | Permitted use |
|---|---|
| Complete historical analyst-event feed | Consensus reconstruction, contributor breadth and event counts |
| Identified estimate snapshots | As-observed consensus; economic event counts remain missing unless independently supported |
| Historical PIT consensus snapshots | Provider-consensus research with explicit methodology; no fabricated members or analyst activity |
| Today’s consensus only | Current/as-retrieved context; no reconstruction of past revisions |

**Today’s consensus cannot reconstruct historical analyst estimates or valid historical revision activity.**

### Data-source reality

| Category | Sources and assessment |
|---|---|
| Free/public | SEC submissions/XBRL and issuer releases are useful for actual-reporting evidence and fiscal identity. SEC’s documented APIs provide filings and reported financial facts, not a complete sell-side estimate-event history. [SEC API documentation](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) |
| Limited/free API ecosystems | Alpha Vantage documents annual/quarterly EPS and revenue estimates, analyst counts and revision history. Those descriptions alone do not establish analyst-level identity, correction versions or complete withdrawal history. Endpoint access and historical semantics require qualification. [Alpha Vantage documentation](https://www.alphavantage.co/documentation/) |
| Limited/free API ecosystems | FMP documents projected analyst financial estimates. Historical target periods must not be mistaken for historical as-known vintages; the connector needs sample data and an explicit timing/event contract. Free-tier entitlement to required fields is not established by the documentation reviewed. [FMP Financial Estimates](https://site.financialmodelingprep.com/developer/docs/stable/financial-estimates) |
| Commercial/institutional | LSEG documents I/B/E/S estimates and PIT historical snapshots. It is a credible qualification candidate, but a particular licensed dataset must prove the required detail-event fields and timing semantics. [LSEG quantitative-data brochure](https://www.lseg.com/content/dam/data-analytics/en_us/documents/brochures/lseg-data-for-quant-research-brochure.pdf) |
| Commercial/institutional | FactSet documents a PIT consensus product designed to preserve historical snapshots without later QA/currency/dilution changes. Consensus snapshots alone still cannot provide this design’s analyst-level counts and membership. Detail-history rights and semantics need separate confirmation. [FactSet PIT methodology](https://insight.factset.com/hubfs/Resources%20Section/White%20Papers/ID11996_point_in_time.pdf) |

The difficult free-data requirements are historical analyst-level revisions, stable analyst identity, withdrawals, correction versions, historical PIT consensus, basis compatibility and detailed broker provenance.

That assessment does not justify weakening the internal model. A limited first connector should produce explicit missing capabilities.

No provider purchase or implementation is proposed here.

## N. Database / Index Design

Keep PostgreSQL. Use ordinary batch ingestion and transactions.

### Proposed indexes

| Table | Index |
|---|---|
| Fiscal periods | Unique `(company_id, fiscal_calendar_id, target_fiscal_year, target_period_type, target_fiscal_period)` |
| Fiscal evidence | `(fiscal_period_id, available_at DESC, id DESC)` |
| Contributor aliases | `(provider_profile_id, alias_key, available_at DESC)` |
| Observations | Unique `(provider_profile_id, dedupe_key)` |
| Observations | `(security_id, metric, fiscal_period_id, available_at DESC)` |
| Observations | `(contributor_alias_id, fiscal_period_id, metric, available_at DESC)` |
| Observations | `(provider_profile_id, logical_event_key, available_at DESC)` |
| Observations | `(supersedes_observation_id)` |
| Consensus series | Unique complete semantic grouping key |
| Consensus states | Unique `(dataset_id, series_id, calculation_version, state_available_at)` |
| Consensus states | `(dataset_id, series_id, calculation_version, state_available_at DESC)` with selected aggregate columns included if measurements justify it |
| Consensus members | Primary key `(consensus_state_id, contributor_id)` |
| Consensus members | `(selected_observation_id)` for reverse lineage |
| Reporting events | `(fiscal_period_id, available_at DESC)` |
| Coverage assertions | `(provider_profile_id, security_id, coverage_kind, available_at DESC)` |

Use explicit non-null enum sentinels for annual period and nonapplicable dimensions. If optional fields participate in uniqueness, use PostgreSQL 16 `NULLS NOT DISTINCT` or a properly scoped partial unique index. Default SQL null uniqueness would permit duplicates. [PostgreSQL 16 constraints](https://www.postgresql.org/docs/16/ddl-constraints.html)

### State persistence

Create a state when membership, selected observation/version, eligibility, status or value changes—even if the rounded mean is unchanged.

Do not create a state for an identical retransmission.

Persist states, members and exclusions atomically. Serialize competing writers per dataset/series and availability boundary.

Example:

```text
Jan 2  → state A
Jan 8  → state B
Jan 27 → state C

Jan 20 query → state B
```

No daily expansion is required. Backtests use indexed as-of lookups or sweep ordered state boundaries.

A reporting rollover does not require copying absolute-target consensus. It changes the horizon mapping.

### Scale strategy

- Begin with indexed, unpartitioned tables and representative query-plan tests.
- Recompute only affected series.
- Preserve raw payloads outside relational rows through the existing raw store.
- Store selected memberships, not copies of every historical candidate per state.
- Use compact exclusion summaries plus relevant references.
- Batch historical replay by security/series.
- Consider partitioning only after measured table/index growth warrants it.

Partitioning must preserve global event deduplication and foreign-key strategy; PostgreSQL partitioned uniqueness has partition-key restrictions. [PostgreSQL 16 table constraints](https://www.postgresql.org/docs/16/sql-createtable.html)

No Kafka, Spark, Redis, Airflow or Kubernetes is needed.

V1 has no arbitrary age-based estimate expiry. An estimate remains active until a documented update, withdrawal or eligibility change. Preserve estimate age and require source coverage; provider-specific expiry policies must be explicit, versioned semantics.

## O. Test Matrix

Tests below are implementation acceptance requirements, not work performed in this review.

| Area | Acceptance test |
|---|---|
| Future exclusion | An event available one microsecond after T cannot affect state, counts or identity. |
| Boundary inclusion | An event at exactly T is visible. |
| Same analyst revision | New value replaces its active predecessor; contributor count stays constant. |
| Duplicate transmission | Repeated event/version produces no extra observation, state, vote or count. |
| Event ID conflict | Same identity/version with different payload is flagged; no silent overwrite. |
| New initiation | Contributor count rises; revision count and breadth exclude the initiation. |
| Re-entry | A post-withdrawal NEW starts a new episode; it is not compared with the withdrawn estimate. |
| Withdrawal | Removes the contributor only from its specified scope and only from its availability. |
| Withdrawal threshold | Three contributors becoming two yields a missing consensus state. |
| Correction | Old state is visible before correction; corrected state afterward; no new economic revision count. |
| Non-head correction | Correcting an older event does not displace a later revision. |
| Correction cycle | Rejected deterministically. |
| Correction target move | Old/new target membership changes at correction availability, without a new economic initiation. |
| Zero EPS | Zero contributes normally; `0→0` scaled change is zero. |
| Negative EPS | `−1→−0.5` produces positive absolute/scaled change. |
| Loss→profit | `−0.2→0.2` scaled change is `+2`. |
| Profit→loss | `0.2→−0.2` scaled change is `−2`. |
| Tiny denominator | `0.01→0.10` scaled change is `0.9`, not `9`. |
| Positive revenue | `100→110` yields `0.10`. |
| Zero revenue | Zero old revenue produces `DENOMINATOR_UNSUPPORTED`. |
| Nonfinite values | NaN/infinity never enter valid consensus. |
| Contributor threshold | Two contributors missing; exactly three valid. |
| Sample dispersion | `[1,2,3]` produces mean/median `2`, sample standard deviation `1`. |
| EPS basis | GAAP, adjusted and normalized observations cannot mix. |
| EPS share type | Basic and diluted cannot mix. |
| Unknown basis | Stored but excluded from standard consensus. |
| Revenue basis | Reported, organic, constant-currency and segment cannot mix. |
| Currency | EUR cannot enter a USD reporting-currency series. |
| Unit scaling | Explicit millions→base-unit conversion is exact; unknown scale is excluded. |
| Share basis | Split/ADR mismatch blocks comparison; no hidden adjustment. |
| Fiscal ambiguity | Conflicting labels return ambiguous target, never a month-derived answer. |
| January year-end | Explicit FY2027 Q1 mapping survives calendar-year differences. |
| Quarter rollover | Q2 becomes FQ1 exactly when Q1 reporting is known. |
| FY rollover | Annual transition independently maps FY2→FY1. |
| Identity invariant | The same fiscal-period/target IDs remain unchanged while FQ/FY classification changes. |
| Missing intermediate estimates | No compression of horizons when FQ1 has no estimates. |
| Missing actual metadata | Unknown reporting status yields unresolved horizon. |
| After-hours rollover | Pre-release close excludes the event; post-release snapshot includes it. |
| Amendment | Does not move the original known reporting boundary. |
| Date-only timing | Source-local day boundary, DST and unknown-timezone fallback tested. |
| Contributor aliases | Renumbering/spelling changes do not double-count after alias evidence becomes known. |
| Alias PIT | Future identity resolution does not alter earlier snapshots. |
| Delayed event | Older economic sequence arriving later does not replace newer state. |
| Ordering conflict | Conflicting same-precedence records are not resolved by database ID. |
| Historical reconstruction | Event replay and persisted as-of state return identical members and statistics. |
| Lookback identity | Post-rollover FQ1 compares the same target that was formerly FQ2. |
| Missing prior state | No substitution of another target or older valid state. |
| Breadth | One vote per contributor’s latest comparable update; initiations/corrections excluded. |
| Revision count | Multiple true changes by one contributor count separately. |
| Window boundary | Exactly `T−30d` excluded from activity; exactly T included. |
| Complete empty window | Count is zero only with complete coverage. |
| Incomplete feed | Missing coverage yields missing activity, not zero. |
| Provenance | Reconstruct every selected member and exclusion from raw source references. |
| Raw immutability | Two different responses in one second cannot overwrite bytes. |
| Idempotency | Repeated normalization/replay yields identical fingerprints and row identities. |
| Atomicity | Concurrent writers cannot expose partially written consensus membership. |
| Dataset pinning | New historical import cannot silently alter an older published build. |
| Feature ageing | Features recompute correctly as lookback boundaries move without new estimates. |
| Legacy isolation | Fundamental runner cannot select or write estimate registry entries. |
| Fresh PostgreSQL migration | Future migration chain builds from base; constraints/indexes match models; no manual schema repair. |
| Frozen regression suites | Existing 81-unit/83-full baseline remains passing, plus new estimate tests. |

## P. Worked Example

**Fictional company: Helios Compute Semiconductor.**

Explicit fiscal metadata:

```text
FY2027 Q1 ends April 30, 2026
FY2027 Q2 ends July 31, 2026
FY2027 ends January 31, 2027
```

Provider profile:

```text
One qualified provider
Four identified analysts: A, B, C, D
Adjusted diluted EPS
Reported consolidated revenue
USD
Consistent share basis
Complete event/reporting coverage
```

Revenue below is displayed in USD millions; canonical storage uses USD base units.

### Initial state

On March 1, A/B/C initiate both metrics for both targets:

| Analyst | Q1 EPS | Q1 revenue | Q2 EPS | Q2 revenue |
|---|---:|---:|---:|---:|
| A | 0.80 | 100 | 1.20 | 120 |
| B | 1.00 | 110 | 1.40 | 140 |
| C | 1.20 | 120 | 1.60 | 160 |

Prior-year reporting and explicit fiscal sequence establish:

```text
FY2027 Q1 = FQ1
FY2027 Q2 = FQ2
```

### Event ledger

All times are verified UTC availability times.

| Time, 2026 | Event |
|---|---|
| Apr 20, 12:00 | **T1** |
| Apr 25, 12:00 | A revises Q1 EPS `0.80→1.10`; revenue `100→110`. |
| Apr 27, 12:00 | B revises Q1 EPS `1.00→0.90`; revenue `110→108`. |
| May 1, 12:00 | D initiates Q1 EPS `1.20`, revenue `120`. |
| May 5, 12:00 | C withdraws Q1 EPS and revenue; Q2 estimates remain active. |
| May 8, 12:00 | Provider corrects A’s Apr 25 Q1 EPS from `1.10` to `1.00`. |
| May 9, 12:00 | A revises Q2 EPS `1.20→1.50`; revenue `120→150`. |
| May 10, 12:00 | **T2** |
| May 20, 20:05 | Q1 actual results released: **16:05 New York time**. |
| May 21, 09:00 | B revises Q2 EPS `1.40→1.60`; revenue `140→160`. |
| May 21, 12:00 | **T3** |

Each metric is a separate observation/event even when delivered in the same report.

### Selected active estimates

| Snapshot | Target | EPS selections | Revenue selections |
|---|---|---|---|
| T1 | Q1 | A `.80`, B `1.00`, C `1.20` | A `100`, B `110`, C `120` |
| T1 | Q2 | A `1.20`, B `1.40`, C `1.60` | A `120`, B `140`, C `160` |
| T2 | Q1 | A `1.00` corrected, B `.90`, D `1.20` | A `110`, B `108`, D `120` |
| T2 | Q2 | A `1.50`, B `1.40`, C `1.60` | A `150`, B `140`, C `160` |
| Release | Q1 | Same as T2 | Same as T2 |
| Release | Q2 | Same as T2 | Same as T2 |
| T3 | Q1 | Same as T2, historical | Same as T2, historical |
| T3 | Q2 | A `1.50`, B `1.60`, C `1.60` | A `150`, B `160`, C `160` |

### Consensus statistics

All rows have three contributors and status `VALID`.

| Snapshot/target | Metric | Mean | Median | Low | High | Sample stddev |
|---|---|---:|---:|---:|---:|---:|
| T1 Q1 | EPS | 1.000000 | 1.00 | .80 | 1.20 | .200000 |
| T1 Q1 | Revenue | 110.000000 | 110 | 100 | 120 | 10.000000 |
| T1 Q2 | EPS | 1.400000 | 1.40 | 1.20 | 1.60 | .200000 |
| T1 Q2 | Revenue | 140.000000 | 140 | 120 | 160 | 20.000000 |
| T2 Q1 | EPS | 1.033333 | 1.00 | .90 | 1.20 | .152753 |
| T2 Q1 | Revenue | 112.666667 | 110 | 108 | 120 | 6.429101 |
| T2 Q2 | EPS | 1.500000 | 1.50 | 1.40 | 1.60 | .100000 |
| T2 Q2 | Revenue | 150.000000 | 150 | 140 | 160 | 10.000000 |
| T3 Q2 | EPS | 1.566667 | 1.60 | 1.50 | 1.60 | .057735 |
| T3 Q2 | Revenue | 156.666667 | 160 | 150 | 160 | 5.773503 |

### T1

At April 20:

- Q1 is FQ1; Q2 is FQ2.
- The 30-day cutoff is March 21.
- Initial consensus already exists at the cutoff.
- Both targets’ 30-day consensus changes are zero.
- Revision counts are zero.
- Breadth is `MISSING / NO_DIRECTIONAL_REVISIONS`.

### T2

At May 10:

- Q1 remains FQ1.
- Q2 remains FQ2.
- Thirty-day cutoff: April 10.

For Q1 EPS:

\[
\Delta=1.033333-1.000000=0.033333
\]

\[
S=0.033333/1.033333=0.032258
\]

For Q1 revenue:

\[
R=112.666667/110-1=0.024242
\]

Economic activity, separately for each metric:

```text
A: UP
B: DOWN
D: NEW — excluded from breadth
C: WITHDRAWAL — excluded from breadth
Provider correction: excluded from breadth/count

U = 1
D = 1
breadth = 0
revision_count_30d = 2
initiation_count_30d = 1
withdrawal_count_30d = 1
```

Q2 has one upward revision per metric:

```text
breadth = +1
revision_count_30d = 1
EPS absolute change = +0.10
EPS scaled change = 0.10 / 1.50 = 0.066667
Revenue percentage change = 150 / 140 − 1 = 0.071429
```

### Correction PIT proof

Immediately before May 8:

```text
Q1 EPS active: A 1.10, B 0.90, D 1.20
mean = 1.066667
```

From May 8:

```text
Q1 EPS active: A 1.00, B 0.90, D 1.20
mean = 1.033333
```

The original A `1.10` observation remains stored and selected for earlier snapshots. The correction creates a new consensus state but no additional economic revision.

### Earnings release

At May 20, 20:00 UTC:

```text
Q1 = FQ1
Q2 = FQ2
```

At May 20, 20:05 UTC:

```text
Q1 = HISTORICAL
Q2 = FQ1
```

No estimate changes at the release instant.

Q2’s target ID, selected estimates and consensus state remain identical. Only the horizon mapping changes.

### T3

At May 21, Q2 is FQ1.

Its 30-day cutoff is April 21. The historical comparison is **Q2’s** April 21 state, when Q2 was FQ2.

EPS:

\[
\Delta=1.566667-1.400000=0.166667
\]

\[
S=0.166667/1.566667=0.106383
\]

Revenue:

\[
R=156.666667/140-1=0.119048
\]

For each metric:

```text
A: UP
B: UP
C: no economic update in window

breadth = +1
revision_count_30d = 2
```

The system must not compare T3 Q2 EPS `1.566667` with prior Q1 EPS `1.000000`.

Q1 remains available for historical analysis with its corrected as-known state. Reporting does not delete or retarget its observations.

## Q. Open Decisions

No core architectural decision remains unresolved.

The following require external evidence before enabling a production connector:

1. **Provider selection and licensing:** qualifying historical analyst-event coverage, correction versions, withdrawals and retention rights.
2. **Provider-specific contributor semantics:** individual analyst versus house/team forecast, stable identifiers and alias support.
3. **Provider-specific timing and basis evidence:** timezone, dissemination meaning, adjusted-EPS methodology, share basis and reporting-currency normalization.
4. **Actual-reporting coverage source:** whether existing SEC evidence alone establishes the required coverage or must be supplemented with archived structured releases.

These are connector qualification gates. Missing evidence produces explicit missing outputs; it does not permit weaker historical reconstruction.

## R. Recommendation

`SAFE TO IMPLEMENT PHASE 2.5`

PHASE 2.5 FINAL ARCHITECTURE REVISION COMPLETE
