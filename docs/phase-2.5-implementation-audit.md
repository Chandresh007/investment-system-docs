# Phase 2.5 implementation audit — checkpoint 2.5.8

Audit date: 2026-09-21. Starting master/origin:
`15055cf719d450ab1fc1c28421660f8e5b2190af`, clean after fetch and ff-only pull.
Baseline: Python 3.12.14, local package import; **287 focused / 370 full tests**,
**19 / 27 warnings**, zero failures. Sources: README, complete PROJECT_STATE and
Phase 2.5 contract, implementation, PostgreSQL migration and tests, Git history.

Final result: **PASS after corrections below**. Phase 2.5 IMPLEMENTATION COMPLETE;
READY FOR INDEPENDENT REVIEW; NOT YET FROZEN. Phase 2.6 NOT STARTED.
This is implementation QC, not the independent architecture freeze decision.

**Freeze-review addendum, 2026-09-22:** the subsequent independent review approved
the freeze after adding mandatory catalog-backed provider admission at the
FeatureValue publication boundary. The runner now requires a qualification whose
provider/version match the profile, whose catalog contract matches exactly, and
whose coverage contains the full requested lookback. Its admission is persisted in
feature provenance. See
[PHASE_2_5_AND_FOUNDATION_FREEZE_REVIEW.md](PHASE_2_5_AND_FOUNDATION_FREEZE_REVIEW.md).

## Evidence map

Paths below are relative to `src/investment_research/` unless otherwise stated.
`resolvers`, `consensus`, `revisions`, `registry`, and `runner` mean the modules
under `research/estimates/`. Tests are under `tests/estimates/`.

| Audit | Result and concrete evidence |
|---|---|
| 1 PIT | `EvidenceResolver.visible` filters dataset and `available_at <= T`, including its replacement subquery; profile availability gates evidence. `EstimateEventResolver.observations` joins build membership and filters security/T. Corrections/withdrawals share this input. State repository filters `state_available_at <= T`. Activity reconstructs transitions at original availability. Future evidence and exact microseconds are tested. |
| 2 Absolute identity | `EstimateFiscalPeriod` unique key is company/calendar regime/year/type/quarter-or-FY, all nonnull. Period end lives on evidence. No month/quarter/duration inference exists in estimates normalization/resolvers. Relative labels occur only in requests/feature metadata. |
| 3 Horizons | `ForwardHorizonResolver.resolve` allows FQ1–FQ4/FY1–FY2, starts from a verified reported anchor, walks reciprocal explicit links and verifies unreported status. Regime changes require explicit links, not year arithmetic. Quarterly/annual chains are independent. Missing links/coverage are unresolved; no estimate-presence filtering or horizon compression. Rollover/after-hours/amendment tests inspected. |
| 4 Reporting | `project_filing` requires archived primary context, matching company/accession/fy/fp and known filing/fact times; amendments and comparative facts cannot establish first reporting. `resolve` uses explicit events or coverage extending back to evidenced period start. Conflicting branches/cycles now return UNKNOWN, never coverage-derived absence. |
| 5 Contributors | Stable provider analyst ID precedes evidenced broker-ID/full-name fallback. Only Unicode/case/whitespace normalization; no fuzzy merge. Alias effective interval and knowledge time both apply. Unresolved identities excluded; mergers select one contributor. Renumbering, broker spelling, collisions and future aliases tested. |
| 6 Event identity | Normalizer keys profile/event/version/normalization version, hashes semantic payload, and rejects same-version different content with EVENT_ID_CONFLICT. Database unique event/dedupe keys prevent duplicates; transport retries do not create extra economic events. |
| 7 Semantics | Replay NEW starts an episode; REVISION updates it; exact-scope WITHDRAWAL ends it; CORRECTION changes the root event representation. Re-entry NEW gets a new episode. Orphan revisions remain unresolved. |
| 8 Corrections | `correct` validates parent/stream/profile/security/logical key, monotonic availability, branches/cycles and withdrawal replacement payload. Root event kind/order is retained. Tests cover non-head corrections, target moves, retractions and corrected withdrawals. Old versions remain visible historically; corrections never count/vote. |
| 9 Ordering | Causal ancestry precedes documented provider sequence, then documented publication order. Availability is visibility only. Equal-precedence conflicts produce EVENT_ORDER_AMBIGUOUS exclusions; storage IDs never decide economic precedence. Delayed non-head arrivals do not displace current states. |
| 10 Series identity | Nonnull unique series dimensions include profile/security/absolute target/metric/bases/methodology/currency/unit/EPS share type/share basis/scope. Dataset and calculation version additionally identify states. Eligibility validates exact dimensions and PIT reporting currency. No provider, GAAP/adjusted, basic/diluted, reported/organic or FX pooling. |
| 11 Active selection | Replay chooses latest economic head before eligibility; unusable/missing-target/identity/basis heads cannot fall back to old valid events. Contributor-key grouping and member PK enforce one active selection. Adversarial no-fallback tests inspected. |
| 12 Consensus math | At least 3 contributors; otherwise MISSING / INSUFFICIENT_CONTRIBUTORS, null aggregates, preserved members/exclusions. Decimal precision 38, HALF_EVEN; equal-weight mean/median/high/low/sample stddev (n−1)/count. `[1,2,3]` yields mean/median 2, stddev 1. NUMERIC storage has no imposed scale; zero is valid. |
| 13 State persistence | Fingerprint includes build/profile/manifest/policy, members, observation versions/hashes, episodes, aliases/fiscal evidence, exclusions, status and values. Writer publishes changed event/evidence boundaries only. Same-value membership/version changes produce states; identical retries do not. No daily materialization. |
| 14 Atomicity/concurrency | Writer savepoint encloses state/members/exclusions. Dataset FOR UPDATE lock serializes ingestion/sealing/writers; unique dataset/series/version/boundary is final guard. Real concurrent PostgreSQL test proves one state and no uncommitted partial membership; injected child failures roll back all output. |
| 15 As-of repository | Exact series ID or all dimensions required; constructor pins dataset/profile; version filter mandatory in query (default consensus_v1). Descending state boundary with limit 1, bounded by T. Missing/invalid latest states remain selected. History is append-only. |
| 16 Same-target lookback | `_request` resolves horizon once at T. `consensus_change` queries that same series at T and cutoff. Runner also resolves once per horizon per batch. Helios T3 Q2 is compared with April 21 Q2, then classified FQ2, never prior Q1. |
| 17 Windows | UTC `timedelta(days=7|30|90)`, not trading sessions. Activity explicitly checks `cutoff < available_at <= T`. Cutoff/T/future microsecond tests inspected. |
| 18 Prior semantics | Latest prior state <= cutoff is used even if unusable: NO_PRIOR_CONSENSUS for no row; NO_VALID_PRIOR_CONSENSUS for an unusable latest row, preserving status/reason evidence. No backward search for convenient valid values. |
| 19 EPS | Absolute `new-old`. Scaled zero for both zero, otherwise `(new-old)/max(abs(old),abs(new))`. No floor or naive percentage. All seven specified numeric cases are independently asserted in `test_eps_exact_math`. |
| 20 Revenue | `(new-old)/old` only for old>0/new>=0; old=0 INVALID / DENOMINATOR_UNSUPPORTED, negative values INVALID / INVALID_NUMERIC_VALUE. No dollar floor or clipping. |
| 21 Breadth | `(U-D)/(U+D)`, one latest comparable update per resolved contributor; latest UNCHANGED removes direction. NEW/WITHDRAWAL/CORRECTION do not vote. Withdrawals do not erase earlier revisions. No directional denominator gives MISSING / NO_DIRECTIONAL_REVISIONS. |
| 22 Count | Counts distinct UP/DOWN economic events, not analysts. Multiple updates by one analyst count separately. Duplicates, NEW, reaffirmations, corrections and withdrawals excluded. |
| 23 Coverage | Qualified event semantics plus visible complete assertions must cover whole window, exact security/target/metric/scope; gaps/incomplete/snapshot assertions prevent zero. Future coverage cannot repair past outputs. Conflicting evidence remains incomplete. |
| 24 Registry | Exact 15 contract IDs checked against independent EXPECTED mapping in `test_feature_runner.py`; no persisted raw means or extra FQ/FY permutations. Entries remain inactive to legacy scans. |
| 25 Runner | Explicit profile/build/allowlist, research T, horizon resolution and exact series; delegates math/activity to existing calculator. Status/reason, evidence, atomic batch and deterministic reruns inspected/tested. |
| 26 Feature identity | `estimates_v1:<profile_id>:<dataset_id>` plus existing unique security/feature/research-date/T/version key. Separate providers/builds cannot collide; calculation meaning does not use random IDs. |
| 27 Snapshot semantics | `period_end` is UTC research date, `available_at=T`, actual execution time in `calculated_at`; absolute target/horizon in provenance. Legacy UTC-naive columns are explicitly normalized. Non-UTC database roundtrip tested. |
| 28 Ageing | Exact-T retrieval only. Later T recalculates cutoffs/activity/horizon without new observations. Helios revisions age out exactly on June 8/20; cutoff-ageing and rollover tests also inspected. |
| 29 Provenance | Real persisted T3 scaled EPS feature is reloaded from PostgreSQL and walked to definition/profile/build/target/horizon/current/prior states/members/observations/raw bytes/aliases/fiscal/reporting/coverage evidence. Extended assertions below. No missing logical link found; representation caveat below. |
| 30 Idempotency | Ingestion/normalization use immutable events and build membership; writer and runner compare immutable published fingerprints/content. Full lifecycle rerun compares complete row fingerprints. IngestionRun intentionally records every attempt. |
| 31 Build pinning | Explicit membership prevents later global observations entering old builds. Evidence is dataset-owned; input triggers lock and reject additions after sealing. Conflicting published state/feature reruns require new build. Backfill A/B test preserves original IDs/values/provenance. |
| 32 Raw storage | Request/provider hash + payload SHA-256 in path prevents same-second different-payload overwrites. Receipt reuse includes endpoint, request, provider, bytes, receipt timestamp and metadata; later receipts stay distinct. Retry verifies original bytes and refuses corruption. Original raw-row fix reviewed, unchanged. |
| 33 Migration | `25e001`, parent `1f4b7c9d2a31`. Disposable verifier runs base→head→prior→head and estimates suite, 15/0/15 tables. Persistent database never downgraded. Alembic metadata comparison restricted to estimates found no differences. |
| 34 Constraints/indexes | Reviewed observation security/metric/target/T, contributor/target/metric/T, logical event/T, fiscal/reporting/coverage indexes. State unique index supplies backward as-of lookup. Complete series/event/member/state/feature identity keys are nonnull; annual FY/nonapplicable sentinels prevent NULL uniqueness holes. Optional evidence fields are not relied on as unique keys. |
| 35 Frozen isolation | `git diff 00fa6b0..HEAD -- src/investment_research/research ':!src/investment_research/research/estimates'` is empty. All new research files were read. Shared historical changes are only model imports and optional raw receipt timestamp support; additive INVALID enum repair was reviewed. Fundamental/market actual-loop exclusion tests pass. |
| 36 Status/reasons | Null checked explicitly; valid zero survives normalization/math/persistence. Invalid selected observations remain explicit exclusions; insufficient consensus remains MISSING. Revision/runner preserve current and prior failures. Deterministic contributor sorting/exclusion reasons and ordered request/coverage/formula evaluation inspected. |
| 37 Scale sanity | Fixed dataset-wide evidence materialization and unrelated alias replay boundaries, with failing regressions first. Observations remain security/build scoped; as-of states indexed; raw payloads external; no per-day expansion. Whole-security history/prefix replay, pairwise ordering and dataset-wide writer serialization remain throughput limitations, not qualified for tens of millions of rows. No new infrastructure. |
| 38 Integration | Helios provider→filesystem raw→normalization→ingestion→resolvers→replay→consensus→revision→runner→FeatureValue uses production services and PostgreSQL. Only transport/reference/evidence prerequisites are synthetic; no final-state inserts. Separate unit tests inject states solely to test failure handling. |
| 39 Orchestration | **A: explicit, acceptable staged architecture.** Existing runner module docstring and 2.5.5–2.5.7 handoffs already require consensus through T. Operational recipe below makes this visible from README. Reads do not prove publication completeness; caller must honor ordering. |
| 40 Provider limitation | Core interface/profile is provider-neutral; synthetic only. Production qualification remains required for historical events, stable contributors, withdrawals/corrections, PIT dissemination, adjusted EPS methodology/share basis, coverage and licensing/retention rights. No weakened core semantics. |

## Corrections made during this audit

1. **Reporting ambiguity incorrectly became absence.** `active_evidence` returned
   `[]` on conflicting branches; a cycle also removed every row. Reporting then
   used complete coverage to return NOT_REPORTED_AS_OF_T. Reproduced both cases
   against PostgreSQL before fixing. Evidence conflicts now raise an internal
   explicit signal; reporting returns UNKNOWN with evidence IDs. Other resolvers
   conservatively exclude unresolved evidence. Tests:
   `test_conflicting_reporting_versions_never_prove_not_reported`,
   `test_reporting_supersession_cycle_is_unknown_not_absence`.
2. **Unrelated evidence affected correctness and replay work.** Dataset-wide
   evidence scans let another alias's branch conflict erase a valid contributor,
   loaded unrelated companies/periods, and replayed unrelated alias boundaries.
   Three failing regressions proved those effects. SQL now scopes fiscal evidence
   by company, aliases by key, reporting by target, coverage by security, and
   publication boundaries by relevant keys/company. The connected PIT-visible
   supersession component is retained even when
   keys move out of scope, preventing old evidence from resurrecting or sibling
   conflicts from being hidden. Recursive UNION terminates cyclic graphs. Tests:
   `test_unrelated_alias_conflict_does_not_disable_resolved_contributor`,
   `test_fiscal_lookup_does_not_materialize_other_companies`,
   `test_publication_ignores_unrelated_alias_boundaries`; additional PIT cross-key
   and target-moving replacement regressions, including an out-of-scope parent
   with conflicting children, protect the scoping change.

No approved financial formula, canonical identity, frozen phase, model, or
migration was changed. Existing assertions were not weakened. Existing affected
published outputs must be reconstructed in a new dataset; immutability/conflict
checks remain in place. Audit commit locator:
`git log --format='%H' --grep='^Complete Phase 2.5 implementation audit$' -1`.

## Persisted provenance proof

`test_full_helios_lifecycle` reloads the T3 (2026-05-21 12:00 UTC)
`eps_adjusted_diluted_consensus_change_scaled_30d_fq1` FeatureValue. Its value is
`0.10638297872340425531914893617021276598`. The same build's Q2 mean is `4.70/3`;
the April 21 prior Q2 mean is `1.40`. Absolute Q2 identity survives FQ2→FQ1 rollover.

The audit extends that existing test to check the stored ResearchFeature metadata
against the feature's definition, the build/profile link, the requested FQ1, and
every referenced horizon fiscal/reporting/coverage row and its hash-verified raw
object. Both consensus state cutoffs, three members per state, observation versions,
contributor/alias/fiscal IDs, build membership and source record locators are
checked. The primary reporting event links to the supporting FinancialFact.
The full rerun retains row identities and content. All test rows roll back; this
is a persisted PostgreSQL proof, not a claim that synthetic data was deployed.

**Representation for independent review:** the implementation uses structured
IDs in existing `FeatureValue.provenance`, with relational state/member/observation
lineage, rather than the separate `estimate_feature_evidence` relation proposed
in spec §C. This choice was already documented in checkpoint 2.5.6. No logical
link is missing in the persisted trace, but feature-to-evidence IDs lack their
own foreign keys. The independent reviewer must assess this representation
before freezing; this audit does not silently amend the specification.

## Required orchestration

Prepare a qualified profile, absolute series and complete dataset inputs first.
Seal the build before production publication where possible. Publish all required
series through research T, then calculate features in the same transaction:

```python
from investment_research.research.estimates.consensus import ConsensusStateWriter
from investment_research.research.estimates.runner import EstimateFeatureRunner

# session is the caller-owned transaction; IDs belong to the pinned build/profile.
writer = ConsensusStateWriter(session, dataset_id)
for series_id in required_series_ids:
    writer.replay(series_id, through=research_t)

values = EstimateFeatureRunner(
    session, provider_profile_id=profile_id, dataset_id=dataset_id,
    source_qualification=source_qualification,
    allow_synthetic=False,  # True only for an explicitly synthetic development run.
).calculate_for_security(security_id, research_t)
session.commit()
```

Replay reconstructs all boundaries through T, including prior lookback states.
Do not merely publish the newest state. Do not infer that a state must have its
boundary equal to T: an unchanged valid consensus can correctly be much older.
The feature runner is a reader of published consensus, not an ingestion/replay
orchestrator. Missing or stale publication can produce missing or stale magnitude
features; it is not a source-coverage signal. Callers of the revision calculator
have the same prerequisite. A new requested T still needs feature recalculation
for ageing/horizon changes. Existing snapshot conflicts require a new build.

## Remaining limits and release gate

- No live/commercial provider, historical event contract or actual-reporting feed
  has been qualified. Stable identities, event/correction/withdrawal semantics,
  timestamp/timezone evidence, adjusted methodology, comparable share basis and
  completeness must be established before production use. No internal FX or
  estimate split conversion is implemented.
- Whole-security reconstruction and per-event prefix replay prioritize correctness;
  this audit is not a provider-scale benchmark. Measure representative workloads
  before broad production rollout. Build locks intentionally serialize writers.
- Feature evidence is structured JSON at the FeatureValue boundary, as explained
  above; relational lineage begins at the referenced stored evidence/state rows.
- UTC/SQLAlchemy deprecation warnings remain in existing shared code.

PostgreSQL EXPLAIN of the exact dataset/series/version/T as-of lookup used
`Index Scan Backward using uq_estimate_state_boundary`; this verifies the lookup
index path, not production-scale throughput.

Final validation: **295 focused / 378 full tests**, **19 / 27 warnings**,
**zero failures**. Disposable PostgreSQL base → head → prior → head and all
**295 focused tests** passed; metadata differences: none; current/head: **25e001**.
`git diff --check` passed. See the checkpoint 2.5.8 PROJECT_STATE entry.
Next action: independent Phase 2.5 QC/architecture review. No freeze and no Phase 2.6.
