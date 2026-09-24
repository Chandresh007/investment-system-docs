# Investment Research System — Project State

## PHASE 2.8 FROZEN

**PHASE 2.9 IMPLEMENTATION IN PROGRESS — SPECIFICATION FROZEN**

The approved design is
`docs/phase-2.9-historical-replay-backtest-architecture-review.md`. The
authoritative implementation contract is
`docs/phase-2.9-historical-replay-backtest-spec.md`. Phase 2.9 is not yet
fully implemented or independently frozen; no Phase 2.10 work has started.

### Checkpoint 2.9.8 — Delisting, censoring, and terminal outcomes

Status: **COMPLETE**.

- Added the frozen `terminal_total_return_v1` outcome policy behind the one-way
  outcome-service boundary. A terminal-enabled build must pin the exact policy,
  storage-adapter, canonical-details, cash-holding, stock-conversion,
  zero-recovery, and Decimal versions. The DEVELOPMENT adapter requires the
  exact persisted outcome build, every declared runtime provider, a sealed
  `security_identity` dataset, original action receipts, finite values, and
  provider coverage from actual entry through the outcome-build cutoff.
- Terminal evidence uses a strict versioned `terminal_v1` envelope rather than
  inferring proceeds from generic legacy actions. It admits qualified positive
  cash proceeds; one exact successor and positive conversion ratio; or explicit
  legally evidenced zero recovery. It verifies lifecycle date/status agreement,
  canonical event-version ordering, raw receipt/cutoff integrity, a unique
  source security, and an exact regular effective session. Conflicting qualified
  events, lifecycle overlaps, or incompatible corrections are `DISPUTED` and
  never selected by convenience.
- Implemented cash/stock/zero terminal wealth paths without adjusted-close or
  last-quote shortcuts. Cash proceeds enter on the effective session and remain
  nominal cash at 0% through the horizon. Stock conversions use only the exact
  conversion ratio and successor close on that effective session, then
  liquidate to cash without stitching successor performance. Legally evidenced
  zero recovery produces total return and maximum drawdown `-1`; volatility is
  explicitly unavailable after zero wealth.
- Lifecycle termination, disappearance, or a missing successor observation
  without qualified terminal value is retained as
  `CENSORED/TERMINAL_VALUE_UNKNOWN`, not dropped, carried forward, or assumed to
  be a total loss. Continuous active lifecycle coverage is required across the
  evaluation interval; a gap cannot be hidden by an active identity at the
  horizon. A still-active security with no close in the bounded exit window
  remains the separate `MATURE_MISSING/EXIT_PRICE_UNAVAILABLE` case.
- Terminal discovery is aligned with the security's selected-or-exhausted
  three-session exit window. A lifecycle termination inside that bounded window
  cannot masquerade as an ordinary missing exit. Terminal cash before the
  boundary is held through the intended horizon; a termination during a delayed
  exit window anchors the actual endpoint to its exact session. SPY and PIT
  industry comparisons continue to use that same actual interval.
- Extended the PIT dynamic-industry benchmark to resolve every constituent's
  lifecycle and terminal path for each interval. Qualified terminal returns
  enter the equal-weight basket; unknown or disputed constituent terminal value
  fails the benchmark closed rather than silently constructing a survivor-only
  basket. Terminal-enabled benchmark builds now require a same-build terminal
  source. A zero-wealth basket remains a valid total-loss path with drawdown
  `-1`, not an arithmetic error.
- Terminal outcome quality is independent from prediction quality and is the
  weakest admitted lifecycle/action/price/calendar side. Missing interval
  coverage forces a null censored result even under a development override.
  The legacy provider/build-unaware action and market tables remain explicitly
  **DEVELOPMENT-only**, and all results disclose that no production or
  survivorship-qualified claim is available. Provider rights and comprehensive
  delisting/terminal datasets remain external hard gates.
- Added **13 focused terminal regressions** covering cash acquisition, unknown
  delisting before a horizon and between horizons, delisting inside the bounded
  exit search, legally evidenced bankruptcy zero recovery, stock conversion,
  missing successor valuation, conflicting qualified events, incomplete
  provider coverage, lifecycle gaps, future-known terminal proceeds exclusion,
  survivorship-unqualified provider disclosure, and terminal returns inside the
  PIT industry basket. Market/terminal suite: **25 passed**; combined
  replay/outcome suite: **90 passed**.
- Full regression: **995 passed, 8 existing warnings, zero failures** in 119.94s.
  `git diff --check` and bytecode compilation passed. Alembic remains
  current/head **`p29003`**; checkpoint 2.9.1 already supplied the immutable
  outcome storage, so this calculation/policy checkpoint requires no schema
  change. Frozen Phase 2.2–2.8 formulas, thresholds, and rules are unchanged.
- Next: **2.9.9 — minimal signal-specific fundamental and estimate outcomes**.

### Checkpoint 2.9.7 — Market outcome labels

Status: **COMPLETE**.

- Added the exact immutable code-owned registry for all nine v1 market labels:
  security total/price return, aligned SPY total/excess return, PIT dynamic
  industry total/excess return, maximum drawdown, maximum favorable excursion,
  and forward realized volatility. Definition publication is idempotent with
  full-content conflict verification; outcome builds pin every definition hash
  before calculation.
- Implemented the four canonical `calendar_month_from_entry_v1` horizons (`3m`,
  `6m`, `12m`, `24m`) from the actual entry session with calendar-month
  end-of-month clamping. Exit selection starts at the first regular close on or
  after the boundary and searches only that session plus two scheduled sessions.
  Pending windows remain pending; a completed fixed window without a valid exact
  security close is `MATURE_MISSING/EXIT_PRICE_UNAVAILABLE`.
- Added a deterministic Decimal total-return engine with 80-digit intermediate
  precision, 38-significant-digit half-even output, finite/domain guards, and
  pinned policy identity. It consumes exact raw closes only, applies the frozen
  Phase 2.4 split-continuity and distribution-reinvestment semantics, includes
  entry wealth one, and calculates total return, price return, running-peak
  maximum drawdown, favorable excursion, and annualized sample log-return
  volatility. Volatility requires 20 consecutive intervals and otherwise emits
  an explicit mature-missing label rather than zero.
- Added a separately admitted DEVELOPMENT corporate-action adapter. It requires
  original raw receipts, provider/build/cutoff consistency, positive split
  factors and nonnegative distributions, ignores actions first known after the
  outcome-build cutoff, and never uses provider adjusted close. Terminal actions
  encountered inside a path fail closed with
  `TERMINAL_EVENT_REQUIRES_POLICY`; qualified proceeds/censoring are deliberately
  owned by checkpoint 2.9.8 and are not guessed here.
- SPY uses the same raw-close/action path over the security's exact actual delayed
  entry and exit sessions. It never chooses its own convenient endpoint. Excess
  returns subtract aligned simple total returns, not log returns. Missing
  benchmark evidence makes only the dependent benchmark/excess labels missing;
  it does not substitute a different benchmark.
- Added `pit_dynamic_industry_equal_weight_v1` over the actual sealed Phase 2.6
  classification, primary-eligibility, identity/lifecycle, permanent node, and
  explicit lineage contracts. Membership is reconstructed at every interval's
  start close using only evidence known then; current `Security.is_active` is not
  a selector. V1 follows only uniquely pinned `CONTINUES` lineage, requires at
  least three valid primary constituents, equal-weights their next close-to-close
  simple total returns, and rebalances after the interval. Missing/ambiguous
  lineage, datasets, release evidence, or constituent coverage fails closed.
- Security predictions use the security path for return/risk and the industry
  path as a benchmark. Industry-node predictions use the PIT industry path as
  their subject, including drawdown/MFE/volatility, and compare that path with
  aligned SPY without fabricating constituent-security predictions or security
  entry prices. Calculation remains in the one-way outcome package; it does not
  persist outcomes or expose future data to replay/prediction code. Immutable
  outcome publication and versioned maturity orchestration remain checkpoint
  2.9.10 work.
- The legacy provider/build-unaware `MarketPrice` and `CorporateAction` tables and
  repository calendar remain hard-capped at **DEVELOPMENT**. Every result carries
  their limitations and weakest-side quality; no output is labeled PIT,
  survivorship-qualified, or production-ready. Provider rights, complete
  delisting/terminal coverage, and production calendar/market datasets remain
  external hard gates.
- Added **12 focused tests** covering the exact definition set and manifest hash
  gate; all four horizons and end-of-month clamping; positive and negative
  returns; finite Decimal arithmetic; 20-interval volatility; split neutrality;
  cash distributions; after-close entry; bounded missing exit; delayed endpoint
  alignment; drawdown/MFE; future action exclusion; node-scope semantics; PIT
  interval-start industry membership; future taxonomy correction exclusion; and
  present-day survivor-flag isolation. Market entry/outcome suite: **22 passed**;
  combined replay/outcome suite: **77 passed**.
- Full regression: **982 passed, 8 existing warnings, zero failures** in 115.48s.
  `git diff --check` and bytecode compilation passed. Alembic remains current/head
  **`p29003`**; checkpoint 2.9.1 already supplied the immutable outcome storage,
  so this calculation/definition checkpoint requires no schema change. Frozen
  Phase 2.2–2.8 formulas, thresholds, and rules are unchanged.
- Next: **2.9.8 — delisting, censoring, and terminal outcomes**.

### Checkpoint 2.9.6 — Deterministic entry/execution evaluation policy

Status: **COMPLETE**.

- Added the outcome-side `next_regular_session_close_v1` implementation behind a
  one-way `research.outcomes` service boundary. Replay/prediction code does not
  import the outcome package. Entry evaluation consumes an immutable published
  prediction and future market observations only.
- The eligible session is exactly the first pinned US regular session whose
  official open is strictly later than `decision_ready_at`; the selected entry is
  that session's official close. Pre-open decisions can use that day's close,
  exact-open and after-close decisions move to the following session, and
  weekends, observed holidays, exceptional closures, DST, and early-close times
  follow the same content-addressed calendar used by replay.
- Search is bounded to the intended session plus exactly two subsequent scheduled
  sessions. Only an exact observation at an official session close is eligible.
  Missing/invalid closes advance within that fixed window, with delay persisted in
  the result; stale prior rows and a valid fourth-session row are never carried or
  discovered. Before the remaining window closes the result is `PENDING`; after
  all three closes without valid evidence it is terminal
  `MATURE_MISSING/ENTRY_PRICE_UNAVAILABLE`, including suspended securities.
- Added an explicit qualified-close source contract and a legacy
  `MarketPrice` adapter that is hard-gated to **DEVELOPMENT**. The adapter requires
  a persisted canonical outcome-market build, exact provider and normalization
  pins, the same calendar build as the selector, a structured development
  override, an original raw receipt/content hash, an exact retrieval cutoff, and
  a finite positive provider-reported raw `close`. It reads no approximate or
  stale observation and never reads provider `adjusted_close`; evidence records
  `adjusted_close_used=false`.
- The deterministic `EntrySelection` records intended and searched sessions,
  exact observation/raw receipt/build identity, actual entry date/time/value,
  delay, independent outcome-data quality/admission, propagated provider/calendar
  limitations, and a canonical evidence fingerprint. Evaluation time must not
  precede the published prediction. No entry row is persisted prematurely;
  outcome persistence remains with the later outcome evaluator checkpoints.
- The architecture's production gate remains intact: the legacy
  provider/build-unaware `market_prices` identity and repository calendar can
  support synthetic/development qualification only. They cannot emit PIT,
  survivorship-qualified, or production outcome claims; a future qualified
  provider/build-aware observation adapter can satisfy the same source contract
  without changing entry semantics.
- Added **10 focused tests** covering strict after-close/pre-open/exact-open
  boundaries, weekend/holiday/exceptional closure routing, same-day-close
  rejection, adjusted-close trap, intended-session fallback and delay, pending
  maturity, the exact three-session cap, fourth-session exclusion, stale-price
  rejection, suspended-security retention, future-retrieval exclusion, invalid
  price basis, and development override/build-policy gates. Combined replay plus
  outcome suite: **65 passed**.
- Full regression: **970 passed, 8 existing warnings, zero failures** in 117.30s.
  `git diff --check` and bytecode compilation passed. Alembic remains current/head
  **`p29003`**; this checkpoint adds a deterministic service/result contract and
  requires no schema change. Frozen Phase 2.2–2.8 formulas and rules and prior
  Phase 2.9 artifacts are unchanged.
- Next: **2.9.7 — market outcome labels at 3m/6m/12m/24m**.

### Checkpoint 2.9.5 — Immutable prediction ledger publication

Status: **COMPLETE**.

- Added a prediction-side `PredictionLedger` that consumes only the immutable
  snapshot/`SignalResult` episode derivation from checkpoint 2.9.4. It publishes
  exactly one `FIRED_START` row for each immediately consecutive
  `VALID/false -> VALID/true` transition and rejects any stored prediction outside
  the derived episode-start population. Continued true, true -> false, false ->
  false, warm-up, left-censored, missing, invalid, and excluded observations do
  not publish predictions.
- Added `ResearchPredictionPublisher`, which composes snapshot publication and
  prediction publication inside the runner-owned date transaction. Checkpoint
  prediction counts and the combined snapshot/prediction artifact hash are
  sealed together; complete-checkpoint verification recomputes both populations
  without inserting or repairing history.
- Every prediction references the exact replay run/checkpoint, research snapshot,
  current and prior `SignalResult`, exact `ResearchSignal` ID/version/hash, signal
  and upstream factor builds, permanent subject, T/effective/decision timestamps,
  and origin. Strength, severity, condition coverage/state, and signal quality
  are copied from the fired result and remain protected by the existing
  PostgreSQL insert guard and insert-only immutability trigger.
- Implemented the frozen `phase_2_8_signal_objectives_v1` mapping for all nine
  Phase 2.8 signal definitions. Each prediction seals a canonical, hashed payload
  containing its objective family and predeclared primary/secondary outcome
  registry keys and target horizons before any outcome is evaluated. A new or
  unknown signal version is a separate series and fails closed until a new
  objective-policy mapping is explicitly pinned; it cannot silently inherit v1.
- Prediction quality is the weakest checkpoint/snapshot/current-signal research
  side, with artifact IDs and input levels captured in immutable quality
  provenance. Future evidence is absent from the module and cannot improve that
  level. Semantic fingerprints include the exact derived episode transition and
  all prediction content, while `published_at` remains a separate real
  publication timestamp and is never allowed to precede T.
- Publication is conflict-safe and idempotent: insert-conflict-do-nothing is
  followed by full-content comparison, one current `SignalResult` remains unique,
  an exact repeat reuses the same row and publication hash, and changed replay/
  signal builds produce distinct predictions. A conflicting row requires a new
  build/run rather than mutation.
- Added **10 focused/integration cases** covering the complete objective registry,
  one prediction per episode, exact copied lineage/evidence, repeat idempotency,
  weakest-side quality, non-start transitions, a separate replay build, an
  unmapped new signal version, policy mismatch, missing/invalid exclusion, atomic
  runner counts (`0, 1, 1`), complete-run verification, and no duplicate rows.
  Combined replay suite: **55 passed**.
- Full regression: **960 passed, 8 existing warnings, zero failures** in 113.93s.
  `git diff --check` and bytecode compilation passed. Alembic remains current/head
  **`p29003`** because checkpoint 2.9.1 already supplied the prediction schema and
  immutable database guards; no model or migration change was required. Frozen
  Phase 2.2–2.8 formulas, signal thresholds/rules, and prior Phase 2.9 artifacts
  are unchanged.
- Next: **2.9.6 — deterministic entry/execution evaluation policy**.

### Checkpoint 2.9.4 — Exact v1 signal-episode detection

Status: **COMPLETE**.

- Added `SignalEpisodeDetector` and a pure frozen transition policy for
  `valid_false_to_true_weekly_v1`. It derives state only from immutable snapshots,
  exact `SignalResult` IDs, and the immediately preceding checkpoint in the same
  replay run. It creates no mutable `signal_episodes` table and imports or queries
  no outcome/backtest service.
- Exactly `VALID/false → VALID/true` yields `EPISODE_START`. Repeated true is
  continuation, true → false closes the observable episode by derivation, and a
  later false → true starts another episode. Strength/severity are intentionally
  absent from the policy, so there is no cooldown, hysteresis, strength restart,
  or other unfrozen threshold.
- Initial/warm-up true, new-subject true, new signal-version true, and true after
  `MISSING`, `INVALID`, an excluded prior subject, a missing/incomplete prior
  checkpoint, or a seven-day schedule discontinuity are explicitly
  `LEFT_CENSORED` and never episode starts. A valid false after any gap re-arms
  the series for a later immediately consecutive true observation.
- Security and industry-node permanent identities are distinct; node continuity
  follows the permanent industry node rather than a date-specific peer-group ID.
  Signal series use the exact immutable `research_signal_id`, so versions and two
  signals on one security never cross. Current/prior snapshot-result lineage is
  revalidated for subject, T, effective date, origin, signal build, upstream
  factor build, and node peer group before a transition is classified.
- Detection accepts a `RUNNING` current checkpoint only for use inside the
  runner's atomic date publication, or an already `COMPLETE` checkpoint for
  deterministic verification. The predecessor must be complete and exactly one
  scheduled ordinal/week earlier. Decisions carry exact current/prior
  checkpoint, snapshot, and signal-result IDs plus a deterministic transition
  fingerprint; repeated derivation is identical and writes nothing.
- Added **19 tests**: the complete transition/left-censor matrix, false → true →
  true → false → true restart sequence, warm-up rules, missing/invalid/excluded
  prior states, missing/incomplete/gapped checkpoints, absent signal version,
  two independent signals on one security, deterministic repeated detection,
  future-checkpoint isolation, and a production replay/snapshot integration that
  emits one first-requested-date start then suppresses its continuation.
  Combined replay suite: **45 passed**.
- Full regression: **950 passed, 8 existing warnings, zero failures** in 114.21s.
  `git diff --check` passed. Alembic remains current/head **`p29003`** with no
  model or migration change. Frozen Phase 2.2–2.8 formulas, thresholds, signal
  rules, and Phase 2.9 snapshot rows are unchanged.
- Next: **2.9.5 — immutable prediction ledger publication at episode starts**.

### Checkpoint 2.9.3 — Immutable full-population research snapshots

Status: **COMPLETE**.

- Added `ResearchSnapshotPublisher`, the prediction-side date publisher that
  creates one immutable snapshot for every sealed-universe candidate and every
  exact evaluated peer-group node. Eligible subjects reference the complete
  ordered requested `FactorValue` and `SignalResult` populations; excluded
  candidates are retained with their stable exclusion reason and intentionally
  empty artifact sets. Snapshot rows copy historical identity/classification
  context but reference upstream factor/signal lineage by IDs and independent
  ordered-set hashes instead of duplicating component or condition graphs.
  Publication uses bounded multi-row inserts and one checkpoint-population read,
  avoiding date × subject insert/select loops.
- Snapshot lookup is exact on subject, T, effective date, origin, signal build,
  upstream factor build, peer policy/group where applicable, and frozen registered
  definition/version/hash pins. Other-T, future, other-subject, missing, and
  wrong-build rows are never cache hits. `VALID/true`, `VALID/false`, `MISSING`,
  and `INVALID` signal results are all retained as authoritative observations;
  missing/invalid does not mean an absent snapshot.
- Each fingerprint covers the complete canonical snapshot content, including
  run/checkpoint/build identity, PIT evidence references, ordered artifact IDs,
  counts, quality/admission, limitations, warning/contradiction counts, and the
  historical decision anchor (`decision_ready_at = T`). Exact reruns use
  conflict-safe insert plus full content and full checkpoint-population
  verification. Any changed content requires a new run/build.
- Replay definitions now require exact factor and signal version maps as well as
  ordered registry keys and SHA-256 definition hashes. Snapshot quality is the
  weakest checkpoint/peer/factor/signal admission and cannot be upgraded; for
  missing or invalid signals it uses the explicit signal admission quality.
- Added Alembic revision **`p29003`** after `p29002`. It requires complete counts
  for eligible/node subjects and zero artifact counts for exclusions. Its insert
  guard validates distinct ID arrays, exact subject/T/effective-date/origin/build
  lineage, one artifact per frozen definition in canonical order, decision
  timing, warning/contradiction counts, and no quality upgrade. Existing snapshot
  and upstream immutability guards remain in force; no pushed migration changed.
- Added **3 regressions** using the real frozen Phase 2.7 definition registry and
  Phase 2.8 `SignalRunner`: a five-candidate plus evaluated-node, three-date
  production-flow scenario covers valid true, valid false, missing, invalid,
  excluded, quality inheritance, ordered lineage, a preloaded future result, and
  exact-rerun idempotency; a
  wrong-factor-build date fails atomically; and PostgreSQL rejects a signal ID
  substituted from another security. Combined replay suite: **26 passed**.
- Disposable PostgreSQL lifecycle passed fresh base → `p29003` → `p29002` →
  `p29001` → `p28002` → `p29003`, including eleven-table checks, downgrade
  behavior, metadata parity, and all 26 replay tests. Persistent Alembic
  current/head/check are **`p29003`** with no pending operations.
- Full regression: **931 passed, 8 existing warnings, zero failures** in 115.31s.
  `git diff --check` passed. Frozen Phase 2.2–2.8 formulas, thresholds, and rules
  are unchanged; this checkpoint adds no episode predictions, future outcomes,
  backtest aggregation, or Phase 2.10 behavior.
- Provider rights, historical completeness, survivorship qualification, and
  production calendar/data coverage remain hard external gates. Synthetic data
  supports only `DEVELOPMENT` research snapshots.
- Next: **2.9.4 — exact v1 signal-episode detection**.

### Checkpoint 2.9.2 — Replay schedule and runner foundation

Status: **COMPLETE**.

- Implemented `weekly_friday_2000_new_york_v1`: every local Friday inside the
  inclusive request plus one immediately preceding state-only warm-up. The
  20:00 New York cutoff converts through `zoneinfo`, so standard-time and DST UTC
  timestamps differ correctly. Friday holidays and pinned exceptional closures
  retain Friday T while the effective date moves to the latest qualified regular
  session; early closes do not move T.
- Added a content-addressed development calendar build and preserved the hard
  production gate: the repository calendar records
  `REPOSITORY_CALENDAR_NOT_PRODUCTION_QUALIFIED`, contributes `DEVELOPMENT`
  quality, and cannot support a stronger historical claim until an official
  calendar source is separately qualified.
- Added canonical per-date research/factor/signal manifest bundles. Every
  manifest is rebuilt and rehashed, must match exact T/effective date/origin and
  all frozen replay policies, and must form the exact research → factor → signal
  build chain. The existing `research_build_manifests` store and provider gate
  are reused; every provider is checked over the complete declared lookback-to-T
  interval, with no automatic quality downgrade.
- Added full sealed-universe reconstruction over the actual Phase 2.6
  classification, `peer_primary_common_equity_v1` eligibility, identity, taxonomy
  release, and node-version contracts. It selects the latest evidence version
  known by T before testing the effective interval, rejects ambiguous/empty
  corpora, enforces one eligible security per company, and sources ticker,
  exchange, lifecycle, and historical classification only from PIT evidence.
  Mutable `Security.ticker`, `exchange`, and `is_active` are not selectors.
- Added `HistoricalReplayRunner` with immutable semantic run fingerprints,
  deterministic checkpoint identities, strict chronological processing,
  date-level transactions, exact build/admission/candidate hashes, monotone
  run/checkpoint states, failed-date stop, and explicit deterministic resume.
  An injected prediction-side date publisher is the only extension point; this
  package imports no outcome/backtest service and does not require future data.
- Added Alembic revision **`p29002`** after `p29001` for checkpoint realized
  quality, admission, and limitations. Database guards prevent checkpoint
  downgrade, prevent snapshot quality from exceeding its admitted checkpoint,
  and require a completed run's count and weakest quality to equal its completed
  dates. No pushed migration was rewritten.
- Added **10 checkpoint tests** for weekly ordering/warm-up, New York DST,
  holiday Friday, exceptional closure, early close, historical-universe PIT,
  future membership exclusion, present-day survivor-field isolation,
  provider-coverage failure, chronological execution, sealed future-build
  rejection, failed-date atomicity, and resume after a completed date. Combined
  replay suite: **23 passed**.
- Disposable lifecycle passed fresh base → `p29002` → `p29001` → `p28002` →
  `p29002`, including the eleven-table boundary, checkpoint-column downgrade,
  metadata parity, and all 23 replay tests on fresh PostgreSQL. Persistent
  Alembic current/head/check are **`p29002`**.
- Full regression: **928 passed, 8 existing warnings, zero failures**.
  `git diff --check` passed. Frozen Phase 2.2–2.8 calculation modules and rules
  are unchanged; no snapshot publication, episode prediction, future outcome,
  backtest aggregation, or Phase 2.10 behavior was added here.
- Next: **2.9.3 — immutable full-population research snapshots**.

### Checkpoint 2.9.1 — Core models and migration

Status: **COMPLETE**.

- Added the frozen eleven-table storage boundary: replay runs/checkpoints,
  full-population research snapshots, immutable episode-start predictions,
  versioned outcome definitions/outcomes and prediction links, plus versioned
  backtest definitions/runs/populations/results. Existing signal, factor, peer,
  security, universe, and research-build artifacts remain the authoritative
  upstream lineage; they are referenced rather than copied.
- Added Alembic revision **`p29001`** after `p28002`. Relational guards require
  exact run/checkpoint/T/build identity, exact current fired SignalResult and
  immediately prior completed valid-false observation for predictions, weakest-
  side prediction quality, coherent outcome supersession, and exact case/control
  provenance. Immutable semantic rows reject update/delete; operational run and
  checkpoint rows permit only explicit monotone status transitions.
- Storage constraints cover semantic/run identities, episode uniqueness,
  definition and outcome versioning, quality enums, nonnegative counts, finite
  numerics, status-state coherence, fingerprints, and foreign-key integrity.
  The migration adds no speculative episode or result-slice table and changes no
  frozen Phase 2.2–2.8 formula, threshold, registry, or calculation rule.
- Added **13 focused PostgreSQL tests** covering the exact schema boundary, run
  and checkpoint identity, immutable predictions/outcomes, episode uniqueness,
  supersession/versioning, quality fail-closed behavior, status guards, finite
  outputs, backtest integrity, and foreign keys. All 13 pass.
- Disposable migration verification passed fresh base → `p29001` → `p28002` →
  `p29001`, confirmed all eleven tables, metadata parity, and reran all 13 focused
  tests on the fresh database. Persistent Alembic current/head/check are
  **`p29001`**, with no pending metadata operations.
- Full regression: **918 passed, 8 existing warnings, zero failures**.
  `git diff --check` passed. The warnings remain the pre-existing datetime and
  SQLAlchemy deprecations.
- Provider rights, historical completeness, survivorship, and production-data
  qualifications remain hard external gates. This checkpoint is storage only;
  it makes no predictive-performance or production-readiness claim.
- Next: **2.9.2 — weekly replay schedule and runner foundation**.

### Checkpoint 2.9.0 — Freeze historical replay specification

Status: **COMPLETE**.

- Started from approved clean `master` checkpoint
  `67b6e36a4f77de4648b67e3468859218f3f653e8`, equal to `origin/master`.
- Reproduced the required baseline: **905 passed, 8 existing warnings, zero
  failures**; Alembic current/head **`p28002`**.
- Read the complete Phase 2.9 architecture review, Phase 2.8 specification and
  freeze record, ledger and validation designs, data trust/provider gates, and
  project state. Inspected the actual manifest, provider-quality, bitemporal
  universe/security/classification, peer, feature, factor, signal, market-price,
  corporate-action, and trading-calendar contracts.
- Created the authoritative implementation specification:
  [phase-2.9-historical-replay-backtest-spec.md](phase-2.9-historical-replay-backtest-spec.md).
  It freezes the exact weekly New York schedule and warm-up; quality and universe
  gates; full-population snapshot; false-to-true episode policy; next-session
  entry; calendar-month horizons; total-return, aligned benchmark, drawdown/MFE/
  volatility formulas; terminal/censoring treatment; first-reported fundamental
  and same-target estimate labels; outcome maturity/versioning; descriptive
  case/control aggregation; quality/provenance; and atomic resume semantics.
- The minimum storage boundary is frozen at the architecture's eleven logical
  tables, reusing existing build manifests and upstream lineage. There is no
  speculative episode or slice table. The legacy provider/build-unaware market
  price store remains explicitly development-only.
- No application code, model, migration, database row, frozen formula/threshold,
  provider qualification, or Phase 2.10 behavior changed in this checkpoint.
- Next: **2.9.1 — core immutable replay/snapshot/prediction/outcome/backtest
  models and the migration after `p28002`**.

### Checkpoint 2.8.10 — Independent adversarial freeze review

Status: **PASS — FREEZE APPROVED**.

- Independently reviewed implementation commit
  `9e40f75c83c5179d919a10315c322487a85c01e8` against the complete frozen spec,
  architecture review, internal audit, Phase 2.7 freeze/specification, future
  prediction/outcome contracts, and the actual registry, typed evaluator,
  detectors, runner, storage, migrations, tests, lifecycle, and scale verifier.
- Confirmed exactly nine signal definitions, two detector identities, and 24
  conditions with the frozen Decimal thresholds and grouping. There is no dynamic
  expression surface, omnibus score, ranking, recommendation, probability,
  portfolio logic, or Phase 2.9 prediction/backtesting behavior.
- Independently exercised missing/invalid determinacy, reason precedence,
  multiple simultaneous proof branches, strength-first/canonical-ordinal proof
  selection, selected-proof-only quality, all warning/contradiction coexistence,
  exact industry scope, full condition lineage, factor-build/T isolation, and
  definition/result fingerprint content. The 4,160-case Early Growth lattice is
  a meaningful independent oracle over production evaluator types, not a mock of
  the production implementation.
- Reproduced and fixed two fail-closed request-boundary defects without changing
  economics or schema. All required signal/upstream manifest collection fields
  are now type-checked, and malformed nested admission quality or origin values
  cannot leak raw `TypeError`. Non-iterable, non-text, or unhashable registry-key
  requests now return `SIGNAL_REQUEST_INVALID`. Eighteen new regression cases
  prove controlled failure and zero result publication. The required-subject-scope
  path was separately probed and already behaved correctly.
- Live-schema inspection confirmed all four dedicated signal tables, lookup and
  fingerprint indexes, immutable-table triggers, and result/condition relational
  guards. Publication remains one atomic savepoint; injected failure rolls back
  complete batches, exact replay is idempotent, changed content fails closed, and
  real concurrent workers converge on one complete graph.
- `SignalRunner` consumes only immutable `FactorValue` artifacts. The production
  T1/T2 lifecycle contains no manually injected SignalResult. Exact factor/signal
  builds, T/effective date/origin, industry mapping, definitions, conditions, and
  proofs remain reconstructible. The Phase 2.2/2.4/2.5/2.6/2.7 economics are
  unchanged.
- Reviewed the 1k/5k/10k structural proof: SQL statements are 220/364/544,
  exactly 36 more per 1,000 securities after fixed setup; 10k publishes 80,001
  factor artifacts, 70,002 results, and 220,002 condition rows in 313.169 seconds
  with 275.438 MiB signal-run RSS growth and zero PostgreSQL temp I/O. The review
  changes add only constant-size request validation, so no scale rerun was
  warranted.
- Tests: signal suite **168 passed**; full regression **905 passed, 8 existing
  warnings, zero failures**; Early Growth lattice **4,160 passed**. Fresh
  disposable base → `p28002` → `p27001` → `p28002`, metadata parity, all signal
  tests, and real concurrency passed. Persistent Alembic current/head/check remain
  **`p28002`**; `git diff --check` passed.
- The complete independent record is
  `docs/PHASE_2_8_FREEZE_REVIEW.md`. This freeze says signals deterministically
  interpret evidence. It does not assert predictive power, alpha, calibrated
  probability, or recommendation quality.
- Next: **Phase 2.9 is READY TO DESIGN** as the prediction replay, outcome ledger,
  and backtesting layer. Do not treat frozen Phase 2.8 signals as validated
  predictions or silently mutate their v1 semantics.

### Checkpoint 2.8.9 — Final internal implementation audit

Status: **COMPLETE — PASS AFTER TWO CORRECTIONS**.

- Starting checkpoint `8a859963c2eef80b1a0b4bb75b6d678b419b2939` was equal
  to `origin/master` with a clean worktree.
- Audited the implementation against the frozen Phase 2.8 contract and original
  architecture across exact scope, all nine definitions/two detectors/24
  conditions, thresholds and grouping, the closed typed tree, missing/invalid
  determinacy, evidence strength/severity, proof ordering, quality isolation,
  coverage, PIT/build pinning, relational provenance, idempotency, immutability,
  atomicity/concurrency, migrations, T1/T2 lifecycle, and 10k scale behavior.
- The diff from approved checkpoint `84066fa05cc6bc4e3dcb49089755c1779f225b05`
  adds only the Phase 2.8 implementation, migrations, tests, scripts, and docs
  plus signal-model exports. It changes no frozen Phase 2.2, 2.4, 2.5, 2.6, or
  2.7 formula, calculator, normalization, registry, or runner semantics. Searches
  found no expression evaluator/dynamic code or prohibited recommendation,
  ranking, probability, portfolio, valuation, news, AI, ledger, or backtester
  output in the signal implementation.
- Added an independent Early Growth falsification oracle covering all **4,096**
  pass/fail/missing/invalid combinations across the two core leaves and four
  confirmation families, plus all **64** nested Fundamental Economics member
  combinations. All **4,160** cases match the frozen `ALL`/`AT_LEAST(1)`
  determinacy. Eighteen additional exact Decimal cases cover below, exactly at,
  and above every Early Growth confirmation threshold.
- A combined adversarial case fires all five Early Growth diagnostics at their
  exact inclusive boundaries—margin deterioration, cash weakness, weak
  profitability, industry headwind, and fragility—while Early Growth still fires
  through its selected Market proof. Separate Cash Flow Contradiction, Industry
  Headwind, and Fragility Warning results fire simultaneously; none is a veto.
- Reproduced and fixed one frozen-semantics mismatch: when no confirmation passed,
  the independent-confirmation group hid specific build/timestamp/quality/context
  failures behind generic `INSUFFICIENT_CONFIRMATION_COVERAGE`. Generic missing
  alternatives still use that reason, but higher-precedence exact reasons now
  survive. A production-runner test proves a wrong-build Estimate Revision yields
  persisted `MISSING/BUILD_MISMATCH` with condition-local lineage. No threshold,
  grouping, proof ordering, quality policy, factor formula, or definition hash
  changed.
- Reproduced and fixed one request-error boundary: wrongly typed nested signal or
  upstream manifest collections leaked `AttributeError`. The runner now validates
  calculation-version/configuration mappings first and fails closed with
  `BUILD_MANIFEST_INVALID`; parameterized tests prove no result is published.
- Updated the README with the feature/factor/signal distinction, exact nine-signal
  registry, Early Growth rule, missing-estimate semantics, intentional
  warning/opportunity coexistence, non-probability boundary, current status,
  migration head, and validation count.
- The complete audit is recorded in
  `docs/phase-2.8-implementation-audit.md`. Phase 2.8 remains `CANDIDATE` and is
  ready for independent review; implementation completion is not semantic freeze,
  empirical predictive validation, calibration, provider qualification, or a
  production recommendation.
- Tests: signal suite **150 passed**; combined frozen factor plus
  signal/lifecycle regression **367 passed**; full regression **887 passed, 8
  existing warnings, zero failures**. Fresh disposable migration lifecycle and
  metadata parity passed with all 150 signal tests. Persistent Alembic current
  and head remain **`p28002`**.
- Next: **independent Phase 2.8 QC/architecture review**. Do **not** begin Phase
  2.9, promote signals, or weaken PIT/quality/provider gates.

### Checkpoint 2.8.8 — 10,000-security structural scale sanity

Status: **COMPLETE**.

- Starting checkpoint `c64c561a3cea4e57bb80c7e1a13cfe5f3d046e38` was equal
  to `origin/master` with a clean worktree.
- Added a disposable PostgreSQL scale verifier that migrates a random database
  from base to head, seeds exact immutable Phase 2.7 factor artifacts, executes
  the production `SignalRunner` for all nine definitions and both detectors, and
  force-drops only the random database. It measures SQL statements, wall time,
  sampled process RSS, PostgreSQL temp I/O, exact artifact/status cardinality,
  and representative `EXPLAIN (ANALYZE, BUFFERS)` plans.
- The final 1k/5k/10k runs published exactly 7,002/35,002/70,002 signal results
  and 22,002/110,002/220,002 condition rows from 8,001/40,001/80,001 factor
  values. All results were `VALID/PIT_QUALIFIED`; fired, non-fired, full/reduced
  coverage, and two node-result counts matched their closed-form expectations.
  Early Growth and Fragility Warning intentionally fired together at scale.
- SQL totals were 220, 364, and 544. After fixed setup the increase is exactly
  36 statements per additional 1,000 securities, matching four bounded
  250-security batches. There are no per-security definition queries,
  per-condition factor queries, or observed N+1 access. The 10k plans returned
  at most 5,502 rows per captured batch, completed in at most 40.749 ms, and
  used zero temp blocks. Database temp-file and temp-byte deltas were both zero
  at every scale.
- Wall times were 31.676 s, 154.791 s, and 313.169 s on the designated 2-vCPU,
  8-GB development host. This is structural evidence, not a production latency
  SLO or benchmark-vanity target.
- The initial matrix exposed avoidable retention of every result's large
  admission/provenance documents. Large multi-batch result queries now defer
  those ORM attributes while still persisting the full JSON. Exact reruns use
  an independent batched PostgreSQL JSON-content comparison, so deferral does
  not weaken immutable publication checks. Signal-run RSS growth fell from
  179.137/568.438/1,053.480 MiB to 111.871/185.055/275.438 MiB at 1k/5k/10k.
- Added focused tests proving caller-order preservation, deferred/lazy payload
  access, exact replay across batches, and fail-closed detection of deliberately
  corrupted deferred provenance. The full method and limitations are recorded
  in `docs/phase-2.8-signal-scale-sanity.md`.
- Tests: runner **16 passed**; combined frozen factor plus signal/lifecycle
  regression **356 passed**; full regression **876 passed, 8 existing warnings,
  zero failures**. `git diff --check` passed. Alembic remains **`p28002`**; no
  schema or economic signal semantics changed.
- Next: **2.8.9 — final internal audit, adversarial Early Growth falsification,
  README boundary documentation, and independent-review handoff**.

### Checkpoint 2.8.7 — Full synthetic T1/T2 signal lifecycle

Status: **COMPLETE**.

- Starting checkpoint `26427e26717e974db1c1973673497d4981ff9950` was equal
  to `origin/master` with a clean worktree.
- Added one deterministic fictional Semiconductor Infrastructure lifecycle at T1
  and T2. It publishes frozen upstream source features, runs the real Phase 2.6
  `PeerFeatureRunner` and `IndustrySnapshotRunner`, runs the real Phase 2.7
  `FactorRunner` for all nine factors, and finally runs the production Phase 2.8
  `SignalRunner` for all nine definitions and both detector identities. No peer,
  industry, factor, signal, or signal-condition result is manually injected.
- T1 uses broad growth, acceleration, profitability, margin, cash, estimate,
  market, and industry strength together with intentionally high observed market
  fragility. T2 uses a separate later source/factor/signal build with weaker
  growth and acceleration, deteriorating margins/cash, negative revisions,
  weaker market/industry evidence, and continuing high fragility. Production
  factor outputs are:

  | Factor | T1 | T2 |
  | --- | ---: | ---: |
  | `growth_v1` | `0.9075` | `0.0625` |
  | `growth_acceleration_v1` | `0.755` | `-0.32666666666666666666666666666666666667` |
  | `profitability_v1` | `0.96428571428571428571428571428571428572` | `0.15571428571428571428571428571428571428` |
  | `margin_expansion_v1` | `0.8475` | `-0.33375` |
  | `cash_flow_quality_v1` | `0.885` | `-0.21666666666666666666666666666666666667` |
  | `estimate_revision_v1` | `0.885` | `-0.4025` |
  | `market_leadership_v1` | `0.74` | `-0.47833333333333333333333333333333333333` |
  | `industry_strength_v1` | `0.66562499999999999999999999999999999997` | `-0.48784722222222222222222222222222222221` |
  | `market_fragility_v1` | `0.70` | `0.70` |

- T1 fires Fundamental Acceleration, Margin Inflection, Estimate Confirmation,
  Market Confirmation, Industry Tailwind, Fragility Warning, and Early Growth.
  Industry Headwind and Cash Flow Contradiction are valid non-fires. Critically,
  Fragility Warning coexists with and does not suppress Early Growth.
- At T2, the opportunity/confirmation rules transition to valid non-fires because
  the strong Growth/Acceleration core and confirmations weakened. Fragility
  Warning remains the sole fired signal. The just-above-`-0.50` node value remains
  a valid Industry Headwind non-fire; no threshold was tuned around the fixture.
- Each snapshot publishes exactly nine authoritative results and 24 complete
  relational condition rows. Their factor-value FK set equals the exact nine
  production factor artifacts, including the exact historical node/group factor.
  Both builds retain `HISTORICAL_REPLAY` origin and synthetic `DEVELOPMENT`
  quality; this is an orchestration/invariant test, not provider qualification or
  evidence of predictive success.
- Publishing T2 creates distinct factor and signal build artifacts without
  changing T1. A final T1 signal replay after T2 returns the original nine IDs,
  fingerprints, and calculation times. The test therefore proves build pinning,
  PIT isolation, signal transition, complete lineage, and idempotent history.
- Fixed a test-discovery robustness issue exposed by combined directory-order
  execution: the runner test now defines its timestamp locally and obtains its
  fixture factory by type rather than importing an ambiguous bare `conftest`.
  No production semantics changed.
- Tests: lifecycle **1 passed**; adjacent Phase 2.7 plus Phase 2.8 lifecycles
  **2 passed**; combined factor/signal/lifecycle regression **355 passed**; full
  regression **874 passed, 8 existing warnings, zero failures**.
  `git diff --check` passed. Alembic remains **`p28002`**.
- Next: **2.8.8 — 10,000-security structural scale sanity: SQL statements,
  runtime, RSS, PostgreSQL temp I/O, and N+1/query-plan audit**.

### Checkpoint 2.8.6 — Build-pinned SignalRunner and atomic persistence

Status: **COMPLETE**.

- Starting checkpoint `58670d68aab79f537b035a697e3d8b7850c5e2e2` was equal
  to `origin/master` with a clean worktree.
- Added the isolated `SignalRunner`, `SignalSubject`, and functional
  `calculate_signals` entry point. The runner consumes only already-published
  immutable Phase 2.7 `FactorValue` artifacts; it never invokes `FactorRunner`
  or recreates a factor formula.
- The signal manifest now fail-closed pins the exact requested registry and
  definition versions/hashes, detector versions, runner and all five policy
  versions, factor registry and definition versions/hashes, upstream factor
  build/manifest hash, T/effective date/origin, signal quality admission, and
  `security_peer_group_member_exact_v1` context-mapping policy. The pinned
  upstream manifest is independently re-hashed and validated.
- Implemented bounded set-based factor retrieval and publication. Definitions
  and manifests are validated once; security subjects are resolved in bounded
  batches; exact node/group context and included membership are resolved once;
  Decimal trees are evaluated in memory; and result/condition rows are inserted
  in parameter-safe batches inside one caller savepoint. No condition or subject
  performs its own factor query.
- Published fired, valid non-fired, missing, and invalid results with every
  declared condition. Each condition pins the exact factor-value ID when an
  exact artifact exists and retains its value/status/reason/quality/coverage/
  admission, threshold/operator, relation, exclusion, selected-proof state, and
  selection provenance. Compact result provenance reconstructs the subject,
  detector/policies, both builds, definition, factor IDs, node mapping, proof
  arithmetic, coverage, quality, and final output; content fingerprints cover
  the full ordered evidence and decision.
- Added migration **`p28002`** without modifying `p28001`. It permits selected
  evaluated failure witnesses for deterministic valid non-fire proofs and adds
  a relational insert guard requiring an included exact `PeerGroupMember` when a
  security signal references a node-scoped industry factor. Models and migration
  metadata match.
- Exact replay preserves result/condition IDs, fingerprints, and calculation
  times. Changed content at an existing semantic identity fails closed. New
  signal/factor builds create separate artifacts. Wrong-build and future factor
  values are excluded; missing estimates/context remain unavailable rather than
  zero; selected `DEVELOPMENT` proof quality propagates while unselected
  development evidence does not contaminate a qualified proof.
- Added **14 runner tests** covering all nine definitions and matching scopes,
  complete relational lineage, exact replay, valid non-fire failure proof,
  missing Estimate Revision with alternate Market proof, selected/unselected
  proof quality, wrong/future factor exclusion, signal/detector/build isolation,
  runner isolation, malformed requests, single- and cross-batch rollback, and
  real two-writer PostgreSQL convergence on one complete graph.
- Disposable lifecycle proof passed fresh base → `p28002` → `p27001` →
  `p28002`, four-table cardinality, metadata parity, and the full signal suite on
  fresh PostgreSQL. Persistent Alembic current/head/check is **`p28002`**.
- Tests: **137 signal tests passed**; combined frozen factor plus signal
  regression **353 passed**; full regression **873 passed, 8 existing warnings,
  zero failures**. `git diff --check` passed.
- Next: **2.8.7 — full T1/T2 fictional semiconductor lifecycle using production
  factor artifacts and the production SignalRunner, with no manual signal-row
  injection**.

### Checkpoint 2.8.5 — Early Growth detector

Status: **COMPLETE**.

- Starting checkpoint `3ee5e6d4468afbc3f313a31104ff53252c07ea92` was equal
  to `origin/master` with a clean worktree.
- Added the explicit `early_growth_detector_v1` execution boundary. It emits
  exactly `early_growth_candidate_v1` at security scope and delegates to the
  frozen typed rule/evidence/proof evaluator without recalculating factors or
  producing ranks, probabilities, recommendations, or portfolio output.
- Verified the exact required core of Growth and Growth Acceleration both
  `>= +0.50`, plus at least one independent confirmation family: Estimate
  Revision, Market Leadership, or node-scoped Industry Strength `>= +0.50`, or
  one Fundamental Economics member (Profitability, Margin Expansion, or Cash
  Flow Quality) `>= +0.10`. Multiple Fundamental Economics members still count
  as one independent family.
- Missing Estimate Revision remains explicitly unavailable and reduces coverage,
  but does not block a candidate proved by another admitted confirmation. When
  no alternate proof exists, potentially decisive missing confirmation yields
  `MISSING`, not `VALID/fired=false`. A missing required Growth core input also
  yields `MISSING`; a valid below-threshold required input yields a valid
  non-fire.
- Confirmed deterministic selected-proof quality in both directions: an
  unselected `DEVELOPMENT` estimate does not downgrade a stronger selected
  qualified Market proof, while a selected strongest `DEVELOPMENT` estimate
  propagates `DEVELOPMENT`. Proof selection remains strength-first with frozen
  ordinal tie-breaking.
- Verified intentional coexistence with Fragility Warning, Cash Flow
  Contradiction, Industry Headwind, and Margin Deterioration diagnostic evidence.
  These warnings/contradictions remain visible and unselected; none vetoes Early
  Growth. Tailwind and Headwind remain mutually consistent for the same exact
  industry factor state.
- Added **19 Early Growth detector tests** spanning every confirmation family,
  missing and failed required inputs, missing estimates with and without an
  alternate proof, warning/contradiction coexistence, selected-proof quality,
  and confirmation-family cardinality.
- Tests: **123 signal tests passed**; combined frozen factor plus signal
  regression **339 passed**; full regression **859 passed, 8 existing warnings,
  zero failures**. `git diff --check` passed. Alembic remains **`p28001`**; no
  model or migration changed in this checkpoint.
- Next: **2.8.6 — build-pinned batch SignalRunner, relational publication,
  idempotency, isolation, and atomic/concurrent persistence**.

### Checkpoint 2.8.4 — Atomic/context/warning signal detector

Status: **COMPLETE**.

- Starting checkpoint `708ff9c49bd7c9bf1061efd6edd24c5859394edd` was equal
  to `origin/master` with a clean worktree.
- Added the explicit `factor_pattern_detector_v1` execution boundary over exactly
  the first eight frozen definitions. It emits the six security-scoped results
  and two node-scoped industry-context results in canonical registry order, with
  no Early Growth, ranking, probability, recommendation, or implicit extra output.
- Verified the exact rules: Growth and Growth Acceleration both `>= +0.50`;
  Margin Expansion `>= +0.50`; Estimate Revision `>= +0.50`; Market Leadership
  `>= +0.50`; Industry Tailwind `>= +0.50`; Industry Headwind `<= -0.50`;
  Growth `>= +0.50` with Cash Flow Quality `<= -0.10`; and Market Fragility
  `>= +0.50`. Market Fragility retains the frozen orientation where positive is
  more fragile.
- Margin Inflection persists exact negative-Growth contradiction context without
  vetoing the opportunity. Missing Estimate Revision yields `MISSING`, not a
  false/neutral result, and selected synthetic estimate evidence propagates
  `DEVELOPMENT`. Market Confirmation and Fragility Warning can fire together.
- Added **36 atomic detector tests** covering one 38-digit Decimal unit below,
  exactly at, and above every strong positive, strong negative, and negative
  threshold; both Fundamental Acceleration required leaves; contradiction
  grouping; missing/synthetic estimates; warning coexistence; scope-specific
  output sets; invalid detector requests; and Tailwind/Headwind mutual
  consistency for the same exact node factor.
- Tests: **104 signal tests passed**; combined frozen factor plus signal regression
  **320 passed**; full regression **840 passed, 8 existing warnings, zero
  failures**. `git diff --check` passed. Alembic remains **`p28001`**; no model or
  migration changed in this checkpoint.
- Next: **2.8.5 — `early_growth_detector_v1`, all independent confirmation
  families, missing-estimate behavior, and warning/contradiction coexistence**.

### Checkpoint 2.8.3 — Evidence strength, proof, quality, and coverage

Status: **COMPLETE**.

- Starting checkpoint `f043949cbc7ea4541423f2a02b3046dba5696dee` was equal
  to `origin/master` with a clean worktree.
- Implemented the exact passed-leaf margins
  `(x - theta) / (1 - theta)` for `GTE` and
  `(theta - x) / (theta + 1)` for `LTE` under the frozen 38-digit
  `ROUND_HALF_EVEN` Decimal context. `ALL` uses the minimum child margin;
  `AT_LEAST(k)` uses the kth-largest passing child margin.
- Implemented recursive deterministic proof selection. Passing `ALL` selects all
  required child proofs; passing `AT_LEAST(k)` selects exactly the k strongest
  branches, breaking equal-strength ties by canonical child ordinal. Nested
  Fundamental Economics first selects its strongest member before competing as
  one Early Growth confirmation family. Supplemental passing evidence remains
  visible but is not selected.
- Only fired valid results receive evidence strength and severity. Severity uses
  exact `3 * strength` comparisons with guard precision at the rational one-third
  and two-third boundaries: `LOW`, `MEDIUM`, then `HIGH`. Threshold equality
  correctly fires with numeric strength zero and `LOW` severity.
- Signal quality is the weakest signal admission and selected proof factor only.
  A selected synthetic Estimate Revision yields `DEVELOPMENT`; an unselected
  development estimate remains supplemental and does not contaminate a qualified
  Market/Industry/Fundamental-Economics proof. Deterministic strength/ordinal
  selection occurs before quality propagation, so quality cannot choose a branch.
- Added distinct-factor admitted/missing/invalid/selected counts, exact condition
  coverage, minimum selected upstream weight coverage, and
  `FULL | REDUCED | NOT_APPLICABLE`. Unavailable or invalid non-decisive evidence
  reduces coverage without blocking a determinate proof. A selected upstream
  `VALID/REDUCED` factor makes signal coverage reduced.
- Valid non-fires retain a deterministic minimal failure proof and proof-local
  quality but have null strength/severity. Missing/invalid results retain counts
  and condition evidence while making no signal-quality claim.
- Added **24 proof/strength tests** covering exact threshold/one-unit-above/extreme
  margins, negative-side margins, exact severity neighbors, multiple passing
  families, nested proof selection, canonical ties, selected development and
  selected qualified proofs, supplemental-quality isolation, missing/invalid
  alternatives, reduced upstream coverage, and valid non-fire proof quality.
- Tests: **68 signal tests passed**; combined frozen factor plus signal regression
  **284 passed**; full regression **804 passed, 8 existing warnings, zero
  failures**. `git diff --check` passed. Alembic remains **`p28001`**; no model or
  migration changed in this checkpoint.
- Next: **2.8.4 — the eight atomic/context/warning signal definitions and exact
  economic-boundary verification under `factor_pattern_detector_v1`**.

### Checkpoint 2.8.2 — Closed typed rule evaluator

Status: **COMPLETE**.

- Starting checkpoint `6a3ca9757fa0c20579d32d60f13a9882839721ea` was equal
  to `origin/master` with a clean worktree.
- Added a side-effect-free evaluator over only the frozen `Comparison`, `ALL`,
  and `AT_LEAST(k)` dataclasses. Comparisons are inclusive exact Decimal `GTE` /
  `LTE`; every leaf is evaluated and returned in canonical tree/ordinal order.
  There is no parser, expression string, `eval`, dynamic Python, arbitrary SQL,
  or user-provided execution surface.
- `ALL` requires every child to be valid before claiming a determinate true or
  false result. `AT_LEAST(k)` becomes true as soon as k valid alternatives pass;
  missing/invalid alternatives make it unavailable only when they could still
  change the outcome. Integrity/invalid reasons precede definition/scope/time,
  build, quality, industry-context, and generic missing reasons.
- Added typed exact factor evidence and immutable evaluation records. Valid
  evidence must be finite, bounded `[-1,+1]`, quality-admitted, and carry valid
  upstream coverage. Missing remains unavailable rather than zero; invalid and
  non-finite inputs fail closed. Diagnostics are evaluated and retained but never
  affect primary rule determinacy.
- Early Growth now evaluates correctly at the pure rule layer: missing estimates
  do not block a valid Market/Industry/Economics witness, while no passing
  confirmation plus a potentially decisive missing family yields
  `MISSING/INSUFFICIENT_CONFIRMATION_COVERAGE` rather than false.
- Added **21 rule-engine tests** covering exact threshold neighbors, `ALL`, nested
  `AT_LEAST(1)` and generic `AT_LEAST(k)`, missing/invalid inputs, required versus
  alternative behavior, reason precedence, canonical branch ordering, missing
  estimates with alternate confirmation, malformed numerics/admission, and the
  absence of dynamic execution.
- Tests: **44 signal tests passed**; combined frozen factor plus signal regression
  **260 passed**; full regression **780 passed, 8 existing warnings, zero
  failures**. `git diff --check` passed. Alembic remains **`p28001`**; no model or
  migration changed in this checkpoint.
- No factor calculation, signal persistence runner, ranking, probability,
  recommendation, or Phase 2.9 behavior was added.
- Next: **2.8.3 — exact rule-margin strength, severity, selected proof, coverage,
  and proof-local quality propagation**.

### Checkpoint 2.8.1 — Signal definitions and dedicated storage

Status: **COMPLETE**.

- Starting checkpoint `3ece4b4c20b3e396cdeaabbcba61d62710626bd0` was equal
  to `origin/master` with a clean worktree.
- Added the exact code-owned registry of nine immutable `CANDIDATE` signals and
  two detector identities. The registry contains 24 ordered decision/diagnostic
  leaves linked to the actual nine frozen `ResearchFactor` definitions; canonical
  payloads and SHA-256 hashes include exact factor ID/version/hash, grouping,
  role, operator, threshold, detector, and all policy versions.
- Added four dedicated SQLAlchemy models and migration **`p28001`**:
  `research_signals`, `research_signal_conditions`, `signal_results`, and
  `signal_result_conditions`. Signals do not reuse feature/factor value storage,
  and no redundant detector-result table exists.
- Definition registration is idempotent for exact content, creates a distinct
  artifact for a new signal version, and fails rather than repairing a published
  collision. Result storage distinguishes `VALID`, `MISSING`, and `INVALID`;
  enforces nullable fired/strength/severity semantics; accepts only finite bounded
  evidence strength/coverage, exact quality/severity enums, and one valid subject
  scope; and pins both signal and upstream factor build manifests.
- PostgreSQL guards verify definition scope, distinct declared-factor count,
  manifest timestamp and exact factor-build pin, node/group identity, condition
  ownership, exact factor definition/T/build/subject lineage, snapshotted factor
  value/status/quality/coverage/admission, operator/threshold, proof role, and
  evidence relation. All four signal tables reject `UPDATE` and `DELETE`.
- Added a destructive-safe disposable verifier. Fresh base → `p28001` → `p27001`
  → `p28001`, four-table cardinality, Alembic metadata parity, and focused tests
  passed. Persistent Alembic current/head/check is **`p28001`**.
- Tests: **23 signal storage/registry tests passed**; combined frozen factor plus
  signal regression **239 passed**; full regression **759 passed, 8 existing
  warnings, zero failures**. `git diff --check` passed.
- No frozen Phase 2.2, 2.4, 2.5, 2.6, or 2.7 formula/runner semantics changed.
  No signal evaluation, ranking, probability, prediction, or Phase 2.9 behavior
  was added in this checkpoint.
- Next: **2.8.2 — closed typed rule evaluator, determinacy, and deterministic
  branch-order tests**.

### Checkpoint 2.8.0 — Freeze signal and detector specification

Status: **COMPLETE**.

- Started from approved clean `master` checkpoint
  `84066fa05cc6bc4e3dcb49089755c1779f225b05`, equal to `origin/master`.
  Reproduced the required baseline: **736 passed, 8 existing warnings, zero
  failures**; Alembic current/head **`p27001`**.
- Read the complete Phase 2.8 architecture review, Phase 2.7 specification and
  freeze review, prediction/outcome ledger, historical validation plan,
  validation/learning design, and project state. Inspected the actual frozen
  factor registry, definition/value models, manifest adapter, quality/admission,
  provenance, fingerprint, and atomic publication implementation.
- Created the authoritative implementation contract:
  [phase-2.8-signal-detector-spec.md](phase-2.8-signal-detector-spec.md). It freezes
  exactly nine candidate signals and two detectors; the closed typed rule tree;
  exact thresholds/grouping/diagnostics; deterministic proof selection;
  missing/invalid determinacy; rule-margin strength; exact-third severity;
  selected-proof quality; coverage; dedicated relational storage; factor-build
  pinning; provenance; immutability; publication; and scale/test contracts.
- No application code, model, migration, definition row, signal result, factor
  artifact, frozen formula/semantics, provider qualification, prediction, rank,
  probability, recommendation, or Phase 2.9 behavior changed in this checkpoint.
- Next: **2.8.1 — signal definitions, dedicated storage, migration,
  immutability, build linkage, and relational-integrity tests**.

### Independent Phase 2.7 freeze review

Status: **PASS — FREEZE APPROVED**.

- Independently reviewed clean `master` at
  `76eab0897a6d0d09b4bb6f79c35e32d0f7d86337` against the factor specification,
  architecture review, implementation audit, scale report, prior Phase 2.5/2.6
  freeze reports, actual models/migration/code/tests, and Git history. The exact
  registry remains nine v1 factors, 19 subfactors, and 47 components.
- Source IDs, grouping, required anchors, fixed component/subfactor weights,
  orientation, deterministic Decimal normalization, exact `2p-1` percentiles,
  60% subfactor coverage, 70% factor coverage, reduced-state semantics, and local
  hierarchical renormalization match the frozen contract. Missing is not zero;
  non-finite values fail closed. No factor economics were retuned.
- Reproduced and fixed fail-closed defects in four areas: weakest row/manifest
  quality admission and malformed-admission fallback; declared direct-source and
  exact estimate dataset/build identity; peer effective-date/source-build/dataset
  lineage; and industry metric-to-source-statistic/dataset lineage. Each has an
  adversarial regression. Upstream-artifact corruption and the internal audit's
  deferred-payload replay fix both fail closed on rerun.
- PIT/build resolution now verifies exact T, period/target, effective date,
  calculation build, declared dataset, authoritative peer result/statistic/group,
  and industry metric/statistic/source feature. Quality remains factor-local and
  is the weakest factor/source admission. Synthetic evidence cannot claim
  production quality or downgrade unrelated factors.
- Definitions, values, subfactors, component decisions, and manifests remain
  immutable and idempotent in seven dedicated factor tables at `p27001`.
  Included and excluded decisions retain raw/normalized values, weights,
  contributions, reasons, exact source FKs, admissions, and selection evidence.
  Concurrent publication still converges atomically on one complete graph.
- Current-code scale proofs passed at 1k/5k/10k securities with 128/368/668 SQL
  statements and 353.258/453.484/560.969 MiB peak RSS. The corrected 10k run
  persisted 80,001 factors, 160,003 subfactors, and 390,008 component decisions
  in 1,345.070 seconds with 36,913,152 temp bytes. Statement growth is fixed per
  250-security batch; only the wide exact-source sort spilled at `work_mem=4MB`.
  There is no N+1 or industry Cartesian expansion.
- Validation: **216 factor tests**, **223 focused factor/peer/lifecycle tests**,
  **736 full-suite tests with 8 existing warnings and zero failures**, disposable
  base → `p27001` → `p26008` → `p27001`, metadata parity, concurrent publication,
  and corrected-code 10k scale proof. Alembic current/head/check is `p27001`.
- Frozen upstream Phase 2.2, 2.4, 2.5, and 2.6 formula/runner bodies remain
  unchanged. Phase 2.7 contains no recommendation, probability, final score,
  universe rank, or signal generation. Features are measurements; factors are
  inspectable evidence summaries; signals belong to Phase 2.8.
- Production-provider qualification, licensing, broad historical coverage,
  complete identity/delisting history, empirical predictive validation,
  calibration, and promotion remain external/future gates. Freeze does not claim
  alpha or turn a factor value into a probability.
- Canonical decision and detailed evidence:
  `docs/PHASE_2_7_FREEZE_REVIEW.md`. Any semantic change to v1 requires a new
  factor version. **Phase 2.8 is ready to design but has not started.**

### Checkpoint 2.7.10 — Final implementation audit

Status: **COMPLETE**.

- Starting checkpoint `5c186947ed3aa8f102dfe7c38831ae3f6c98781d` was equal
  to `origin/master` with a clean worktree. The audit used the frozen
  `docs/phase-2.7-factor-spec.md` and architecture review as the standard rather
  than treating existing tests as the specification.
- The exact registry remains nine inactive `CANDIDATE` factors, 19 subfactors,
  and 47 components: eight security factors plus one node-scoped Industry
  Strength factor. All scopes, orientations, source IDs/versions/units,
  subfactor/component weights, anchors, normalization knots, 60%/70% coverage
  gates, quality policy, definition hashes, and explicit exclusions match the
  frozen contract. No rank, signal, probability, recommendation, omnibus score,
  valuation, business-quality, sentiment, peer-leadership, or operating-leverage
  factor was added.
- PIT/build review confirmed exact manifest pinning; fundamental no-fallback and
  135-day semantics; exact-T estimate/market/peer/industry selection; fiscal,
  target, peer-source, and node/group coherence; immutable historical artifacts;
  and factor-local weakest-input quality. A diff from the approved `f51f174...`
  base changes no frozen Phase 2.2, 2.4, 2.5, or 2.6 formula/runner file.
- Dedicated storage at `p27001` retains canonical manifests, immutable versioned
  definitions, complete relational component/subfactor lineage, build identity,
  finite bounded values, statuses/reasons, coverage, quality/admission, exact
  source FKs, fingerprints, and append-only promotion hooks. Factors are not
  stored in `FeatureValue`; Phase 2.8 has not started.
- **Reproduced and fixed one final-audit defect:** large requests deferred factor
  admission/provenance and compared them only through the stored fingerprint.
  A rollback-only test bypassed the immutability trigger, changed only persisted
  provenance while preserving the fingerprint, and proved the rerun previously
  accepted it. Pre-existing/concurrently won rows now receive one batch-level
  PostgreSQL JSONB content comparison; current-statement inserts avoid redundant
  readback and returned JSON remains deferred. The tamper case now fails closed
  with `PUBLISHED_FACTOR_CONFLICT_USE_NEW_BUILD_OR_VERSION`.
- Closed explicit verification gaps: changed-content conflict, a v2 definition
  producing a distinct value artifact beside v1, a forced multi-batch exact
  replay, and two real concurrent transactions converging on one complete graph.
  While the first writer was uncommitted, an independent reader saw zero factor,
  subfactor, and component rows; afterward both writers returned the same identity
  and exactly one factor/one subfactor/three components remained.
- The accepted 10,000-security checkpoint evidence remains 80,001 factors,
  160,003 subfactors, 390,008 component decisions, 667 SQL statements,
  1,337.533 seconds, 549.125 MiB peak process RSS, and 120,815,616 PostgreSQL temp
  bytes. The final fix does not execute its JSONB comparison for fresh inserts.
  Its existing-row path passed a maximum internal-batch replay over 250 securities
  plus one industry node, all nine definitions, 2,001 identical factor artifacts,
  and one equality query. Detailed late-session memory constraints and rejected
  alternatives are recorded in `docs/phase-2.7-implementation-audit.md`.
- Updated README status and plain-language factor documentation: features are
  measurements, factors are interpretable evidence summaries, and signals remain
  a future combination. It lists all nine meanings and states explicitly that a
  factor score is neither a probability nor a buy recommendation.
- Focused factor/runner/lifecycle verification: **212 passed**. Full regression:
  **725 passed, 8 existing warnings, zero failures**. Disposable migration proof:
  fresh base → `p27001` → `p26008` → `p27001`, seven factor tables,
  `alembic check` metadata parity, **209 factor tests**, and concurrent atomic
  publication.
  Persistent Alembic current/head remains **`p27001`**; no new migration or rewrite
  was required.
- External provider quality, licensing, retained historical coverage, delisted/
  identity completeness, and empirical validation remain gates. Synthetic factor
  evidence is not production research. Phase 2.7 is complete but **not frozen**.
- Next: **independent Phase 2.7 QC/architecture review**. Do not begin Phase 2.8.

### Checkpoint 2.7.9 — Factor scale / performance sanity

Status: **COMPLETE**.

- Starting checkpoint `1fa7bd5778dd8afae77551b7b58c2301bb8cc2e5` was equal
  to `origin/master` with a clean worktree.
- Added `scripts/verify_factor_scale.py`, a destructive-safe PostgreSQL proof
  that creates a random disposable database, migrates it from base to `p27001`,
  seeds 10,000 synthetic securities with the exact manifest-pinned representative
  source set, runs the exact nine definitions, captures SQL/runtime/process-memory/
  database-temp-I/O/query-plan evidence, and drops only the database it created.
  No configured research database is populated, downgraded, or dropped.
- The successful PostgreSQL 16.15 run published **80,001 factor values, 160,003
  subfactor rows, and 390,008 relational component decisions**. It produced
  10,008 `VALID`, 69,993 explicit `MISSING`, zero `INVALID`, and 80,001
  `DEVELOPMENT` results. Exact cardinalities prove the single industry-node
  result and its eight components were not multiplied across 10,000 securities.
- Measured factor-run wall time was **1,337.533 seconds**. The run issued **667
  SQL statements** (304 `SELECT`, 341 `INSERT`, 11 `SAVEPOINT`, 11 `RELEASE`),
  approximately 0.067 statements per security and bounded by fixed batches rather
  than security/component count. Baseline RSS was 81.227 MiB, peak sampled RSS
  was **549.125 MiB**, and the factor-run increase was 467.898 MiB. PostgreSQL
  reported 20 temporary files / **120,815,616 bytes**; the bounded temp I/O was
  attributable to an ordered wide-row source sort under the development host's
  work-memory setting.
- Representative `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` evidence covers exact
  source selection, source-presence aggregation, and component-lineage
  verification. Source reads use bitmap index paths; lineage uses the component
  FK/index path with a bounded-side scan; none of the three measured plans has a
  Cartesian expansion. Detailed measurements and limitations are frozen in
  `docs/phase-2.7-scale-sanity.md`.
- **Reproduced and corrected structural scale defects:** the runner now validates
  definitions once while resolving/publishing fixed 250-security batches inside
  one outer savepoint; publishes the industry subject once; performs parameter-
  safe 2,000-row Core inserts; scopes publication reads to the current subjects;
  and verifies lineage through joins instead of unbounded ID parameter lists.
  Registry-major/caller subject order is reconstructed exactly after batching.
- Large requests retain scalar values and immutable fingerprints in memory while
  deferring only the two large factor JSON payloads. The payloads remain
  lazy-accessible, fingerprints cover their exact expected content, database
  immutability remains enforced, and authoritative evidence stays in relational
  component rows. The v1 top-level component-ID provenance is retained so the
  performance change does not alter historical fingerprints under the frozen
  runner identity.
- An initial 1,000-security batch run under allocation tracing and a subsequent
  unbounded-result-payload run exceeded this approximately 2 GiB host. Their exact
  random databases were identified and force-dropped before retry. The final
  bounded run completed and performed normal cleanup. These failures and the
  resulting design changes are retained in the scale report rather than hidden.
- Added a forced multi-batch regression proving result order, complete lineage,
  payload deferral, and lazy provenance access. Focused runner/lifecycle tests:
  **12 passed**. Full regression: **722 passed, 8 existing warnings, zero
  failures**. `git diff --check` passed. Alembic current/head remains **`p27001`**;
  this checkpoint has no model or migration change.
- The 22-minute development-host runtime is not a production SLA, and the
  synthetic workload is not a provider-production claim. Future operational
  optimization may evaluate staging/COPY, work memory, or faster hardware, but
  must preserve atomicity, immutability, PIT, build identity, quality, coverage,
  and provenance. No Phase 2.2, 2.4, 2.5, or 2.6 formula/semantic was changed.
- Next: **2.7.10 — final implementation audit, fresh migration lifecycle,
  documentation/README handoff, and only reproduced defect fixes**.

### Checkpoint 2.7.8 — Full synthetic factor integration

Status: **COMPLETE**.

- Starting checkpoint `2b7a87146724117794d5d9af1f394d86b34bf6d3` was equal
  to `origin/master` with a clean worktree.
- Added one deterministic PostgreSQL lifecycle for a fictional company and five
  same-node Semiconductor Infrastructure peers at two research timestamps. Each
  snapshot pins separate fundamental, estimate, and market datasets/calculation
  builds plus the exact membership, peer, industry, factor, policy, definition,
  and normalization identities in one content-addressed manifest.
- The test publishes **30 frozen upstream feature IDs** for the six securities,
  then runs the real Phase 2.6 `PeerFeatureRunner` for all ten peer percentiles,
  the real `IndustrySnapshotRunner` for all eight industry metrics, and the Phase
  2.7 `FactorRunner` for the exact nine factor IDs. No peer result, industry
  metric, subfactor, component decision, or final factor value is injected.
- T1 models broad strength: strong revenue growth/acceleration, margins, cash
  confirmation, positive revisions, leadership, and industry breadth with low
  observed fragility. T2 is a distinct later source build with weaker or negative
  growth, margin, cash, revision, market, and industry evidence plus higher
  fragility. Exact golden T1 -> T2 values are:

  | Factor | T1 | T2 |
  | --- | ---: | ---: |
  | `growth_v1` | `0.9075` | `0.0625` |
  | `growth_acceleration_v1` | `0.755` | `-0.32666666666666666666666666666666666667` |
  | `profitability_v1` | `0.96428571428571428571428571428571428572` | `0.15571428571428571428571428571428571428` |
  | `margin_expansion_v1` | `0.8475` | `-0.33375` |
  | `cash_flow_quality_v1` | `0.885` | `-0.21666666666666666666666666666666666667` |
  | `estimate_revision_v1` | `0.885` | `-0.4025` |
  | `market_leadership_v1` | `0.74` | `-0.47833333333333333333333333333333333333` |
  | `industry_strength_v1` | `0.66562499999999999999999999999999999997` | `-0.48784722222222222222222222222222222221` |
  | `market_fragility_v1` | `-0.84` | `0.70` |

- Both snapshots produce all nine `VALID/FULL` factors at exact weight coverage
  one. Every result is correctly `DEVELOPMENT` because the provider qualification
  is synthetic. All **47 expected component decisions** are included and
  relationally linked: ten carry authoritative peer-result FKs and eight carry
  exact industry-metric FKs. Component contributions sum to their final values in
  the frozen 38-digit Decimal context, and every expected subfactor is persisted.
- PIT is exercised with an exact-build revenue correction one microsecond after
  T1 and an extreme available wrong-build row. Neither enters the direct factor,
  peer distribution, or industry snapshot; the selected source FK is the exact
  admitted T1 row and both exclusion diagnostics remain inspectable. Estimate
  targets, exact-T market/estimate sources, quarterly fiscal identity, group/node,
  build, quality, and source admissions are preserved through final lineage.
- Exact same-build reruns preserve source, peer, industry, factor, subfactor, and
  component identities. The later T2 build creates nine new artifacts without
  changing T1. A final factor-only T1 replay after T2 publication returns the
  original IDs, fingerprints, values, and execution timestamps.
- **Reproduced and fixed a Phase 2.7 defect:** included component selection
  provenance persisted a global `any_count`. Merely appending an ineligible row
  from a later source build could therefore change an old factor fingerprint.
  Persisted provenance now excludes only that non-semantic global count while
  retaining exact-build, future, wrong-build, and exact-timestamp diagnostics.
  No Phase 2.2, 2.4, 2.5, or 2.6 implementation was changed.
- Validation: lifecycle **1 passed**; complete peer/industry suite **85 passed**;
  focused factor plus runner integration suite **208 passed**; full regression
  **721 passed, 8 existing warnings, zero failures**. `git diff --check` passed.
  Alembic current/head remains **`p27001`**; there is no model or migration change.
- This remains educational synthetic evidence. It validates orchestration and
  invariants, not commercial-provider quality, licensing, or production-scale
  readiness.
- Next: **2.7.9 — approximately 10,000-security scale/performance sanity with SQL
  count, runtime, peak memory, PostgreSQL temp I/O, index and Cartesian/N+1 audit**.

### Checkpoint 2.7.7 — Factor runner and complete registry

Status: **COMPLETE**.

- Starting checkpoint `500c6a3c5d4174a70589b30bb2cfc67c48b8964e` was equal
  to `origin/master` with a clean worktree.
- Added the explicit registry-backed `FactorRunner` API and the exact ordered
  nine-ID aggregate registry. Requests validate the canonical content-addressed
  `ResearchBuildManifest`, requested definition hashes, weight/normalization/
  coverage/formula/quality/runner versions, source registry contracts, exact
  source calculation builds, dataset references, factor admissions, timestamp,
  effective date, and subject scope before publication. Unknown or extra factor
  IDs remain rejected; no Phase 2.8 signal, rank, probability, or recommendation
  output exists.
- Source retrieval is batched by family. Fundamentals select the newest eligible
  exact-build quarterly row at or before T, preserve the inclusive 135-day rule,
  and require one anchor period across the factor. Estimate, market, and peer
  projections require exact T/period/build rows. Future and wrong-build rows are
  explicitly excluded without fallback. Estimate FQ1/FY1 target identity is
  coherent within each horizon.
- Peer inputs require the immutable projected `peer_v1` `FeatureValue` plus its
  exact authoritative `PeerRelativeResult`, target source `FeatureValue`,
  `PeerStatisticSnapshot`, and same-manifest `PeerGroupSnapshot`. Industry inputs
  require the exact factor-build/T/effective-date/node/group and link each
  `industry_v1` metric to its matching frozen statistic. Metric aggregation,
  source version, units, group, status, and lineage are validated.
- Quality remains factor-local. Each included direct component carries its exact
  source admission. Peer components propagate the weakest projected/result/
  statistic/group quality and retain all four admissions; industry components
  propagate the weakest metric/statistic/group quality. Excluded optional inputs
  affect coverage only. Synthetic peer evidence therefore downgrades only the
  consuming factor.
- Publication is one savepoint over the complete resolved batch. It stores the
  immutable factor value, every named subfactor, and every expected included or
  excluded component with relational source FKs, raw/normalized values, fixed and
  effective weights, contribution, coverage, quality, selection evidence, compact
  provenance, and a content fingerprint. Exact reruns preserve factor,
  subfactor, component IDs, fingerprints, values, and execution time; changed
  content at the same identity fails closed. A new manifest build creates a new
  artifact without mutating the old result.
- The calculator now assigns deterministic effective weights/contributions to
  source-admitted components even when an anchor or coverage gate makes the
  containing result missing. This satisfies the frozen relational rule that an
  included component has complete attribution while the failed subfactor/factor
  value and contribution remain null. Numeric factor semantics for valid results
  are unchanged.
- Added **10 runner tests** covering the exact nine-ID registry and 47 definition
  components, all eight security factor IDs plus the node-scoped industry ID,
  exact lineage, valid hierarchy arithmetic, future/wrong-build exclusion,
  manifest mismatch rollback, weakest quality, new-build isolation, request
  validation, and exact rerun idempotency. One peer test runs the actual frozen
  Phase 2.6 peer resolver/statistic/projection path before factor resolution; one
  industry test proves exact node/metric/statistic lineage.
- Focused factor plus runner integration suite: **207 passed**. Full regression:
  **720 passed, 8 existing warnings, zero failures**. `git diff --check` passed.
  Alembic current/head remains **`p27001`**; this checkpoint has no model or
  migration change. No frozen Phase 2.2, 2.4, 2.5, or 2.6 formula/runner was
  modified.
- Provider-quality gates remain unchanged. The valid peer and industry fixtures
  are synthetic/educational evidence and make no production-readiness claim.
- Next: **2.7.8 — deterministic PostgreSQL full synthetic lifecycle through
  frozen source features, peer/industry outputs, factor calculation, persistence,
  PIT/build/quality/coverage/provenance, version isolation, and idempotency**.

### Checkpoint 2.7.6 — Market and industry factor definitions

Status: **COMPLETE**.

- Starting checkpoint `fb5411ab19087138d427c1df449633268beeb46e` was equal
  to `origin/master` with a clean worktree.
- Added the exact `market_leadership_v1`, `industry_strength_v1`, and
  `market_fragility_v1` candidate definitions. Market Leadership preserves fixed
  Trend (35), Relative Strength (40), and Momentum (25) subfactors. Industry
  Strength is correctly `INDUSTRY_NODE` scoped with fixed 4/12 Fundamental,
  Estimate, and Market shares. Market Fragility is security-scoped and explicitly
  `RISK_HIGHER_WORSE`: **+1 means more observed fragility**, never positive
  investment evidence.
- Market inputs reference exact frozen `market_v1` daily source IDs and the
  actual `peer_return_63d_percentile` `peer_v1` output. Industry components use
  `INDUSTRY_SNAPSHOT_METRIC` lineage and all eight actual `industry_v1` metric
  keys with their output units. No return, moving average, risk statistic,
  membership, peer percentile, or industry aggregate is recalculated here.
- Leadership excludes the legacy industry basket return, embedded/duplicative
  return horizons, directionless volume and all risk inputs. Fragility uses only
  realized volatility, maximum drawdown and volatility regime; the drawdown sign
  is reversed exactly once and positive raw drawdown is invalid. `vol_trend_21`
  and future qualitative risk concepts remain excluded.
- Added **34 tests** for the exact three-ID boundary; all mappings, scopes,
  orientations, source kinds/versions/units/frequencies, subfactors, weights,
  anchors, directions and transforms; actual peer/industry registry linkage;
  immutable relational definition registration; neutral and positive golden
  arithmetic; complete component attribution; every required anchor; optional
  stale/missing evidence; invalid industry evidence; factor-specific market and
  estimate-derived industry quality; fragility orientation; drawdown domain; and
  explicit exclusions. The recurring 4/12 hierarchy is asserted under the exact
  38-digit Decimal contribution sum rather than display-rounded arithmetic.
- Focused Phase 2.7 suite: **197 passed**. Combined factor/market/industry
  regression: **243 passed**. Full regression: **710 passed, 8 existing warnings,
  zero failures**. No schema, migration, Phase 2.4 formula/runner, or Phase 2.6
  statistic/snapshot change was made; Alembic remains **`p27001`**.
- Exact PIT/build selection and persisted factor result lineage remain reserved
  for the isolated registry-backed runner. No source/provider gate is weakened.
- Next: **2.7.7 — complete nine-ID registry and isolated batch `FactorRunner`
  with exact build/PIT source resolution, persistence, lineage, and idempotency**.

### Checkpoint 2.7.5 — Estimate Revision factor definition

Status: **COMPLETE**.

- Starting checkpoint `aeebc9a1286798fe4ed4144fffe407d335060885` was equal
  to `origin/master` with a clean worktree.
- Added the exact `estimate_revision_v1` candidate definition with fixed
  `absolute_revisions` (70) and `peer_revisions` (30) subfactors. It references
  six actual inactive `estimates_v1` outputs and three actual `peer_v1` outputs,
  all at the frozen 30-calendar-day horizon, with exact FQ1/FY1 mappings, weights,
  anchors, units, transforms, and version metadata.
- Required evidence is the FQ1 EPS scaled-magnitude feature, FQ1 revenue
  percentage-magnitude feature, and their two peer-magnitude percentiles. EPS and
  revenue breadth, both FY1 magnitude checks, and peer EPS breadth are optional
  only under the frozen coverage gates. Revision counts, absolute EPS changes,
  and correlated 7/90-day horizons are deliberately excluded.
- Frozen Phase 2.5 meanings are preserved: magnitude zero remains valid numeric
  zero; no directional revisions leaves breadth missing; missing prior consensus
  remains missing source evidence with the upstream reason retained. The
  deterministic no-revision fixture therefore produces `VALID/REDUCED`, value
  zero and exact 74% weighted coverage rather than manufacturing zero breadth.
- Synthetic estimate admission produces `DEVELOPMENT` only for the estimate
  factor calculation. A separately calculated fundamental factor retains its own
  admitted quality. No production-provider qualification is claimed or implied.
- Added **20 tests** for the exact single-ID boundary, actual Phase 2.5/peer
  registry metadata, relational definition lineage and idempotency, magnitude and
  breadth transforms, deterministic golden value `0.346`, all four required
  anchors, missing prior consensus, no revisions, reduced FY1 coverage, synthetic
  quality isolation, out-of-domain breadth, and the explicit exclusion list.
- Focused Phase 2.7 suite: **163 passed**. Combined factor/estimate/peer regression:
  **502 passed**. Full regression: **676 passed, 8 existing warnings, zero
  failures**. No schema, migration, consensus, estimate-feature, peer-statistic,
  or upstream formula change was made; Alembic remains **`p27001`**.
- PIT/build-coherent retrieval of exact estimate profiles/datasets/targets and
  persisted result lineage remain reserved for the isolated factor runner. This
  checkpoint consumes resolved frozen outputs only.
- Next: **2.7.6 — exact Market Leadership, Industry Strength, and Market
  Fragility definitions with frozen Phase 2.4/2.6 source mappings**.

### Checkpoint 2.7.4 — Fundamental factor definitions

Status: **COMPLETE**.

- Starting checkpoint `7a0b1593c5241e76b12dce25c71335df8392f425` was equal
  to `origin/master` with a clean worktree.
- Added the five exact candidate definitions `growth_v1`,
  `growth_acceleration_v1`, `profitability_v1`, `margin_expansion_v1`, and
  `cash_flow_quality_v1`. Their ordered subfactors, fixed weights, required
  anchors, normalization knots, directions, orientations, and common policy
  versions are immutable registry content.
- Direct components reference the 13 implemented `ResearchFeature.version = v1`
  quarterly source IDs with their exact units. Six peer-relative components
  reference six actual `peer_v1` registry outputs and inherit the frozen peer
  unit/frequency/version contract directly. Definitions remain inactive
  `CANDIDATE` factors in dedicated factor storage; nothing was added to or
  activated in `ResearchFeature`.
- The factor layer does not recalculate any fundamental or peer metric. In
  particular, Growth excludes QoQ growth and acceleration; Profitability excludes
  cash-flow metrics; Margin Expansion is distinct from margin level; Cash Flow
  Quality excludes the FCF currency amount; and missing upstream FCF margin or
  conversion remains explicit missing evidence rather than fabricated zero.
- Added **44 tests** for the exact five-ID boundary; every component mapping,
  source version/unit/frequency, global ordinal, subfactor and component weight,
  required anchor, direction and transform; immutable registry data; idempotent
  relational definition registration; exact neutral and deterministic golden
  arithmetic; complete component attribution; every required-anchor omission;
  every optional-component omission; FCF missingness; and factor-specific peer
  quality isolation. Golden full-coverage values are exactly `0.40`, `0.30`,
  `0.38`, `0.25`, and `0.40` for the five families' fixed fixtures.
- Focused Phase 2.7 suite: **143 passed**. Broader factor/fundamental/peer
  regression: **199 passed**. Full regression: **656 passed, 8 existing warnings,
  zero failures**. No model or migration change was needed; Alembic remains
  **`p27001`**. Frozen Phase 2.2/2.6 source implementations were not modified.
- This checkpoint defines and calculates resolved inputs only. PIT/build source
  resolution and persisted result lineage remain explicitly reserved for the
  isolated runner checkpoint; it makes no provider-production claim.
- Next: **2.7.5 — exact `estimate_revision_v1` definition, frozen Phase 2.5
  mappings, and estimate-specific missingness/quality tests**.

### Checkpoint 2.7.3 — Coverage and hierarchical factor calculator

Status: **COMPLETE**.

- Starting checkpoint `5601f5f0d8f7be07e7842dd7d86efa32ed65eaed` was equal
  to `origin/master` with a clean worktree.
- Added a source-agnostic, side-effect-free hierarchical factor calculator over
  immutable factor definitions and resolved component inputs. It enforces the
  exact `factor_coverage_v1`, `hierarchical_weighted_mean_v1`,
  `factor_quality_v1`, and `factor_fixed_piecewise_v1` version pins before doing
  arithmetic.
- Every required anchor must be admitted. Each named subfactor requires at least
  exact 60% expected weight; total factor coverage requires at least exact 70%.
  Optional missing/invalid/status- or quality-inadmissible evidence is excluded,
  never changed to zero, and can produce `VALID/REDUCED` only when all gates pass.
  Reweighting occurs only within its original semantic subfactor; fixed subfactor
  shares never drift toward whichever evidence happened to be available.
- Implemented deterministic `VALID | MISSING | INVALID` and reason precedence,
  including `NO_COMPONENTS`, required timestamp/period/build/staleness/status/
  quality/missing decisions, subfactor versus total coverage, required upstream
  invalidity, and global fail-closed handling for a source claiming `VALID` with
  non-finite or out-of-domain content. `ESTIMATED` is not admitted.
- Component effective weights and contributions are calculated with 38-digit
  Decimal arithmetic and sum to the reported factor value. Valid numeric zero is
  retained distinctly from missing. Result objects retain every expected
  component decision, each subfactor's weights/coverage/value/contribution, total
  count/weight coverage, coverage state, orientation, and quality.
- Quality is the weakest factor-specific admission plus every included input.
  Excluded optional evidence does not contaminate quality, and separate factor
  calculations do not inherit unrelated synthetic/development inputs.
- Added **24 calculator tests** covering all-present hierarchy, missing required
  anchors, optional missingness, exact/below 60% and 70% boundaries, valid zero,
  required and optional invalid inputs, `VALID` non-finite/domain integrity
  failures, `ESTIMATED`, reason precedence, synthetic/development quality,
  factor-specific quality isolation, excluded-input isolation, request-level
  version/input errors, and higher-is-worse fragility orientation.
- Focused Phase 2.7 suite: **99 passed**. Full regression: **612 passed, 8 existing
  warnings, zero failures**. Alembic current/head remains **`p27001`**; no upstream
  formula, runner, migration, or Phase 2.8 behavior changed.
- Next: **2.7.4 — exact fundamental factor definitions and frozen-source
  mappings for Growth, Growth Acceleration, Profitability, Margin Expansion, and
  Cash Flow Quality**.

### Checkpoint 2.7.2 — Versioned normalization engine

Status: **COMPLETE**.

- Starting checkpoint `0c9aca94a34eba647416d0348ac6b6293447adb6` was equal
  to `origin/master` with a clean worktree.
- Added isolated `factor_fixed_piecewise_v1` normalization using a 38-digit
  `ROUND_HALF_EVEN` Decimal context. The engine implements only the frozen bounded
  three-knot piecewise transform, `2p-1` percentile/breadth mapping, bounded
  identity, and the single explicit drawdown `NEGATE_THEN_PIECEWISE_LINEAR`
  transform.
- Inputs and knots must be finite exact Decimal-compatible values; binary floats,
  missing values, malformed/out-of-order knots, out-of-domain percentiles,
  out-of-range identity values, positive maximum drawdowns, unknown methods,
  undeclared parameters, unknown orientation, and unknown normalization versions
  fail explicitly. Saturation occurs only at declared piecewise anchors; percentile
  inputs are never clipped.
- Factor orientation is validated metadata and never changes the numeric sign.
  In particular `RISK_HIGHER_WORSE` preserves a positive high-fragility value
  rather than silently turning it into positive investment evidence.
- Added **52 exact normalization tests** covering below/at/between/above knots,
  non-zero neutral points, negative and zero values, 38-digit precision,
  NaN/infinity/float rejection, `PCTL` endpoints and interior values, identity
  bounds, drawdown direction, dispatch/version parameters, and orientation.
- Focused Phase 2.7 suite: **75 passed**. Full regression: **588 passed, 8 existing
  warnings, zero failures**. Alembic current/head remains **`p27001`**; no schema,
  upstream formula, source runner, or Phase 2.8 behavior changed.
- Next: **2.7.3 — required anchors, hierarchical coverage, status/reason
  precedence, within-group weighting, and quality propagation**.

### Checkpoint 2.7.1 — Factor definitions and dedicated storage

Status: **COMPLETE**.

- Starting checkpoint `b015943a84d96f402830b4d7d2e6fcee10ebc9ed` was equal
  to `origin/master` with a clean worktree.
- Added dedicated SQLAlchemy models and migration `p27001` for canonical
  `ResearchBuildManifest` persistence, immutable versioned factor definitions,
  ordered definition components, the append-only future promotion hook, factor
  values, subfactor values, and one relational lineage row for every expected
  component decision. Factors are not stored as `ResearchFeature` or
  `FeatureValue` rows.
- Definition registration canonicalizes exact Decimal metadata, hashes the full
  ordered hierarchy, requires explicit anchors and coherent group weights, is
  idempotent for identical content, and fails rather than repairing a collision.
  Definition, component, manifest, promotion, value, subfactor, and lineage tables
  reject PostgreSQL `UPDATE` and `DELETE`.
- Database constraints and insert guards enforce one matching subject, manifest
  timestamp, expected definition count/weight, valid status/value/coverage/quality
  combinations, finite bounded values, exact definition-component ownership, and
  source-kind/key/version/unit/frequency/subject linkage. Included peer components
  require their authoritative `PeerRelativeResult`; industry components bind the
  exact industry snapshot group.
- Added 23 focused PostgreSQL tests for definition uniqueness/conflict/versioning,
  manifest identity, definition/value/lineage immutability, exact zero retention,
  included and excluded component provenance, quality/status constraints,
  non-finite/out-of-range rejection, build timestamp isolation, FK/source/definition
  integrity, and the empty-by-default append-only promotion hook.
- An initial PostgreSQL boolean cast in a model check was found by the full suite's
  SQLite metadata fixtures and replaced with an equivalent portable `CASE`
  expression before publication. The 62 directly affected frozen/factor tests
  then passed.
- Disposable PostgreSQL verification passed fresh base → `p27001` → `p26008` →
  `p27001`, all seven factor tables, Alembic metadata parity, and all 23 focused
  tests. Persistent Alembic current/head is **`p27001`**.
- Full regression: **536 passed, 8 existing warnings, zero failures**. Frozen
  Phase 2.2/2.4/2.5/2.6 formulas and runners were not changed.
- Next: **2.7.2 — versioned Decimal normalization engine and exact boundary
  tests**.

### Checkpoint 2.7.0 — Freeze factor specification

Status: **COMPLETE**.

- Starting checkpoint `f51f1741488a74ce68db7ce3336382b4df03eed8` was equal
  to `origin/master` on a clean worktree.
- Baseline reproduced exactly: **513 passed, 8 existing warnings, zero failures**;
  Alembic current/head **`p26008`**.
- Read the complete Phase 2.7 architecture review, Phase 2.6 specification/freeze
  record, Foundation/Phase 2.5 freeze record, project state, and README. Verified
  every mapped source ID directly against the implemented fundamental, market,
  estimate, peer, and industry registries.
- Created the authoritative concise implementation contract:
  [phase-2.7-factor-spec.md](phase-2.7-factor-spec.md). It freezes the exact nine
  candidate factor IDs; component/source mappings; required anchors; subfactor and
  component weights; versioned transforms; 60%/70% coverage gates; orientation;
  quality isolation; PIT/build identity; dedicated relational storage;
  immutability; provenance; publication; scale; and test contracts.
- No application code, model, migration, factor definition/value, source formula,
  provider qualification, signal, rank, probability, recommendation, or Phase 2.8
  work changed in this checkpoint.
- Next: **2.7.1 — factor definitions, dedicated storage, manifest linkage,
  migration, immutability, and relational-integrity tests**.

The preceding architecture review remains approved:

- Phase 2.7 architecture review complete.
- Phase 2.7 implementation started only through the frozen specification checkpoint.
- Approved design:
  [phase-2.7-factor-architecture-review.md](phase-2.7-factor-architecture-review.md).

### Independent Phase 2.6 freeze review

Status: **FROZEN**.

- Independently reviewed baseline commit
  `7c689f4a44d3f3b66d97ad09363ee8458661c56f` on `master`, initially equal to
  `origin/master` with a clean worktree. The specification, architecture review,
  internal audit, code, models, migrations, tests, lifecycle, and scale verifier
  were reviewed directly; the internal audit was not assumed correct.
- Reconfirmed latest-known-before-effective, end-exclusive PIT membership;
  versioned taxonomy/node hierarchy; exact same-sub-industry resolution; no hidden
  fallback; target exclusion; separate 5-member/5-observation/60% gates; explicit
  feature exclusions; finite Decimal median/delta/ascending-midrank arithmetic;
  weakest-input quality; survivorship failure; source build isolation; immutable
  publication; and historical industry membership.
- Reproduced and fixed three freeze-blocking defects: source rows could claim
  provider coverage outside the runtime qualification; member evidence was not
  relationally constrained to the snapshot's exact sealed datasets; and persisted
  peer provenance lacked the canonical build-manifest payload. The regressions
  fail on the reviewed baseline and pass after the fixes.
- Added `p26008`. Group/member rows now carry the exact classification,
  eligibility, and identity dataset IDs. Composite foreign keys bind members to
  the group's datasets and bind every evidence ID to its dataset membership. The
  migration safely backfills existing immutable rows from their pinned provenance.
- Final 10,001-security PostgreSQL 16.15 proof produced 10,001 included members,
  10,000 non-target peers, and 10,000 valid observations in **52 SQL statements**,
  **69.282 seconds**, and **248.163 MiB** peak traced Python memory. Four window
  plans completed in **66.804 / 65.546 / 57.015 / 28.770 ms** with no temporary
  I/O. This is a correctness/query-shape proof on the small development host, not
  a production latency benchmark.
- Disposable migration verification passed fresh base → `p26008`, the populated
  `p26007` → `p26008` backfill, every earlier Phase 2.6 boundary, metadata parity,
  concurrent one-winner publication, and all focused tests on fresh PostgreSQL.
- Validation: **82 peer tests passed**; full regression **513 passed, 8 existing
  warnings, zero failures**; Alembic current/head **p26008**; `alembic check` and
  `git diff --check` clean.
- Git history confirms Phase 2.2 fundamental, Phase 2.4 market, and Phase 2.5
  estimate formula runners were unchanged. Phase 2.6 reads their immutable outputs.
- Licensed historical classification, qualified primary-equity and delisted-name
  coverage, production source providers, and strict accounting provenance remain
  external gates. Synthetic evidence remains `DEVELOPMENT`; the runtime fails
  closed for stronger claims. Peer percentiles and industry snapshots are
  descriptive context, not buy/sell signals.
- Freeze decision and evidence:
  [PHASE_2_6_FREEZE_REVIEW.md](PHASE_2_6_FREEZE_REVIEW.md).
- Phase 2.7 architecture review is **COMPLETE** and implementation is **NOT
  STARTED**.

### Checkpoint 2.6.7 — Implementation audit, migration/scale proof, and handoff

Status: **COMPLETE**.

- Starting checkpoint `e80111241c0866d6777aa34536504a7c8327aa5d` was equal
  to `origin/master` with a clean worktree after checkpoint 2.6.6. The earlier
  interrupted 2.6.3 test report was stale: its invalid short manifest hash and
  publish-then-delete missing-value fixture had already been corrected without
  weakening immutable `FeatureValue` history; the recovered focused file passed.
- Audited the implementation against the frozen 2.6 specification, not merely its
  existing tests. Corrected global primary-security ambiguity, mandatory row-level
  source admission and coverage validation, exact effective-date projection reads,
  non-finite numeric handling, and missing relational source/member evidence
  guards. Nine new adversarial regressions cover each reproduced defect.
- Replaced Phase 2.6's Python-side classification, eligibility, and identity head
  reduction with PostgreSQL window-ranked, dataset-pinned queries. Frozen shared
  foundation resolvers and Phase 2.2/2.4/2.5 formulas were not changed. Member and
  observation publication is batched at 1,000 rows to stay below PostgreSQL's
  parameter ceiling.
- Added migration `p26006`, binding observations/results to the correct included
  member, security, source feature, calculation build, target observation, and
  `FeatureValue`. Added `p26007`, which preflights and guards member evidence
  security/taxonomy/release/policy/company consistency and adds database checks for
  finite peer/industry values and frozen VALID thresholds.
- Added `scripts/verify_peer_scale.py`. A disposable PostgreSQL 16.15 run with
  **10,001 securities** and two evidence versions per classification, eligibility,
  and identity key produced 10,001 included members, 10,000 non-target peers and
  10,000 valid observations in **52 SQL statements**, **63.914 seconds**, and
  **245.017 MiB** peak traced Python memory. Four window plans returned 10,001 rows
  each in **66.862 / 65.006 / 56.669 / 28.556 ms**, with no temporary I/O.
- Disposable migration verification passed fresh base → `p26007`, `p26006` and
  `p26005` hardening boundaries, every earlier Phase 2.6 downgrade/upgrade boundary,
  populated `p26004` count backfill, concurrent one-winner publication, metadata
  parity, and all peer tests on the rebuilt database.
- Tests: **77 peer tests passed**; full regression **508 passed, 8 existing
  warnings, zero failures**. Persistent Alembic current/head are **p26007** and
  `alembic check` reports no upgrade operations; the read-only member-integrity
  preflight found zero inconsistent existing rows. `git diff --check` passes.
- Full findings, corrections, scale plans, limitations, and review targets are in
  [phase-2.6-implementation-audit.md](phase-2.6-implementation-audit.md).
  Production classification/universe/security-master and source-feature providers,
  coverage, survivorship, licensing, and rights remain externally gated. A source
  `FeatureValue` without its own adequate admission is intentionally excluded.
- Next: **independent Phase 2.6 QC/architecture review**. Phase 2.6 is **NOT YET
  FROZEN** and Phase 2.7 is **NOT STARTED**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Complete Phase 2.6 implementation audit$'`.

### Checkpoint 2.6.6 — Full synthetic PostgreSQL lifecycle

Status: **COMPLETE**.

- Starting checkpoint `0b480f5e1e08d43a3cbb340342d04442490f7f38` was equal
  to `origin/master` with a clean worktree.
- Added one coherent A–G synthetic lifecycle over two fictional GICS-style
  releases and two independently sealed evidence builds. It covers the stable
  sub-industry leaf being reparented in R2, F's future-known June 1 entry, C's
  June 15 exit, D's future-known July 1 delisting despite its present-day inactive
  flag, E's July 10 PIT correction effective from May 1, B's future-known
  September 1 exit, and G's July 15 re-entry.
- The timestamp matrix proves membership before/at every boundary, including
  microsecond-before versus exact announcement/correction knowledge, effective
  dates, R1 versus R2 hierarchy/path, deterministic permanent-security-ID order,
  five-peer admission, no implicit hierarchy fallback, and a separately identified
  explicit industry-level policy. Later evidence cannot enter or extend the first
  sealed corpus, and exact replay of its June 10 build reuses every original group
  member, fingerprint, ID, and creation timestamp.
- The canonical June 10 pipeline publishes target `0.30` against peers
  `[0.10, 0.20, 0.40, 0.50, 0.30]`, proving median `0.30`, delta `0`, `L=2`,
  `E=1`, `N=5`, and ascending midrank `0.5`. A second negative/zero/tie
  cross-section proves ordinary numeric ordering and positive breadth. Peer and
  all-member industry outputs share the exact persisted source distributions while
  retaining their distinct target and count semantics.
- The same integrated run proves membership retention for one never-published
  feature and one value stale by 135 days plus one microsecond; exactly 135 days
  remains eligible. It covers target missing/invalid precedence, the exact 60%
  coverage state, all-member treatment of an invalid anchor, exact-T estimate and
  market inputs, immutable upstream source rows, weakest synthetic quality, and
  unit, accounting, currency, share-basis, dataset, provider, and build exclusions.
- Walked a projected `FeatureValue` through its target-relative result, statistic
  snapshot, all observations, group/members, classification evidence, taxonomy
  release, sealed datasets, source values, manifest hash, provider admission, and
  raw-inventory hash. Exact reruns preserve every output ID, value, status,
  provenance, fingerprint, and timestamp. An injected outer-transaction failure
  rolls back group, members, statistics, observations, relative results, industry
  metrics, and projected features together.
- Fixed one integration defect exposed by the lifecycle: the general replay gate
  honored a structured development override, but the peer-specific taxonomy/
  classification capability gate rejected it unconditionally. Normal execution
  remains fail-closed. Only a named `DevelopmentOverride` may admit a current-only
  or incomplete provider; the missing peer capabilities are now persisted as
  limitations and force `DEVELOPMENT` quality.
- Disposable PostgreSQL now also races two independent sessions publishing the
  same snapshot and proves both converge on the one immutable identity. It still
  validates fresh base → `p26005`, every Phase 2.6 downgrade/upgrade boundary, a
  populated immutable `p26004` → `p26005` reason-count backfill, metadata parity,
  and the complete peer suite.
- Tests: **5 new lifecycle tests**, **68 total peer tests**, and full regression
  **499 passed, 8 existing warnings, zero failures**. Existing 50-member query-count
  tests, exact 5/60% boundary tests, migration lifecycle, and all frozen Phase 2.2,
  2.4, and 2.5 formula/regression tests remain green.
- No migration or model change was needed. Persistent Alembic current/head remain
  **p26005** and `alembic check` reports no upgrade operations. Production
  classification, survivorship, market, and estimate use remains externally
  provider/rights gated; the acceptance data and all its results are explicitly
  synthetic `DEVELOPMENT` evidence.
- Next: **2.6.7 — independent implementation audit, defect regressions,
  migration/scale checks, and review handoff**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Add Phase 2.6 synthetic peer lifecycle$'`.

### Checkpoint 2.6.5 — Exact industry snapshot metrics

Status: **COMPLETE**.

- Starting checkpoint `3e34f5f8ab187e3b80a8a78e8fa6917e3723f582` was equal
  to `origin/master` with a clean worktree after the recovered 2.6.4 work was
  completed and pushed.
- Added the exact eight-entry immutable `industry_v1` registry: five medians and
  three positive breadth measures over the frozen fundamental, estimate, and
  market source features. Industry snapshots are direct formal-node results, not
  fabricated securities or generic `FeatureValue` rows. Breadth is strictly
  `value > 0` over valid observations, so zero and missing values are not positive.
- Added `IndustrySnapshotRunner`. It resolves the canonical PIT sub-industry once,
  includes every eligible member including the anchor, shares one authoritative
  source distribution between metrics with the same source, and preserves exact
  manifest, provider, dataset, calculation-version, compatibility, staleness, and
  weakest-quality gates from peer statistics. A formal node needs five valid
  observations and 60% member coverage; the target-relative requirement for five
  *non-target* peers does not incorrectly reject a valid five-member industry.
- Added a target-independent `PeerStatisticCalculator.calculate_group()` path.
  Existing target-relative behavior remains unchanged and still publishes the
  group distribution plus target result atomically. A group snapshot that is
  `MISSING / INSUFFICIENT_PEERS` only because it has fewer than six total members
  can still support an all-member industry metric when its independent 5/60%
  thresholds pass; all other group validity failures remain fail-closed.
- Added immutable `IndustrySnapshotMetric` persistence linked relationally to its
  exact peer-group and source-statistic snapshots. Each result stores total,
  valid, excluded, missing, invalid, stale, and other-excluded counts; coverage;
  median or breadth numerator/denominator/value; status/reason; weakest quality;
  admission; source-policy/statistic fingerprints; evidence IDs; and an immutable
  content fingerprint. Exact reruns reuse identity and timestamps, while changed
  content conflicts and rolls back the requested batch atomically.
- Migrations `p26004` and `p26005` respectively add the industry metric table and
  its explicit exclusion-category counts. The second is a forward migration so an
  already-applied `p26004` development schema is never silently rewritten; it
  safely reconstructs any existing category counts from immutable statistic
  observations before making the columns non-null. PostgreSQL guards enforce
  group/statistic consistency and reject UPDATE/DELETE.
- Tests: **10 new industry tests**, **63 total peer tests**, and full regression
  **494 passed, 8 existing warnings, zero failures**. Coverage includes the exact
  registry and all eight values, target inclusion, shared source distributions,
  target independence, five-observation and 60% boundaries, retained missing and
  stale membership with reason counts, relational guards, immutability,
  manifest/allowlist failures, idempotency, atomic conflict rollback, and bounded
  query count.
- Persistent Alembic current/head are **p26005** and `alembic check` reports no
  upgrade operations. Disposable PostgreSQL passed fresh base → `p26005` →
  `p26004` → `p26005` → `p26003` → `p26005` → `p26002` → `p26005` → `p26001`
  → `p26005` → `fnd006` → `p26005`, a populated `p26004` count-backfill proof,
  exact schema checks, metadata comparison, and all 63 peer tests.
- No Phase 2.2, 2.4, or 2.5 formula/runner behavior changed. The complete synthetic
  lifecycle, final Phase 2.6 audit, factors, signals, rankings, backtesting, and
  Phase 2.7 remain unimplemented. Production taxonomy, classification,
  survivorship, market, and estimate inputs remain externally provider/rights
  gated; synthetic fixtures remain `DEVELOPMENT`.
- Next: **2.6.6 — full synthetic PostgreSQL lifecycle/integration scenario**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Add Phase 2.6 industry snapshot metrics$'`.

### Checkpoint 2.6.4 — Exact peer feature registry + isolated runner

Status: **COMPLETE**.

- Recovered `master` at pushed checkpoint
  `2c5330135e22a8b4a5ef876e1497b55d893b029d`. The interrupted working tree
  contained a coherent untracked ten-feature registry and a small statistics
  provenance adapter; both were preserved, reviewed against the frozen spec, and
  completed rather than discarded.
- Added the exact ten-entry immutable `peer_v1` registry. Every entry is numeric,
  inactive for legacy runner scans, snapshot-frequency, unit `ratio`, ascending
  raw-value order, `investment_direction=NONE`, canonical same-sub-industry policy,
  `peer_stats_v1`, and an explicit frozen source-feature/build/compatibility map.
  Registration is idempotent and rejects conflicting metadata or extra `peer_v1`
  feature IDs without repairing or overwriting existing rows.
- Added the isolated `PeerFeatureRunner`. It resolves canonical membership once,
  delegates all eligibility/median/delta/midrank work to the authoritative 2.6.3
  calculator, and projects only the requested result percentile. It does not
  recalculate source formulas, broaden the taxonomy level, activate peer features,
  or enter the fundamental/market legacy scans.
- Runner manifests must pin `peer_feature_runner_v1`, every requested output
  registry version, every exact source feature/calculation/dataset identity,
  source-provider sets, and the existing source-policy fingerprints. Output
  calculation identity is build-qualified as
  `peer_v1:<manifest_hash>:classification_sub_industry_v1:1`; exact-T lookup never
  carries an earlier projection forward.
- Each projected `FeatureValue` references its authoritative
  `PeerRelativeResult`, statistic snapshot, group snapshot, target source value,
  source build and policy fingerprints. Status/reason and weakest quality
  propagate unchanged. Synthetic membership/source inputs remain `DEVELOPMENT`.
  Exact reruns preserve IDs, values, provenance and execution timestamps; changed
  content conflicts under the frozen immutable publication contract. Requested
  batches roll back atomically on a projection conflict.
- Added narrow adapters for the already-frozen upstream provenance shapes:
  Phase 2.5 estimate definition/dataset fields and Phase 2.4 list-based TRI price
  basis. Dataset-ID-backed estimate evidence remains exact-build checked; no
  upstream formula, runner, or historical row was changed.
- Tests: **20 new runner/registry tests**, **53 total peer tests**, and full
  regression **484 passed, 8 existing warnings, zero failures**. Coverage includes
  an independent ten-ID/metadata contract, all ten outputs, exact midrank
  projection, real estimate/market provenance shapes, relational provenance walk,
  exact-T/build retrieval, idempotency, immutable conflict rollback, manifest and
  allowlist failures before publication, missing-never-published peer retention,
  weakest quality, registry tampering/extra-ID rejection, and non-UTC input
  normalization.
- No model or migration was needed. Persistent Alembic current/head remain
  **p26003** and `alembic check` reports no upgrade operations. Disposable
  PostgreSQL passed fresh base → `p26003` → `p26002` → `p26003` → `p26001` →
  `p26003` → `fnd006` → `p26003`, exact 15/12/15/9/15/0/15 table checks,
  metadata comparison, and all 53 peer tests.
- No Phase 2.2, 2.4, or 2.5 formula/runner behavior changed. Industry aggregates,
  the complete synthetic lifecycle, final Phase 2.6 audit, factors, signals,
  rankings, backtesting, and Phase 2.7 remain unimplemented. Production taxonomy,
  classification, survivorship, market and estimate inputs remain externally
  provider/rights gated.
- Next: **2.6.5 — exact eight industry snapshot metrics**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Add Phase 2.6 peer feature registry and runner$'`.

### Checkpoint 2.6.3 — Peer statistics + feature eligibility

Status: **COMPLETE**.

- Starting checkpoint `80674cabd604e1614089971eee6d0c1c30d2a268` was equal
  to `origin/master` with a clean worktree.
- Added immutable `PeerStatisticSnapshot`, relational
  `PeerStatisticObservation`, and target-specific `PeerRelativeResult` evidence.
  Every retained group member receives an observation decision, so missing,
  stale, invalid, estimated, wrong-version, provider-unqualified, dataset-, unit-,
  accounting-, currency-, share-basis-, and period-incompatible values remain in
  the membership/coverage denominator with a machine-readable exclusion reason.
- Added manifest-pinned `SourceFeaturePolicy`. It freezes source family, registry
  version, calculation version, source dataset/build, provider set, units,
  frequency, and compatibility fingerprint. The statistic service rejects an
  unpinned or conflicting policy/build and never selects a generic latest version.
- Fundamental retrieval selects the greatest measured `period_end <= D`, then the
  latest `available_at <= T` within that period and never falls back when that row
  is missing, invalid, incompatible, or stale. Exactly 135 days is accepted and
  135 days plus one microsecond is `FEATURE_STALE`. Estimate and market families
  require an exact T/D snapshot and never carry an earlier value forward. Future
  FeatureValues are excluded.
- Added exact 38-digit `ROUND_HALF_EVEN` Decimal median, target-minus-peer-median,
  and ascending midrank `(L + 0.5E) / N`. Ties use exact canonical Decimal
  equality; negative and zero values keep ordinary numeric order. No float,
  winsorization, desirability inversion, mean, or z-score is used.
- Result gating is target status first, then five membership peers, five valid
  non-target observations, and 60% non-target coverage. Median/difference/rank are
  published only for a `VALID` target result. Group distributions and target
  leave-one-out results retain their distinct counts and coverage.
- Quality is the weakest group, target, and valid peer input. Feature-level
  provider version/qualification/synthetic identity is checked against the
  manifest admission. Excluded observations affect coverage but do not lower the
  numeric result's quality; synthetic inputs and attempted stronger claims remain
  `DEVELOPMENT`.
- Cross-section reads are set-based: one window-ranked feature selection and one
  grouped presence query for the whole membership set, followed by one bulk
  observation insert. A 50-peer regression stays within the fixed query bound and
  proves no query-per-peer path. Exact reruns reuse snapshot, observation, result,
  fingerprint, and creation identities; changed published content conflicts.
- Migration `p26003` adds the three statistic/result tables, relational
  group/member validation triggers, indexes, checks, and immutable UPDATE/DELETE
  guards. Persistent current/head is **p26003**; `alembic check` reports no
  upgrade operations. Disposable PostgreSQL passed fresh base → `p26003` →
  `p26002` → `p26003` → `p26001` → `p26003` → `fnd006` → `p26003`, exact
  15/12/15/9/15/0/15 table checks, metadata comparison, and focused tests.
- Tests: **15 statistics passed**, **33 total peer tests passed**; **171 focused
  foundation/frozen-runner regressions passed, 1 existing warning**; full
  regression **464 passed, 8 existing warnings, zero failures**. Coverage includes
  exact percentile examples/ties, negative/zero values, missing/invalid targets,
  5/5 and 60% boundaries, missing/stale retention, exact 135-day freshness,
  future-value exclusion, newest-row no-fallback, exact-T no-carry, unit/dataset/
  provider exclusions, weakest quality, provenance, idempotency, immutability,
  migration lifecycle, and bounded query count.
- No Phase 2.2, 2.4, or 2.5 formula/runner behavior changed. The ten canonical
  peer feature registrations and runner, industry aggregates, factors, signals,
  rankings, backtesting, and Phase 2.7 remain unimplemented.
- Next: **2.6.4 — exact ten-feature registry + isolated PeerFeatureRunner**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Add Phase 2.6 peer statistics and eligibility$'`.

### Checkpoint 2.6.2 — Deterministic peer resolver

Status: **COMPLETE**.

- Starting checkpoint `b233ba14bd17959f759cfd85c7f6e357f24539f8` was equal
  to `origin/master` with a clean worktree.
- Added the deterministic `PeerResolver` for every explicit classification
  policy. The canonical policy resolves the target's PIT sub-industry, excludes
  the target, orders members by permanent security ID, requires five non-target
  peers, and returns `INSUFFICIENT_PEERS` without silently broadening to industry,
  industry group, or sector. Broader levels remain separate explicit policies.
- Resolution pins and verifies the exact research timestamp, session/effective
  date, taxonomy release, peer-policy fingerprint, classification dataset,
  eligibility dataset, security-identity dataset, manifest hash, provider
  qualification versions, and application/schema/config/calculation versions.
  Only sealed datasets named by the canonical manifest can contribute evidence.
- Candidate classification, historical eligibility, security identity, taxonomy
  release, and node data are loaded in bounded set-based queries. Peer-member
  publication uses one bulk insert; a 50-peer regression remains below the query
  bound and proves there is no per-peer feature/identity query loop.
- Added immutable, target-independent `PeerGroupSnapshot` and relational
  `PeerGroupMember` evidence. Snapshots preserve every classification candidate,
  included members, excluded members and exact reasons, classification/universe/
  identity evidence IDs, provider admission, output quality, manifest pins, and a
  content fingerprint. Exact reruns reuse the same rows; changed content under the
  same identity conflicts instead of rewriting published history.
- Added sealed `SecurityIdentityEvidenceDatasetRow` input identity, including a
  database lifecycle guard. One primary security per company is admitted at the
  historical effective date; missing identity, duplicate primaries, inactive
  historical eligibility, dataset mismatch, taxonomy-release mismatch, and source
  qualification failures remain explicit rather than disappearing from the group.
- Added explicit provider capabilities for historical taxonomy structure and
  classification. Only the fictional synthetic survivorship fixture is admitted,
  only with explicit synthetic opt-in, and all such output remains
  `DEVELOPMENT`. No live source gained taxonomy, classification, or survivorship
  qualification; no proprietary classification data was added.
- Migration `p26002` adds three tables plus sealed-dataset and append-only guards.
  Persistent current/head is **p26002**; `alembic check` reports no upgrade
  operations. Disposable PostgreSQL passed fresh base → `p26002` → `p26001` →
  `p26002` → `fnd006` → `p26002`, exact 12/9/12/0/12 table checks, metadata
  comparison, and the focused suite. The configured research database was
  upgraded only and never downgraded.
- Tests: **18 peer-foundation/resolver passed**; **156 focused foundation/frozen-
  runner regressions passed, 1 existing warning**; full regression **449 passed,
  8 existing warnings, zero failures**. Coverage includes target exclusion, 5/4
  thresholds, no fallback, future-effective and future-known boundaries,
  departures, historical delisting, correction PIT, sealed-dataset and taxonomy-
  release isolation, provider fail-closed behavior, missing/duplicate identity,
  deterministic idempotency, relational provenance, immutability, and bounded
  query count.
- No Phase 2.2, 2.4, or 2.5 formula/runner behavior changed. Statistics, feature
  eligibility, peer-relative feature outputs, industry aggregates, factors,
  signals, rankings, backtesting, and Phase 2.7 remain unimplemented.
- Next: **2.6.3 — peer statistics + feature eligibility**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Implement Phase 2.6 deterministic peer resolver$'`.

### Checkpoint 2.6.1 — Taxonomy + membership models

Status: **COMPLETE**.

- Starting checkpoint `9fd35af552311bda740f11cf2236794513fbb21d` was equal
  to `origin/master` with a clean worktree.
- Added provider-neutral `IndustryTaxonomy`, immutable/evidence-versioned
  `IndustryTaxonomyRelease`, stable `IndustryNode`, release-specific
  `IndustryNodeVersion`, explicit `IndustryNodeLineage`, and the one-to-one
  `SecurityIndustryClassification` extension of `HistoricalUniverse`.
  Classification adds no second effective/knowledge-time columns.
- Added sealed `UniverseEvidenceDataset` / `UniverseEvidenceDatasetRow` corpus
  identity. PostgreSQL permits one serialized `BUILDING` → `COMPLETE|FAILED`
  transition, rejects row additions after sealing, enforces source/universe and
  retrieval-watermark identity, and requires classification extensions before a
  classification dataset can seal.
- Added immutable, fingerprinted `PeerPolicy` plus the exact four explicit v1
  classification policies. Canonical `classification_sub_industry_v1` freezes
  5 membership peers, 5 valid observations, 60% coverage, no fallback, permanent
  security-ID ordering, `peer_stats_v1`, the 135-day inclusive fundamental rule,
  exact-T estimate/market selection, and strict compatibility metadata.
- Extended the existing `HistoricalUniverseResolver` with optional sealed-dataset
  and source boundaries while retaining its single latest-known-version-before-
  effective-interval algorithm. Added a single-security classification evidence
  resolver and strict release-path reconstruction for later peer resolution.
- PostgreSQL guards enforce release correction sequencing/supersession, strict
  four-level parentage, same-release parent paths, sub-industry classifications,
  release/evidence time compatibility, immutable taxonomy/classification/policy
  rows, and sealed dataset membership. Stable node identity survives a release
  reparenting; codes/names remain release metadata.
- Migration `p26001` adds nine tables and required indexes/triggers. Persistent
  current/head is **p26001**; `alembic check` reports no upgrade operations.
  Disposable PostgreSQL passed fresh base → `p26001` → `fnd006` → `p26001`, exact
  9/0/9 Phase 2.6 table checks, metadata comparison, and the focused suite. The
  configured research database was upgraded only and never downgraded.
- Tests: **8 peer-foundation passed**; **138 focused foundation/frozen-runner
  regressions passed, 1 existing warning**; full regression **439 passed, 8
  existing warnings, zero failures**. Coverage includes hierarchy, stable/release
  identity, future-effective membership, correction knowledge boundaries, sealed
  build isolation, re-entry, taxonomy change, historical delisted membership,
  uniqueness, append-only behavior, policy idempotency, and schema clocks/indexes.
- No Phase 2.2, 2.4, or 2.5 formula/runner behavior changed. No peer group,
  statistic, feature output, industry aggregate, factor, signal, ranking,
  backtester, proprietary taxonomy data, or Phase 2.7 work was added.
- Production taxonomy/classification and survivorship-safe admission remain
  external provider/licensing gates. Current tests use fictional synthetic
  structure and cannot qualify live GICS history.
- Next: **2.6.2 — deterministic same-sub-industry peer resolver**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Add Phase 2.6 taxonomy and membership foundation$'`.

### Checkpoint 2.6.0 — Freeze spec

Checkpoint **2.6.0 — Freeze spec** is complete. The authoritative concise
implementation contract is
[phase-2.6-peer-industry-spec.md](phase-2.6-peer-industry-spec.md), derived without
changing the approved decisions in the full
[architecture review](phase-2.6-peer-industry-architecture-review.md).

- Starting checkpoint: `1a6de6374a0969922a42dd535eac9fa7cfa6529d`, equal to
  `origin/master`, with a clean worktree.
- Baseline reproduced: **431 passed, 8 existing warnings, zero failures**.
- Alembic current/head: **fnd006**.
- Frozen decisions include exact effective/knowledge-time ordering, sealed dataset
  pinning, same-sub-industry/no-fallback policy, 5/5/60% thresholds, 135-day
  fundamental freshness, exact-T estimate/market snapshots, Decimal median and
  `(L + 0.5E) / N` percentile, weakest-input quality, the exact ten feature IDs,
  and the exact eight industry metrics.
- No models, migrations, application code, formulas, feature registries, provider
  qualifications, or Phase 2.7 work changed in this checkpoint.
- Production GICS/classification and survivorship-safe use remain externally
  provider- and rights-gated. Synthetic fixtures will remain `DEVELOPMENT`.
- Next: **2.6.1 — taxonomy + membership models**.
- Commit locator:
  `git log -1 --format='%H' --grep='^Freeze Phase 2.6 peer and industry specification$'`.

## FOUNDATION + PHASE 2.5 FREEZE — APPROVED

The independent adversarial review of starting master
`fdd8b7a778547677aaa4e388480744639c91bc89` is complete. See the
[freeze record](PHASE_2_5_AND_FOUNDATION_FREEZE_REVIEW.md).

- **Foundation:** FROZEN for implemented PIT/data semantics.
- **Phase 2.5:** FROZEN for architecture, formulas, event semantics and persistence.
- **Phase 2.6:** implementation started at checkpoint 2.6.0 after this review;
  foundation and Phase 2.5 remain frozen.
- **External provider qualification:** independently tracked and still fail-closed.

The review reproduced and fixed four release-blocking defects: incomplete catalog
contract validation, an estimate-feature provider-admission bypass, deletable
historical FeatureValue rows, and three declared-but-unmigrated temporal indexes.
Estimate publication now requires an exact catalog/profile/version match, coverage
of the complete requested lookback, explicit synthetic opt-in, and persisted quality
admission. Unknown providers require a recorded development override and remain
`DEVELOPMENT`.

Current regression: **431 passed, 8 existing warnings, zero failures**. Alembic
current/head is **fnd006** and `alembic check` reports no pending operations.
Disposable verification passed fresh base→head, prior `fnd005`→head, the deeper
pre-estimate downgrade/re-upgrade, all estimate/foundation tests, and the isolated
database/raw backup/restore/tamper drill.

Remaining dependencies are external rather than silent implementation gaps: a
licensed/versioned market/action/delisted provider; credentialed live ALFRED
qualification; broad accession-level Company Facts reconciliation; a production
estimate feed; verified source rights; and a live encrypted off-host restore drill.
No live provider is survivorship- or production-qualified. Runtime gates enforce
that boundary and record development overrides at `DEVELOPMENT` quality.

| Risk | Status | Meaning |
| --- | --- | --- |
| Historical-universe PIT | RESOLVED | Effective and knowledge time are separate and immutable |
| Market-provider PIT | EXTERNALLY BLOCKED | Yahoo is development-only; historical claims fail closed |
| Macro vintages | PARTIAL / EXTERNALLY BLOCKED | Schema plus live adapter exist; ALFRED live qualification pending |
| Raw reproducibility | RESOLVED for new writes / IRRECOVERABLE LEGACY | Original/reconstructed/missing/mismatch gate; 34 missing test receipts retained |
| Feature-history immutability | RESOLVED | Exact reruns are idempotent; UPDATE and DELETE are rejected; changed inputs require a new build/version |
| Survivorship | EXTERNALLY BLOCKED | Identity mechanism and gate resolved; complete provider coverage absent |
| SEC PIT | PARTIAL / EXTERNALLY BLOCKED | Timing and version selection fixed; Company Facts aggregate remains gated |
| Local durability | RESOLVED | DB plus raw artifact and restore validation |
| Off-host durability | PARTIAL / EXTERNALLY BLOCKED | S3 encrypted/checksummed workflow tested; live account drill pending |
| Production estimates | EXTERNALLY BLOCKED | Runner-enforced catalog admission; synthetic-only qualification retained |

Decision: **PASS — FREEZE APPROVED**. Phase 2.6 is the recommended next phase.
Earlier entries below are historical checkpoint records and retain their original
commit-specific test counts and status language.

## Phase 2.6 architecture review — 2026-09-22

- Phase 2.6 architecture review performed.
- Phase 2.6 implementation not started.

The implementation specification is
[phase-2.6-peer-industry-architecture-review.md](phase-2.6-peer-industry-architecture-review.md).

### Final foundation qualification validation and publication

- Implementation/qualification checkpoint `798b8d3c5a7e04cf59252dccd640eef4b4a13056`
  was pushed to `origin/master` from a clean worktree.
- Final executable checkpoint `0e6977d338ee13240876e1f0766dffe4c752799a`
  adds atomic conditional backup writes and was pushed to `origin/master`.
  Its regression was **428 passed, 8 existing warnings, zero failures** in
  108.13s; focused foundation suite: **50 passed**.
- Clean-checkout disposable drill at that commit passed fresh base→`fnd005`,
  downgrade to `25e001`, re-upgrade to `fnd005`, foundation tests, PostgreSQL plus
  raw-byte backup, isolated restore, full table/query/hash checks, and corrupted-
  dump rejection. It created and dropped only random disposable databases.
- Synthetic drill report:
  `/tmp/irs-foundation-drill-w34u713u/restored_raw/restore_report.json`; backup
  manifest SHA-256
  `b1515ad86095a18b81c24620a2100b5d52d57d30ecacde8ba111465745d16141`.
  The deliberately damaged dump is not a recovery artifact.
- Persistent reconciliation remained exactly 34 missing test-only receipts after
  the full suite, matching `LEGACY_RAW_RECONCILIATION.json`; no row was fabricated
  or deleted. Local Markdown links, Python compilation, and `git diff --check`
  passed.

## Latest extension — qualitative research, 2026-09-21

Qualitative/alternative research architecture documented; implementation remains
unchanged. BusinessQualityResearch, CompanyStrategicEvents, evidence hierarchy,
professional research permissions, corroboration/disagreements and thesis-graph
outcomes are future designs. [Detailed extension](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md).
Strategic CONDITIONAL PASS and R1–R9 remediation gates remain open. Phase 2.5 is
NOT YET FROZEN; Phase 2.6 NOT STARTED.

## Latest status — strategic review, 2026-09-21

Strategic architecture/documentation review complete: **CONDITIONAL PASS**.
Phase 2.5 implementation remains complete / ready for independent review / **NOT
YET FROZEN**. Phase 2.6 **NOT STARTED**. New validation/learning architecture is
documented, not implemented. The [strategic review](STRATEGIC_ARCHITECTURE_REVIEW.md)
identifies source/universe/immutability gates before more feature expansion.
Earlier checkpoint entries below are historical records, not current resume commands.

## Project Identity
- **Repository**: `Chandresh007/investment-research-system`
- **Primary Branch**: `master`
- **Objective**: Build a deterministic, PIT-correct investment research engine that discovers fundamental and market inflections and ranks public companies for research.

## Engineering Principles (Target Architectural Invariants)

These are required guarantees, not a claim that every legacy source/storage path
already enforces them. Current exceptions and release gates are in the strategic review.

1. **No look-ahead bias**: Strict Point-in-Time (PIT) enforcement via `available_at`.
2. **Immutable Raw Data**: Source data is never modified to fit calculations.
3. **Authoritative Fiscal Identity**: Use accession/report context and explicit
   fiscal evidence. SEC Company Facts `fy`/`fp` are reporting-filing context and
   cannot by themselves make a comparative fact the current measured quarter.
4. **No Fuzzy Dates**: No calendar-date or fuzzy matching for primary fiscal identity resolution.
5. **Standalone $\neq$ YTD**: YTD facts cannot be substituted for standalone quarterly data.
6. **Explicit Missingness**: `MISSING` does not mean zero.
7. **Explicit Ambiguity**: `UNKNOWN` does not mean inferred.
8. **Full Provenance**: Every calculated feature must retain a direct lineage to its source `FinancialFact` IDs.
9. **Versioned Logic**: Every calculation must be tagged with a `calculation_version` (e.g., `fundamental_v1`).
10. **Deterministic Source of Truth**: Metrics and rankings are produced by code, not AI.
11. **AI for Synthesis Only**: AI interprets evidence; it does not create or repair it.
12. **PIT-Correct Backtests**: Future information cannot influence historical features or rankings; future outcomes intentionally define separately versioned labels.
13. **Explainable Ranking**: All rankings must be based on inspectable evidence.
14. **Reproducible Results**: Any research snapshot must be reproducible from stored evidence.

## Accepted Milestones

### Phase 1: Data Foundation
**Status**: ACCEPTED
- Established basic company, security, and raw data ingestion pipelines.

### Phase 2.1: Temporal / PIT Foundation
**Status**: ACCEPTED
- **Checkpoint**: `14720d4`
- Implemented `historical_universe`, `corporate_actions`, and PIT-aware `feature_values` schema.
- Established the "latest-known" rule for research timestamps.

### Phase 2.2 Step 1: Fiscal Identity Foundation
**Status**: ACCEPTED
- Established the Phase 2.2 `fy/fp` resolver contract. Final qualification later
  identified Company Facts comparative-report context as a provider caveat and
  preserved it separately without changing frozen formulas.
- Implemented Q1 ambiguity handling (`UNKNOWN` status for ambiguous durations).
- Prevented calendar-year inference for fiscal identity.

### Phase 2.2 Step 2: Fundamental Feature Calculations
**Status**: PASS / COMPLETE
- **Checkpoint**: `b9bc4a4`
- **Implementation**: 15 deterministic features (Revenue, Margins, EPS, OCF, FCF) in `FundamentalFeatureRunner`.
- **Provenance**: Every `FeatureValue` stores a JSON list of source `FinancialFact` IDs.
- **Calculation Version**: `fundamental_v1`.
- **PIT Correctness**: Verified that features are `MISSING` before filing and `VALID` after filing.
- **Final Fixes**:
    - Resolved Q1 ambiguity: `UNKNOWN` types are comparable if `fiscal_period` is present.
    - Fixed Security $\rightarrow$ Company resolution in the runner.
    - Optimized PIT candidate selection: Order by `period_end DESC` then `filed_date DESC`.
    - Fixed numeric truthiness: Exactly `0.0` is now correctly returned as a valid value.

### Phase 2.4: Market Features
**Status**: PASS / FROZEN
- **Frozen Checkpoint**: `c9c365cb99da9b21966b06400bb449ae1833713d`
- **Implementation**: Deterministic market features (Returns, Momentum, Risk, Volume) in `MarketFeatureRunner`.
- **Price Pipeline**: Raw $\rightarrow$ PIT-safe Split-Adjusted $\rightarrow$ TRI.
- **Benchmarks**: SPY, GICS Sector ETFs, and PIT-correct equally weighted industry baskets.
- **PIT Correctness**: Verified that corporate actions and prices are only used if `available_at <= research_timestamp`.
- **Observation Windows**: Strict $W+1$ for returns and $W$ for SMA/Drawdown.
- **Provenance**: Full lineage for every `FeatureValue` including benchmark and adjustment info.
- **Calculation Version**: `market_v1`.
- **Canonical Feature IDs**: `return_1d`, `return_5d`, `return_21d`, `return_63d`, `return_126d`, `return_252d`, `rel_ret_63_mkt`, `rel_ret_63_sec`, `rel_ret_63_ind`, `dist_52w_high`, `price_vs_sma200`, `sma_slope_50`, `realized_vol_63`, `max_dd_252`, `vol_regime`, `vol_trend_21`, `abnormal_vol_1`, `dollar_vol_21`, `mom_accel_63`, `ret_abs_1d`, `ret_abs_21d`, `ret_abs_63d`.
- **Frozen Semantics**:
    - PIT split knowledge: corporate actions are usable only when known at or before the research timestamp.
    - Split-continuous price $P'$: PIT-known splits normalize observations where `t < effective_date`; observations where `t >= effective_date` remain on the already post-split raw basis.
    - Reciprocal volume adjustment: adjusted volume equals raw volume multiplied by the reciprocal split factor, preserving dollar turnover.
    - TRI: `TRI_t = TRI_(t-1) * (P'_t + D_t) / P'_(t-1)` with PIT-known cash distributions only.
    - Exact windows: W-day price windows require exactly W valid trading observations; W-interval returns require W+1 valid prices.
    - Relative strength: broad market uses SPY, sector uses deterministic GICS sector ETF mapping, and industry uses daily-rebalanced equal-weight PIT baskets with minimum 3 valid constituents per interval.
    - Stale policy: latest observation is stale when more than 5 deterministic US equity trading sessions old.
    - Delisted securities: historical observations remain usable for historical snapshots until they become naturally stale.
    - Provenance: persisted `market_v1` values include reproducible source, input-window, benchmark, corporate-action, and methodology context.
    - Idempotency: repeated deterministic `market_v1` runner execution updates/preserves the same unique feature rows without duplicates.
- **Verification**:
    - Phase 2.4 tests: 15 passed.
    - Full suite: 72 passed.
    - PostgreSQL migration head includes corporate-action enum support for `STOCK_SPLIT`, `REVERSE_SPLIT`, and `CASH_DIVIDEND`.
    - Implementation was validated primarily with deterministic synthetic data because live market-history coverage remains insufficient for broad 63/252-session real-data validation.

## Known Data Limitations
- **Coverage**: Some companies have incomplete `FinancialFact` coverage in the SEC Company Facts source.
- **CapEx**: SEC Company Facts may lack reliable CapEx for certain periods, resulting in `MISSING` FCF.
- **Q1 Ambiguity**: Standalone vs YTD Q1 data remains `UNKNOWN` when SEC metadata is ambiguous.
- **Universe**: The current validation universe is a subset of the public company universe.
- **Market History**: Live market-history coverage is still insufficient for broad 63/252-session real-data validation; Phase 2.4 was frozen based on deterministic synthetic validation.

## Technical Rules
- **Fiscal Identity**: require accession/report evidence; preserve SEC `fy`/`fp`
  as reporting context and do not infer identity from calendar dates.
- **PIT Rule**: resolved `FinancialFact.available_at <= research_timestamp`; exact
  acceptance when evidenced, otherwise conservative next-day for date-only SEC data.
- **Period Shapes**: Strict separation of `QUARTERLY` vs `YTD`.
- **Provenance**: Required for all calculated features.
- **Idempotency**: Guaranteed via `UniqueConstraint` on `(security_id, feature_id, period_end, available_at, calculation_version)`.
- **No Fabricated Data**: Do not substitute YTD for quarterly or invent numbers to fill gaps.

## Roadmap
1. **2.1 Temporal/PIT Foundation** (Done)
2. **2.2 Fundamental Feature Store** (Done)
3. **2.4 Market Features** (Done / Frozen)
4. **2.5 Estimates** (Implementation complete / ready for independent review / not yet frozen)
5. **2.6 Peer/Industry Context** (Planned)
6. **2.7 Factors** (Planned)
7. **2.8 Signals** (Planned)
8. **2.9 Backtester** (Planned)
9. **2.10 Calibration** (Planned)
10. **2.11 AI Analyst** (Planned; valuation/deeper snapshot and basic event-evidence prerequisites)
11. **2.12 News/Event Intelligence** (Proposed production expansion)
12. **2.13 Business Quality / Source Expansion** (Proposed extension)

## New Session Startup Procedure
For any agent entering this project in a new session:

1. **Read the Foundations**: Read `README.md` and `docs/PROJECT_STATE.md` first.
2. **Audit the State**: Run `git status --short` and `git log --oneline -10`.
3. **Inspect Implementation**: Read the code before making assumptions about behavior.
4. **Respect the Freeze**: Accepted milestones are frozen foundations. Do not reopen accepted work without evidence of a concrete defect.
5. **Maintain Invariants**: Preserve PIT, provenance, and fiscal identity rules.
6. **Gatekeeping**: Do not advance to a new phase without passing all verification gates.
7. **No Convenience Changes**: Do not modify the architecture for convenience; prioritize correctness and auditability.

## Phase 2.5 implementation handoff — 2026-09-21

### Checkpoint 2.5.0 — Freeze spec
- Status: architecture frozen for implementation; no estimate code implemented yet.
- Starting HEAD/origin: `cad71f308fd3403ddc85d9d01dfd21d27277229d`.
- Reconstructed master with fetch/checkout/ff-only pull; Python 3.12.14 and local
  editable import verified. Found an existing untracked final architecture review;
  preserved it in Git as historical review material.
- Created `docs/phase-2.5-estimates-spec.md`, the authoritative implementation
  contract; preserves all approved semantics, acceptance matrix and worked example.
- Models/migrations: none in this documentation checkpoint.
- Baseline: `python -m pytest -q`: **83 passed, 8 warnings**, 14.32s;
  zero failures. Warnings are existing datetime UTC and SQLAlchemy deprecations.
- `git diff --check`: passed.
- Decisions: floor-free scaled EPS, directional breadth denominator, absolute
  fiscal identity, PIT evidence throughout, pinned datasets, immutable corrections,
  provider-scoped consensus, exact 15 inactive legacy-registry entries.
- Limitations: implementation not started; real provider qualification deferred.
- Next: **2.5.1 canonical models and migration**, then checkpoints 2.5.2–2.5.8.
- Commit: resolve `git log --format='%H' --grep='^Freeze Phase 2.5 estimates architecture$' -1`;
  the exact resulting hash will be recorded by the next handoff update.
- Phase 2.6 is NOT STARTED. Phase 2.5 is NOT FROZEN as an implementation.

### Checkpoint 2.5.1 — Canonical models and migration
- Status: implementation checkpoint complete; services/replay remain pending.
- Previous checkpoint 2.5.0 commit: `00fa6b0e5ef5b34b95077403eab3717cd69cc792` (pushed).
- Created `src/investment_research/models/estimates.py` and registered mappings
  in `models/__init__.py`. Fifteen tables: provider profiles, datasets, absolute
  fiscal periods, fiscal evidence, contributors, contributor aliases, coverage
  assertions, observations, **dataset observation membership**, consensus series,
  states, members, exclusions, actual reporting events, actual reporting facts.
- Migration: `25e001_estimate_foundation.py`, parent `1f4b7c9d2a31`.
- Decisions: BIGINT identities; exact unconstrained NUMERIC; TIMESTAMPTZ with
  microsecond precision; explicit annual FY; nonnullable series uniqueness;
  indexed temporal lookups; relational selected observation/alias/fiscal evidence.
- Pinning: observations are immutable globally and explicitly included in a build;
  temporal evidence belongs to a dataset. PostgreSQL triggers reject mutation of
  history and insertion of inputs after completion. Dataset-row locks serialize
  input additions with sealing. Observation membership enforces the profile,
  normalization version and retrieval watermark. A new backfill needs a new build.
- Database checks reject malformed fiscal identities, event/basis/timing labels,
  invalid status/value combinations, nonfinite observations/multipliers, invalid
  withdrawal payloads and self-supersession. Database triggers enforce matching
  security/target companies. Cross-event cycle/branch validation is next checkpoint.
- **Minimal legacy integration repair:** existing Python FeatureStatus.INVALID was
  missing from the historical PostgreSQL featurestatus enum. This migration adds
  the label and tests legacy FeatureValue persistence. No frozen calculation or
  old migration was changed. The additive label intentionally survives downgrade.
- Tests: `python -m pytest -q tests/estimates/test_models.py`: **38 passed**, no warnings.
- Regression: `python -m pytest -q`: **121 passed, 8 pre-existing warnings**, 14.86s.
- Fresh PostgreSQL: `python scripts/verify_estimate_migrations.py` passed base →
  head → `1f4b7c9d2a31` → head, verified 15/0/15 estimate tables, shared enum,
  and **38 model tests** on the disposable database. Only that database is dropped.
- Existing database upgrade passed; current/head both `25e001`. Model/migration
  Alembic metadata comparison found no estimate differences. `git diff --check` passed.
- Intermediate failures fixed: omitted shared INVALID enum; legacy tests' explicit
  positive raw IDs collided with the sequence. New rollback-only fixtures use a
  negative raw-ID namespace, leaving legacy tests/data unchanged.
- New files: model module, migration, `tests/estimates/conftest.py`,
  `tests/estimates/test_models.py`, `scripts/verify_estimate_migrations.py`.
- Limitations: schema-level foundation only. Multirow semantic validation,
  normalization, fiscal/reporting/contributor resolution, event replay, atomic
  state publication and feature calculations are not implemented yet. Real
  provider qualification remains deferred. PostgreSQL triggers require Alembic,
  not Base.metadata.create_all(), for authoritative persistence guarantees.
- Next: **2.5.2 normalization and event identity**.
- Commit locator: `git log --format='%H' --grep='^Add canonical estimate models and PostgreSQL foundation$' -1`.

### Checkpoint 2.5.2 — Normalization and event identity
STATUS: COMPLETE

- Recovered from pushed `a3d59206d65c2a723e88e8ad23f18a5a38f247c3` with
  uncommitted normalization, connector, runner, tests and migration-verifier work.
  Preserved and reviewed every recovered file; initial verification reproduced
  **71 estimate tests / 154 full-suite tests**, with 3 / 11 warnings.
- Created `connectors/estimates/base.py` (original-byte response and provider
  interface), `normalization/estimates.py` (provider-neutral abstract normalizer,
  synthetic JSON adapter and raw archive), `ingestion/estimate_runner.py`, and
  `tests/estimates/test_estimate_normalization.py`. Paths are under
  `src/investment_research/` unless prefixed by `tests/`.
- Provider semantics live on immutable profiles; ingestion pins the dataset and
  normalization version. Explicit event types must be supported by the profile.
  Historical coverage is separate evidence, never inferred from capability tier.
- Exact Decimal scaling and canonical decimal hashes preserve zero, reject binary
  floats/nonfinite values and deduplicate equivalent representations. Unknown
  targets and negative revenue retain explicit missing/invalid reasons.
- Event/profile/version identity and semantic payload hashes distinguish repeated
  transmission from EVENT_ID_CONFLICT. No existing observations are overwritten.
- Corrections reference a prior version of the same logical event/stream/security;
  reject cycles, branches, absent parents, backwards visibility and incompatible
  replacement values. All predecessor versions must belong to the pinned build,
  including when reusing already-normalized corrections. Chains retain originals
  and newer economic revisions. Economic replay/selection is checkpoint 2.5.4.
- PIT timestamps use certified publication evidence; date-only availability is
  next local midnight converted to UTC (DST tested). Missing/unknown timezone or
  uncertified dissemination uses first retrieval. provider_as_of cannot backdate
  availability; uncertified precise published_at remains stored separately.
- Archive paths incorporate hashed request identity plus content hash; distinct
  responses in one second do not overwrite. Replay verifies stored bytes against
  RawDataObject.content_hash. Added optional aware `retrieved_at` to existing
  RawStorage, preserving its legacy UTC column and default for other callers;
  estimates now retain actual response receipt time without test-time DB repair.
- Existing IngestionRun records success/failure and raw-object reference. Batch
  observations and memberships commit atomically; repeated ingestion saves zero
  duplicates. A multi-record conflict test proves rollback after an earlier row
  in the batch was inserted, while failed-run/raw evidence remains auditable.
- Final focused suite: **81 passed, 4 warnings** (43 normalization + 38 model tests).
  Full regression: **164 passed, 12 warnings**, zero failures. Warnings are legacy
  datetime.utcnow / SQLAlchemy Query.get deprecations, including additional raw
  storage calls exercised by the new audit test. `git diff --check`: passed.
- Migration unchanged: persistent current/head **25e001**. Updated
  `scripts/verify_estimate_migrations.py` to run all estimate tests: disposable
  PostgreSQL fresh base -> head -> prior head -> head passed, 15 estimate tables,
  **81 passed, 4 warnings**. Persistent research DB was not downgraded.
- Limitations: synthetic provider only; no licensed/live provider, no inferred
  event coverage, and no consensus, activity or feature runner yet. Target,
  contributor and reporting eligibility resolution follows in checkpoint 2.5.3.
- Next: **2.5.3 — fiscal target, contributor, and reporting resolvers**.
  Phase 2.5 implementation remains NOT FROZEN. Phase 2.6 is NOT STARTED.
- Commit: resolve `git log --format='%H' --grep='^Implement Phase 2.5 estimate normalization and event identity$' -1`.

### Checkpoint 2.5.3 — Fiscal target, contributor, and reporting resolvers
STATUS: COMPLETE

- Previous checkpoint 2.5.2: `4f93fb671c039b7fa162b2bd0db4b73e689b0b8c`
  (pushed; clean tree verified before advancing).
- Created `src/investment_research/research/estimates/resolvers.py` and
  `tests/estimates/test_estimate_resolvers.py`; no model/migration changes.
- `EstimateTargetResolver`: resolves explicit absolute identity or evidenced
  provider-period crosswalk, filtered by dataset/profile availability and PIT
  supersession. Missing mappings return TARGET_PERIOD_UNKNOWN; conflicting
  crosswalk/explicit labels or period metadata return TARGET_PERIOD_AMBIGUOUS.
  Calendar month, duration and period-end dates never establish fiscal identity.
- `ContributorResolver`: provider/profile-scoped stable-ID and explicit alias
  evidence; approved canonical-broker/full-name fallback requires a
  BROKER_NAME_UNIQUE mapping. Unicode/whitespace/case normalization only. Alias
  knowledge and effective intervals are both enforced, including future-effective
  changes known earlier. Collisions/unresolved identities are excluded explicitly.
- `ActualReportingResolver`: reads the append-only reporting projection and
  returns REPORTED / NOT_REPORTED_AS_OF_T / UNKNOWN with evidence IDs. Absence
  alone is insufficient; complete actual-reporting coverage must include T and
  extend back to the evidenced period start. Incomplete/conflicting coverage or
  unknown reporting evidence produces UNKNOWN. Future evidence is excluded.
- Added transactional, idempotent `project_filing` linking existing Filing and
  FinancialFact rows through a qualified archived `ReportingContext`. Requires
  explicit primary-report fiscal identity, matching company/accession/fy/fp and
  known supporting timestamps. Comparative facts alone and amended filings cannot
  establish first reporting. Conflicting repeated projections reject explicitly.
  Additional reporting confirmations preserve the earliest known boundary;
  explicit identity supersession takes effect only when known.
- `ForwardHorizonResolver`: verifies a reported anchor and walks explicit,
  reciprocal successor/predecessor evidence independently for quarter/FY chains.
  Missing/ambiguous sequence or reporting evidence returns
  FORWARD_HORIZON_UNRESOLVED. No estimate-presence query is involved, so missing
  estimates cannot compress horizons. Absolute IDs remain unchanged at rollover.
- Tests: **23 new PostgreSQL resolver tests**; focused estimates **104 passed,
  4 warnings**; full suite **187 passed, 12 warnings**, zero failures. Covers
  January year-end/non-calendar fiscal labels, unknown/ambiguous/date-only mapping,
  stable IDs/renumbering/fallback names, PIT/effective aliases, reporting gaps,
  after-hours and exact boundary rollover, annual independence, amendments,
  projection provenance/idempotency, fiscal conflicts and incomplete coverage.
- Verification: `git diff --check` passed; current/head remain **25e001**.
  Disposable PostgreSQL base -> head -> prior head -> head passed with 15 estimate
  tables and **104 passed, 4 warnings**. Persistent DB was not downgraded.
- Limitations: evidence adapters must supply qualified raw reporting context and
  canonical broker identities; no live SEC/release adapter or automatic broker
  matching is introduced. No fiscal or reporting history is inferred. Conservative
  conflicts/missing evidence can leave a horizon unresolved. Consensus/replay and
  activity/feature integration are still pending.
- Next: **2.5.4 — event replay and consensus**. Checkpoints 2.5.4–2.5.8 remain;
  Phase 2.5 is NOT FROZEN and NOT ready for independent final audit. Phase 2.6 is
  NOT STARTED. Stop this session at the durable checkpoint boundary so the next
  event-replay checkpoint can be implemented and pushed with sufficient context.
- Commit: resolve `git log --format='%H' --grep='^Implement Phase 2.5 fiscal contributor and reporting resolvers$' -1`.

### Checkpoint 2.5.4 — Event Replay + Consensus
STATUS: COMPLETE

- Starting HEAD/origin: `d783226243eab23acb8257dec5a3f4aa25313764`, master,
  clean after fetch/ff-only pull. Python 3.12.14 and local editable import verified.
  Baseline: **104 estimate tests / 187 full-suite tests**, 4 / 12 existing warnings,
  zero failures.
- Added `src/investment_research/research/estimates/consensus.py`:
  `EstimateEventResolver`, `ConsensusCalculator`, `ConsensusStateWriter`,
  `EstimateConsensusRepository`, and typed economic-event/contributor/result
  records. Reuses the existing immutable models, dataset observation membership,
  target/contributor/reporting resolvers and consensus tables; no migration or
  changes to frozen Phase 2.1, 2.2 or 2.4 behavior.
- Replay filters observations and supporting evidence at `available_at <= T`.
  NEW starts an episode, REVISION replaces its active head, exact-scope WITHDRAWAL
  terminates it, and NEW after withdrawal starts a different episode. An orphan
  revision is unresolved rather than an invented initiation. Unusable current
  heads (including semantic, numeric, identity and target failures) cannot
  resurrect older valid estimates. Ambiguous attachment to parallel semantic
  streams excludes the affected contributor instead of choosing a convenient head.
- Corrections resolve visible version chains before economic replay. The root
  retains event kind, economic order and episode identity. Older-event corrections
  cannot displace later revisions; corrected withdrawals remain withdrawals for
  their corrected exact scope. Explicit RETRACT removes the referenced economic
  representation. Target-moving value corrections change membership at correction
  availability without creating initiations. Cycles, absent/incompatible parents,
  conflicting correction branches and duplicate logical roots fail explicitly.
- Ordering: causal ancestry first, documented provider sequence second, documented
  publication ordering third (`ordering_policy.sequence` / `.published_at`).
  Weaker ordering cannot rank a successor below its causal ancestors. Conflicting
  equal precedence is EVENT_ORDER_AMBIGUOUS, never a database-ID tiebreak. Delayed
  non-head events preserve the selected head and do not manufacture state changes.
- Exact provider/security/absolute-target/metric/basis/methodology/currency/unit/
  share-type/share-basis/scope isolation. Known alternative EPS bases and basic
  EPS remain separately queryable; no cross-vendor pooling or implicit FX/split
  conversion. Revenue consensus requires REPORTED basis. Selected estimates must
  match PIT-evidenced reporting currency and fiscal mapping.
- Consensus uses equally weighted Decimal arithmetic with versioned 38-digit
  ROUND_HALF_EVEN context, mean/median/high/low, sample stddev (`ddof=1`), and count.
  Fewer than three active resolved contributors gives
  MISSING / INSUFFICIENT_CONTRIBUTORS with null aggregates and preserved members
  and exclusions. `[1,2,3]` produces mean/median 2, low 1, high 3, stddev 1.
- Writer API: `ConsensusStateWriter(session, dataset_id).replay(series_id, through)`.
  Reconstructs complete visibility boundaries, including alias/fiscal evidence
  changes; writes only changed fingerprints, with no daily expansion. Fingerprints
  include pinned profile/build/manifest/calculation policy, selected observation
  versions, identity/fiscal evidence, membership, exclusions, status and values.
  Same-value reaffirmations and changed evidence can create states even when
  aggregates match; identical retransmissions/replays do not duplicate states.
- Publication uses a savepoint inside the caller's transaction; the caller commits.
  State, all relational members and all exclusions roll back together on failure.
  The existing PostgreSQL dataset row lock serializes writers with ingestion and
  sealing, and the unique dataset/series/version/boundary constraint remains the
  final guard. Two real concurrent transactions on disposable PostgreSQL return
  the same state; a third reader cannot see uncommitted state or partial members.
- Dataset/build pinning is enforced on replay and lookup. Existing state conflicts
  raise PUBLISHED_STATE_CONFLICT_USE_NEW_DATASET; historical backfills must use
  a new build. Published states are never rewritten. Assemble all same-boundary
  inputs before publication; sealing the input corpus first is recommended.
- Repository API: `EstimateConsensusRepository(session, dataset_id).get_as_of(
  series_id=..., t=..., calculation_version='consensus_v1')`. Also accepts
  `security_id`, `fiscal_period_id` and all nine exact semantic dimensions instead
  of `series_id`; incomplete dimension requests are rejected. Returns a snapshot
  with `.state` (ID, status/reason, statistics, count, fingerprint and provenance),
  `.members`, `.exclusions`, and PIT `.reporting` context, or None before any state.
  Latest state at/before T is selected even if missing/invalid. Reporting changes
  do not duplicate absolute consensus. Call ForwardHorizonResolver separately
  when starting from a relative horizon.
- Added **50 adversarial tests** in `tests/estimates/test_consensus.py`: episodes,
  revisions/reaffirmations, exact withdrawals, corrections/retractions/target moves,
  invalid correction graphs, delayed/causal/sequence/publication ordering, ordering
  conflicts, three-contributor threshold and exact statistics, semantic isolation,
  no-fallback failures, future/exact-boundary visibility, aliases/fiscal evidence,
  relational provenance, equal-mean state changes, retransmission idempotency,
  rollback, concurrent publication, pinned builds, historical as-of lookup, and
  reporting-context PIT behavior. Existing assertions were not weakened.
- Final focused suite: **154 passed, 4 warnings**. Full suite: **237 passed,
  12 warnings**, zero failures. Warnings remain datetime.utcnow / Query.get
  deprecations. `git diff --check` passed. Alembic current/head remain **25e001**.
  Disposable PostgreSQL base -> head -> `1f4b7c9d2a31` -> head passed with all
  **154 estimate tests**, 4 warnings. Persistent DB was not downgraded.
- Architecture deviations: none. Operational limitations: synthetic provider only;
  whole-series reconstruction favors correctness over incremental performance;
  the conservative dataset lock serializes different series within one build.
  Provider sequence/publication guarantees and stream IDs must be documented by
  a qualified adapter. No activity windows, revision features, estimate feature
  runner, live provider qualification or Phase 2.6 work is included.
- Next: **2.5.5 — Revision Mathematics + Activity**. Stop at this durable boundary;
  2.5.5 has not started. Phase 2.5 implementation is NOT FROZEN and checkpoints
  2.5.5–2.5.8 remain. **RESUME FROM PHASE 2.5 CHECKPOINT 2.5.5**.
- Commit locator: `git log --format='%H' --grep='^Implement Phase 2.5 event replay and consensus$' -1`.

### Checkpoint 2.5.5 — Revision Mathematics + Activity
STATUS: COMPLETE

- Starting HEAD/origin: `338a63a046ac979fc88affc11352152b15973340`, master,
  clean after fetch/checkout/ff-only pull. Python 3.12.14 and local package import
  verified. Baseline reproduced: **154 focused / 237 full**, 4 / 12 existing
  warnings, zero failures.
- Added `src/investment_research/research/estimates/revisions.py` with
  `EstimateRevisionCalculator`, `RevisionResult`, and `RevisionActivity`.
  Reuses dataset/profile pinning, contributor and horizon resolvers, the consensus
  repository and the existing event replay engine. Added an optional
  `before_event_key` economic-prefix query to `EstimateEventResolver.resolve`;
  its default consensus behavior is unchanged. No schema/migration changes.
- API: `EstimateRevisionCalculator(session, dataset_id).consensus_change(t=...,
  lookback_days=7|30|90, formula='EPS_ABSOLUTE'|'EPS_SCALED'|'REVENUE_PCT',
  series_id=...)`. Alternatively supply `security_id`, `horizon`, and every exact
  semantic dimension from `consensus.DIMENSIONS`. Horizon resolution occurs once
  at T; current and prior lookups use the same absolute series/target. UTC calendar
  subtraction and `state_available_at <= cutoff` apply without a market calendar.
  Selecting the same valid state gives zero. No prior row yields
  NO_PRIOR_CONSENSUS; a latest missing/invalid prior yields
  NO_VALID_PRIOR_CONSENSUS with that status and its underlying reason in evidence,
  never an older valid fallback. Current missing/invalid states propagate.
- EPS: absolute `new-old`; scaled `(new-old)/max(abs(old),abs(new))`, with both
  zero yielding zero. Scaled evidence always includes absolute change. Revenue:
  `(new-old)/old` as a ratio, requiring old > 0 and new >= 0; zero denominator is
  INVALID / DENOMINATOR_UNSUPPORTED and negative values are
  INVALID / INVALID_NUMERIC_VALUE. Reuses 38-digit ROUND_HALF_EVEN Decimal policy;
  no floats, arbitrary floors, percentage interpretation of EPS, or clipping.
- API: `.activity(t=..., series_id=...)` (same horizon/exact-dimension alternative)
  returns separate `.revision_count` and `.breadth` status/value/evidence records.
  Window is strictly **(T-30 calendar days, T]**. Each documented transition is
  classified at its original availability against the immediate comparable
  economic predecessor in the same active episode. Exact Decimal comparisons
  classify UP/DOWN/UNCHANGED; NEW, WITHDRAWAL and CORRECTION never count or vote.
  Multiple same-boundary events use documented economic order, not storage IDs.
- Revision count counts distinct UP/DOWN events, not contributors. Breadth uses
  one latest comparable economic update per contributor: `(U-D)/(U+D)`; latest
  UNCHANGED removes that contributor's directional vote. No directional votes
  yields MISSING / NO_DIRECTIONAL_REVISIONS. Withdrawals do not erase earlier
  revisions. Re-entry starts a new episode. Initiations and withdrawals are
  separately counted in evidence. Incompatible series events are excluded with
  existing semantic reasons. Delayed events cannot replace a newer breadth vote.
- Corrections can move consensus without economic activity. Transition evidence
  retains prior/new observation versions known at the original boundary; later
  corrections do not reclassify historical directions. A corrected active head
  supplies the baseline for subsequent revisions. Explicit economic retractions
  and unresolved predecessors/order/identity conservatively make activity missing.
  PIT alias mergers consolidate breadth votes at T while preserving the original
  contributor/alias evidence used to classify each transition.
- Completeness requires a qualified tier-1 event profile plus PIT-visible
  ECONOMIC_EVENTS assertions covering the full window for the pinned dataset,
  security, target, metric and scope. Contiguous COMPLETE segments can establish
  coverage; overlapping incomplete/snapshot-only assertions or gaps cannot.
  Supersession applies only when known. Incomplete coverage returns
  MISSING / INCOMPLETE_EVENT_HISTORY even with observed revisions; absence of
  events alone never proves zero. Complete empty windows return count zero.
- Evidence includes T, cutoff/window, requested horizon and resolver evidence,
  absolute target/series, profile/build/manifest/normalization/calculation policy,
  current/prior state IDs, timestamps, statuses, reasons, fingerprints and means,
  formulas, transition versions/values/episodes/aliases/fiscal evidence, selected
  breadth contributors, U/D/Z, observed revision/initiation/withdrawal counts,
  and coverage assertion IDs. Outputs are deterministic research primitives;
  FeatureValue persistence remains checkpoint 2.5.6.
- Added **66 deterministic tests** in `tests/estimates/test_revisions.py`: all
  specified EPS/revenue numbers, nonfinite/float rejection, 7/30/90 calendar
  windows, weekend/same-state zero, real Q2 rollover from FQ2 to FQ1,
  absent/missing/invalid prior states, current missing propagation, all breadth
  examples, repeated events, initiation/withdrawal/re-entry/reaffirmation,
  correction-only consensus movement, corrected baselines, economic ordering,
  exact microsecond boundaries, future events/coverage/aliases, alias merger,
  coverage gaps/supersession/scope, dataset isolation and incompatible dimensions.
- Final focused suite: **220 passed, 4 warnings**. Full suite: **303 passed,
  12 warnings**, zero failures. Warnings remain existing datetime.utcnow and
  SQLAlchemy Query.get deprecations. `git diff --check` passed. Alembic
  current/head remain **25e001**. Disposable PostgreSQL base -> head ->
  `1f4b7c9d2a31` -> head passed with **220 focused tests**, 4 warnings, 15 estimate
  tables. The persistent database was not downgraded.
- Limitations: synthetic provider only; transition replay favors reproducibility
  over incremental performance and reconstructs prefixes per observed event.
  Consensus states must be published via the existing writer before magnitude
  lookup. Unresolved activity is conservative, with observed evidence retained.
  No registry/runner, ranking, factors, signals or Phase 2.6 work was introduced;
  frozen Phase 2.1, 2.2 and 2.4 code is unchanged.
- Next: **2.5.6 — 15-feature registry + EstimateFeatureRunner**. Stop at this
  independently committed/pushed checkpoint; 2.5.6 has not started. Phase 2.5
  implementation remains NOT FROZEN; checkpoints 2.5.6–2.5.8 remain.
  **RESUME FROM PHASE 2.5 CHECKPOINT 2.5.6**.
- Commit locator: `git log --format='%H' --grep='^Implement Phase 2.5 revision mathematics and activity$' -1`.

### Checkpoint 2.5.6 — Estimate Feature Registry + Runner
STATUS: COMPLETE

- Starting HEAD/origin: `67bdf69ee539e6efc978754b8366aa7b8cb58704`, master,
  clean after fetch/checkout/ff-only pull. Python 3.12.14 and local package import
  verified. Baseline: **220 focused / 303 full**, 4 / 12 existing warnings,
  zero failures. Read README, project state and frozen estimate specification.
- Added `src/investment_research/research/estimates/registry.py`,
  `src/investment_research/research/estimates/runner.py`, and
  `tests/estimates/test_feature_runner.py`. No model/migration changes or changes
  to frozen Phase 2.1/2.2/2.4 implementations, estimate formulas or resolvers.
- `registry.FEATURES` is an immutable explicit allowlist of 15 frozen definitions.
  Each definition specifies metric, basis, share type, horizon, lookback, formula,
  unit, family and version. ResearchFeature stores this metadata as deterministic
  JSON in the existing calculation_logic text field, plus name, description,
  category=Estimates, frequency=snapshot and NUMERIC data type. Registry units
  are currency/share for EPS absolute changes, ratio for scaled/percentage/breadth
  changes, and count for event counts. Absolute FeatureValue units use the actual
  series unit, e.g. USD/share. Canonical IDs, exactly:

```text
eps_adjusted_diluted_consensus_change_abs_7d_fq1
eps_adjusted_diluted_consensus_change_abs_30d_fq1
eps_adjusted_diluted_consensus_change_abs_90d_fq1
eps_adjusted_diluted_consensus_change_scaled_7d_fq1
eps_adjusted_diluted_consensus_change_scaled_30d_fq1
eps_adjusted_diluted_consensus_change_scaled_90d_fq1
eps_adjusted_diluted_consensus_change_scaled_30d_fy1
eps_adjusted_diluted_revision_breadth_30d_fq1
eps_adjusted_diluted_revision_count_30d_fq1
revenue_reported_consensus_change_pct_7d_fq1
revenue_reported_consensus_change_pct_30d_fq1
revenue_reported_consensus_change_pct_90d_fq1
revenue_reported_consensus_change_pct_30d_fy1
revenue_reported_revision_breadth_30d_fq1
revenue_reported_revision_count_30d_fq1
```

- Registration: `initialize_feature_registry(session)` or the runner method is
  deterministic/idempotent and uses PostgreSQL ON CONFLICT DO NOTHING followed
  by exact metadata validation. Existing unrelated metadata is never overwritten.
  Conflicting entries or extra estimates_v1 IDs fail atomically. All estimate
  entries remain **active=False**, version=estimates_v1; neither registration nor
  execution temporarily activates them. The estimate runner uses only its explicit
  allowlist and validates the version/family metadata. Fundamental/market runners
  are unchanged and their actual calculation loops exclude estimate entries.
- Runner API: `EstimateFeatureRunner(session, provider_profile_id=..., dataset_id=...)
  .calculate_for_security(security_id, t, feature_ids=None)` returns an ordered
  dictionary of canonical feature IDs to persisted FeatureValue objects. Omitted
  feature_ids means all 15; unsupported IDs reject the whole request before any
  snapshot writes. Profile/dataset mismatches, unknown security and failed builds
  fail explicitly. Caller owns commit; registration and feature batch use savepoints.
- Each requested FQ1/FY1 horizon is resolved once at T and reused across that batch.
  Selection requires the canonical basis/share type/consolidated scope, pinned
  profile methodology, PIT-evidenced reporting currency, and canonical currency
  units. A candidate series must have represented contributors in a PIT-published
  state or matching PIT-visible observations in this build. Empty historical
  states created before a future series' first observation cannot cause historical
  ambiguity. Other datasets cannot supply candidate evidence. Multiple evidenced
  share bases return MISSING / SHARE_BASIS_MISMATCH instead of arbitrary selection.
- Magnitude and activity call the existing EstimateRevisionCalculator with the
  selected absolute series. No revision formula or event classification is copied
  into the runner. Current/prior consensus always compare the same absolute target,
  including when current FQ1 was historical FQ2. Activity results are reused only
  within one requested snapshot. All VALID/MISSING/INVALID statuses and reasons
  propagate; null values remain null, and valid zero remains zero.
- FeatureValue calculation identity is deterministic
  **`estimates_v1:<provider_profile_id>:<dataset_id>`**. Query/persistence uniqueness
  includes that full identity. `period_end` is the UTC research date (not forecast
  period end), `available_at` is T, and `calculated_at` is actual execution time.
  Legacy FeatureValue timestamps remain UTC-naive in storage: input aware timestamps
  are converted to UTC before stripping tzinfo, independent of database timezone.
- `get_snapshot(security_id, t, feature_ids=None)` retrieves only exact T and the
  runner's pinned calculation identity. It does not return a stale earlier row.
  New research timestamps recalculate moving consensus cutoffs, activity windows
  and horizon mapping even with no estimate events. Run-twice persistence preserves
  the same IDs, values, statuses, evidence and original execution timestamps.
- Dataset locking coordinates feature publication with ingestion/replay/sealing;
  the existing FeatureValue unique constraint remains the final duplicate guard.
  Changed content at an already-published identity raises
  PUBLISHED_FEATURE_CONFLICT_USE_NEW_DATASET instead of overwriting historical
  snapshots. A later build receives a different identity and cannot mutate the
  older snapshot. Injected persistence failure rolls back the whole feature batch.
- Provenance retains feature definition, T, absolute target, requested horizon,
  fiscal/reporting/coverage resolver IDs, profile/build/manifest/normalization and
  calculation versions, current/prior consensus IDs, means/fingerprints, lookback
  and formula evidence. Activity stores compact window-only transition/version/
  predecessor/alias/fiscal references, selected contributors, U/D/Z, completeness
  and counts. It does not copy raw payloads or full historical event streams. The
  existing schema has no estimate_feature_evidence table, so structured provenance
  retains relational IDs leading to states, members, observations and raw objects.
- Added **64 PostgreSQL tests**: independent exact registry/metadata expectations;
  every one of the 15 numeric outputs; registry and FeatureValue idempotency;
  no-estimate/horizon/prior/current failures and invalid numeric/denominator
  propagation; incomplete/no-directional activity; dataset/backfill and provider
  separation; same-target rollover at the reporting microsecond; consensus/activity
  ageing; compact provenance and relational traceability; non-UTC session timestamp
  roundtrip; actual fundamental/market loop isolation; unchanged market metadata;
  registry tampering/unapproved IDs; ambiguous/future/other-build series; atomic
  batch rollback; and protection against overwriting a published snapshot.
- Final focused suite: **284 passed, 4 warnings**. Full suite: **367 passed,
  12 warnings**, zero failures. Warnings remain existing datetime.utcnow and
  SQLAlchemy Query.get deprecations. `git diff --check` passed. Alembic
  current/head remain **25e001**. Disposable PostgreSQL base -> head ->
  `1f4b7c9d2a31` -> head passed with **284 focused tests**, 4 warnings and
  15 estimate tables. Persistent database was not downgraded.
- Operational precondition: use ConsensusStateWriter to publish consensus through
  the requested T before calculating magnitude features. The runner reads those
  states; it does not rebuild consensus or ingest inputs. Seal input builds before
  production publication where possible; conflicting BUILDING-corpus reruns fail
  rather than rewrite. Synthetic qualification only; whole-history replay remains
  optimized for correctness, not provider-scale throughput. No raw consensus mean
  features, FQ2/FY2 feature permutations, rankings, factors, or signals were added.
- Next: **2.5.7 — Full Synthetic Integration**. Stop at this independently
  committed/pushed checkpoint; the complete raw-to-feature semiconductor lifecycle
  scenario has not started. Phase 2.5 implementation is NOT FROZEN; 2.5.7–2.5.8
  remain. Phase 2.6 is NOT STARTED.
  **RESUME FROM PHASE 2.5 CHECKPOINT 2.5.7**.
- Commit locator: `git log --format='%H' --grep='^Implement Phase 2.5 estimate feature runner$' -1`.

### Checkpoint 2.5.7 — Full Synthetic Integration
STATUS: COMPLETE

- Starting HEAD/origin: `97926a409a2cd4cf28ae6bcf9fd42144a00475c9`, master,
  clean after fetch/checkout/ff-only pull. Python 3.12.14 and local editable
  import verified. Baseline reproduced: **284 focused / 367 full**, 4 / 12
  warnings, zero failures. Repository specification is the source of truth.
- Added `tests/estimates/test_synthetic_integration.py`: one complete PostgreSQL
  lifecycle using `EstimateProvider` / `EstimateResponse`, real filesystem
  `RawStorage` / `EstimateRawArchive`, `SyntheticEstimateNormalizer`, and
  `EstimateIngestionRunner`. No final observations, consensus states, members,
  exclusions or FeatureValues are manually injected. Reference identities,
  qualified synthetic profile, alias/fiscal/coverage evidence and empty exact
  series are prerequisite fixtures. The fixture binds genuine service sessions
  to a rollback-only PostgreSQL transaction and writes bytes under pytest's
  temporary directory; it does not mock any calculation or persistence service.
- Helios Compute Semiconductor has four provider-scoped analysts A/B/C/D and
  explicit January-year-end FY2027 Q1 (2026-04-30), Q2 (2026-07-31) and FY
  (2027-01-31). Prior Q4/FY2026 reporting anchors and reciprocal fiscal links
  establish the horizons. Target and contributor resolvers are asserted before
  consensus. Fiscal dates validate metadata, never derive quarter identity.
- One tier-1 synthetic qualified profile documents stable contributors, complete
  economic/reporting coverage, sequence ordering, certified historical timing,
  corrections and withdrawals, adjusted diluted EPS, reported consolidated
  revenue, USD and compatible ordinary shares. Build A pins `synthetic_v1` and
  a manifest with a July 1 retrieval watermark. Archived evidence supplies raw
  links for fiscal/contributor/coverage/reporting prerequisites.
- Deterministic timeline (2026; UTC):
  - March 1 12:00: A/B/C NEW for Q1/Q2 EPS and revenue (12 observations).
  - **T1 April 20 12:00**: Q1=FQ1, Q2=FQ2.
  - April 25 12:00: A raises Q1 EPS .80→1.10, revenue 100→110 million.
  - April 27 12:00: B lowers Q1 EPS 1.00→.90, revenue 110→108 million.
  - May 1 12:00: D initiates Q1 EPS 1.20, revenue 120 million.
  - May 5 12:00: C withdraws both Q1 estimates; Q2 remains active.
  - May 8 12:00: provider corrects A's earlier EPS revision 1.10→1.00.
  - May 9 12:00: A raises Q2 EPS 1.20→1.50, revenue 120→150 million.
  - **T2 May 10 12:00**: Q1 remains FQ1; Q2 remains FQ2.
  - May 20 20:05: Q1 primary reporting context (16:05 America/New_York).
  - May 21 09:00: B raises Q2 EPS 1.40→1.60, revenue 140→160 million.
  - **T3 May 21 12:00**: Q2=FQ1; 25 immutable estimate observations total.
- `ConsensusStateWriter.replay` publishes event/evidence boundaries before
  magnitude feature calculation, preserving the 2.5.6 orchestration contract.
  `EstimateConsensusRepository` statistics are checked against production replay
  through `EstimateEventResolver` and `ConsensusCalculator`, as well as explicit
  expected values and memberships. Revenue payloads in millions normalize to
  **base USD**, with exact source multiplier and record locator checks.
- T1 Q1 EPS mean/median/low/high/sample stddev/count =
  **1 / 1 / .80 / 1.20 / .20 / 3**. Revenue equivalents in USD =
  **110000000 / 110000000 / 100000000 / 120000000 / 10000000 / 3**.
  Both activity counts are valid zero; breadth is explicitly
  MISSING / NO_DIRECTIONAL_REVISIONS, never a zero substitute.
- T2 Q1 members are A/B/D: EPS **1.00/.90/1.20**, revenue
  **110/108/120 million**. Mean EPS is `3.10/3`; mean revenue is
  `338000000/3`, calculated under the existing 38-digit HALF_EVEN policy.
  EPS absolute change is approximately **.0333333333**, scaled change
  **.0322580645**, revenue ratio **.0242424242**. For each metric the production
  activity calculator verifies U=1, D=1, breadth=0, revision_count=2,
  initiation_count=1 and withdrawal_count=1. Neither correction nor initiation
  nor withdrawal contributes another revision or directional vote.
- Correction PIT: one microsecond before May 8 12:00 selects A=1.10; exactly at
  availability selects A=1.00. Distinct original/corrected states remain queryable
  after all subsequent events and after the complete rerun. Original observation
  remains stored; the corrected member references its immutable predecessor.
- Reporting uses `ActualReportingResolver.project_filing` over synthetic archived
  primary `Filing` / `FinancialFact` evidence, with relational reporting-fact
  linkage. At release-minus-one-microsecond Q1 is unreported/FQ1 and Q2 FQ2;
  exactly at release Q1 is reported/historical and Q2 FQ1. Q2's absolute ID and
  consensus state ID remain unchanged at the release boundary: no duplication
  just for horizon rollover. Repeated reporting projection returns the same ID.
- T3 Q2 EPS members **1.50/1.60/1.60**, mean `4.70/3`; revenue members
  **150/160/160 million**, mean `470000000/3`. Both current/prior state IDs in
  the persisted EPS feature resolve to the same absolute Q2 series/build;
  the April 21 cutoff independently confirms Q2 was then FQ2. Exact persisted
  38-digit-policy representative results:
  - EPS absolute: `0.1666666666666666666666666666666666667`.
  - EPS scaled: `0.10638297872340425531914893617021276598`.
  - Revenue ratio: `0.11904761904761904761904761904761904764`.
  - Each metric: U=2, D=0, breadth=+1, revision_count=2.
- All **15** registry entries are requested/persisted at T1/T2/T3 and later
  ageing snapshots, with exact status/null/target assertions:
  - The six FQ1 magnitude IDs at 7/30 days (EPS absolute/scaled and revenue
    percentage) plus both revision counts are VALID at T1:
    **8 VALID, 7 MISSING**.
  - T2/T3: those six magnitude IDs plus both counts and both breadth IDs are
    VALID: **10 VALID, 5 MISSING**.
  - The three 90-day FQ1 magnitude IDs are intentionally NO_PRIOR_CONSENSUS at
    T1/T2/T3 because initial history begins March 1.
  - Both 30-day FY1 IDs are intentionally NO_ESTIMATES: annual fiscal identity
    and horizon exist, but no annual analyst estimates were manufactured.
  - Both breadth IDs are additionally NO_DIRECTIONAL_REVISIONS at T1.
- Feature ageing without new observations: June 8 12:00 moves A's May 9 revision
  exactly onto the excluded cutoff, leaving count=1/breadth=+1; June 20 09:00
  ages out B's May 21 revision, leaving count=0 and missing breadth. Q2 consensus
  remains unchanged. Feature timestamps are distinct requested research snapshots.
- Build B contains the same initial 12 observations through real ingestion, with
  separate evidence and published consensus. Its T3 Q2 EPS change/count are zero,
  distinguishable from A's positive change/count=2. Event replay, consensus and
  FeatureValue queries remain build-pinned; calculation identities and feature
  IDs differ and B cannot mutate A's result.
- Full pipeline rerun proves unchanged complete row fingerprints (including IDs,
  timestamps, values, evidence and fingerprints) for RawDataObject, observations,
  dataset memberships, contributors, states, members, exclusions and FeatureValue.
  IngestionRun rows intentionally record each attempt; repeated records_saved=0.
  Distinct event payloads retain eight distinct content hashes and archive paths.
- Provenance walk starts at persisted T3 EPS FeatureValue and checks calculation
  identity/version, profile, dataset/manifest/normalization version, absolute Q2,
  both consensus states, members, selected immutable observations, contributor
  and alias/fiscal evidence, dataset membership, SHA-256-verified raw bytes and
  source record locators. Horizon provenance reaches the actual reporting event
  and its supporting fact. Selected evidence is bounded by each applicable cutoff.
- **Defect found and fixed:** the first complete rerun preserved every downstream
  table but appended duplicate raw rows for the identical transport receipt.
  `EstimateRawArchive.save` now reuses identical provider/request/endpoint/content/
  receipt-time/metadata evidence, verifying original bytes before reuse. A
  PostgreSQL transaction advisory lock serializes receipt retries; no migration
  is needed. Later retrievals remain distinct, even when bytes match. Legacy
  RawStorage and frozen Phase 2.1/2.2/2.4 implementations are unchanged. Added two
  focused regressions for receipt identity/isolation and rejection of corrupted
  original bytes without silent repair. An intermediate savepoint-lifetime issue
  in this fix was caught by PostgreSQL tests and resolved before validation.
- Validation: **287 focused tests, 19 warnings; 370 full tests, 27 warnings;
  zero failures**. Warnings are existing datetime.utcnow / Query.get deprecations,
  now also exercised by the full raw-to-feature path. `git diff --check` passed.
  Alembic current/head **25e001**; no migration or model changes. Disposable
  PostgreSQL base → head → `1f4b7c9d2a31` → head and the complete focused suite
  passed; the persistent research database was not downgraded.
- Remaining limitations: synthetic qualification only; no real commercial feed,
  licensing, live contributor semantics or actual-reporting coverage qualified.
  Whole-history replay favors correctness over throughput. Publication still
  requires the documented consensus-through-T orchestration. Checkpoint 2.5.8's
  final implementation audit remains a separate gate; this integration success
  does not declare the implementation frozen or ready for independent review.
- Next: **2.5.8 — Final Implementation Audit**. Phase 2.6 is NOT STARTED.
  Stop at the clean pushed 2.5.7 checkpoint to preserve a careful separate audit.
  **RESUME FROM PHASE 2.5 CHECKPOINT 2.5.8**.
- Commit locator: `git log --format='%H' --grep='^Add Phase 2.5 full synthetic integration$' -1`.


### Checkpoint 2.5.8 — Final Implementation Audit
STATUS: Phase 2.5 IMPLEMENTATION COMPLETE — READY FOR INDEPENDENT REVIEW — NOT YET FROZEN

- Starting HEAD/origin: `15055cf719d450ab1fc1c28421660f8e5b2190af`, clean master
  after fetch/checkout/ff-only pull. Python 3.12.14 and local package import verified.
  Baseline reproduced: **287 focused / 370 full**, **19 / 27 warnings**, zero failures.
- Audited all 40 requested areas against the complete specification, source code,
  PostgreSQL models/migration and tests, and Git history. Evidence and operational
  recipe: [phase-2.5-implementation-audit.md](phase-2.5-implementation-audit.md).
  Final audit verdict: **PASS after corrections**; this is not an architecture freeze.
- Defect 1: conflicting reporting supersession branches/cycles were reduced to no
  rows; complete coverage could then incorrectly classify the quarter as unreported.
  Reproduced both failures against PostgreSQL. Explicit evidence conflict handling
  now returns UNKNOWN with evidence references, blocking false horizon reopening.
- Defect 2: dataset-wide evidence resolution allowed unrelated alias conflicts to
  exclude valid contributors, materialized other companies' evidence, and replayed
  unrelated alias boundaries. Reproduced all three failures. Scoped SQL evidence
  loading and relevant publication boundaries remove that interference. Connected
  PIT-visible replacement components preserve cross-key/target supersession and
  branch/cycle detection. An intermediate cross-key branch regression was caught
  and corrected before final validation; no original assertion was weakened.
- Added **8 regressions** across resolver/consensus tests. Extended the existing
  production-path Helios persisted-feature provenance walk to verify the registry
  definition, dataset/profile and all horizon fiscal/reporting/coverage raw links.
  The T3 scaled EPS feature remains exactly
  `0.10638297872340425531914893617021276598`, comparing the same absolute Q2.
- Changed production files: `research/estimates/resolvers.py`, `consensus.py`,
  `revisions.py`. No model/migration change or financial formula change. Added audit
  documentation and corrected README's stale implementation-status rows.
- Final validation: **295 focused tests, 19 warnings; 378 full tests, 27 warnings;
  zero failures**. Disposable PostgreSQL verifier passed base -> head -> prior
  head -> head, **15/0/15 estimate tables**, then **295 tests / 19 warnings**.
  `git diff --check` passed. Warnings remain shared datetime/SQLAlchemy deprecations.
- Alembic persistent current/head: **25e001**. Estimate metadata comparison against
  migrated PostgreSQL: **no differences**. Disposable base -> head ->
  `1f4b7c9d2a31` -> head verification passed as recorded above.
  Persistent database was never downgraded.
- Frozen regression: pre-implementation base `00fa6b0e5ef5b34b95077403eab3717cd69cc792`.
  Research diff outside `research/estimates/` is empty. Inspected meaningful shared
  changes (model registration, optional raw receipt timestamp, additive INVALID
  enum repair); Phase 2.1/2.2/2.4 calculation semantics are unchanged. Existing
  fundamental/market runner exclusion tests remain passing.
- Orchestration decision **A**: explicit staged architecture is acceptable. Call
  `ConsensusStateWriter.replay(series_id, through=T)` for all required series
  before magnitude features, then calculate the requested snapshot and commit.
  README now links to the complete recipe. Reader APIs do not prove publication
  completeness; stale publication is not evidence of no economic change.
- Genuine limitations: synthetic provider qualification only; no licensed real
  event feed/reporting coverage verified; whole-security and per-event prefix
  reconstruction and serialized build writers are not provider-scale qualified.
  FeatureValue evidence IDs live in JSON rather than the separate relation
  proposed in spec §C (already documented at 2.5.6). The persisted trace has no
  missing logical link, but feature-to-evidence FKs are absent; independent review
  must assess this representation before freeze. Existing deprecation warnings remain.
- Production provider gate remains: historical analyst events, stable contributor
  identities, withdrawals, correction semantics, justified PIT dissemination,
  adjusted EPS methodology, comparable share basis, coverage completeness, and
  licensing/retention rights. No inferred history or weaker model is permitted.
- Next action: **independent Phase 2.5 QC/architecture review**. Phase 2.5 is
  **NOT YET FROZEN**. Phase 2.6 is **NOT STARTED**. Affected historical outputs,
  if any, require a new build; published rows are not silently rewritten.
- Commit locator: `git log --format='%H' --grep='^Complete Phase 2.5 implementation audit$' -1`.

## Strategic architecture / documentation review — 2026-09-21

- Starting checkpoint verified: `a5a80ef8bc594500000b4f357192bc82513b5bb8`,
  master = origin/master, clean after fetch/checkout/ff-only pull.
- **Strategic architecture/documentation review complete — CONDITIONAL PASS.**
  Detailed findings and R1–R9 gates: [STRATEGIC_ARCHITECTURE_REVIEW.md](STRATEGIC_ARCHITECTURE_REVIEW.md).
- Rewrote README for the human learner; updated blueprint, learning curriculum,
  architecture/data-source summaries and current state. Corrected stale estimate
  status, provisional EPS-floor/breadth descriptions and a rolling-target example
  in the educational blueprint; frozen specification/formulas were not changed.
- New future designs: prediction/outcome ledger; validation/calibration/learning;
  historical replay/golden cases; data-source trust; provider risk register;
  platform comparison/valuation/business quality; news/macro/event intelligence;
  external independent-review handoff; data durability/recovery.
- Major conditions: membership knowledge time/coverage, SEC same-day visibility
  and comparative fiscal context, Yahoo raw/action qualification, macro vintages,
  immutable prediction publication, generic raw-storage collision protection,
  production estimates qualification/orchestration, and tested off-host recovery.
- New validation/learning architecture is documented only. No tables, application
  code, migrations, backup scripts, commercial feeds, ML models or phase implementation
  were added. Source licensing and production-scale behavior remain unqualified.
- Baseline full suite: **378 passed, 27 warnings, zero failures**, 101.75s.
  Final full suite: **378 passed, 27 warnings, zero failures**, 101.74s.
  Existing datetime/SQLAlchemy deprecations remain. Prior focused audit: 295 passed;
  full suite includes estimates, but no separate focused rerun is claimed here.
- Alembic current/head independently checked: **25e001**. No database migration
  was run. `git diff --check` passed; local file-link validation passed across all
  16 changed/new Markdown documents. No backup restore or empirical backtest run.
- Recommended sequence preserves existing 2.6–2.11 numbering; adds ledger,
  valuation and evidence prerequisites plus proposed 2.12/2.13 extensions.
  Immediate next action: independent Phase 2.5 review and foundation remediation
  design before more features. This strategic review is not the independent freeze.
- **Phase 2.5 IMPLEMENTATION COMPLETE / READY FOR INDEPENDENT REVIEW / NOT YET
  FROZEN. Phase 2.6 NOT STARTED.**
- Documentation commit locator:
  `git log -1 --format='%H' --grep='^Document closed-loop research and validation architecture$'`.
  Commit/push and final HEAD/origin/clean-tree verification are the publication gate;
  a committed file cannot contain its own final commit hash.

## Qualitative and alternative research architecture extension — 2026-09-21

- Starting master/origin verified clean at
  `4c992ca4e2c74773d53ce4135febb3cce56c6deb` after fetch/checkout/ff-only pull.
- Documentation extension complete. Created
  `QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md`,
  `RESEARCH_EVIDENCE_HIERARCHY.md` and `ALTERNATIVE_DATA_ROADMAP.md` under docs.
- Studied public Morningstar methodology/help: business strategy, bull/bear cases,
  moat, cash-flow valuation, financial strength, uncertainty and capital allocation.
  Source links/access limitations are recorded in the detailed report. No subscriber
  content copied, proprietary rating cloned or systematic data rights assumed.
- Designed BusinessQualityResearch dossiers, evidence/counter-evidence moat
  hypotheses and CompanyStrategicEvents with versioned timing/lifecycle. No scores
  or ratings implemented. Professional-research contract distinguishes reference,
  legitimate user-supplied, licensed structured and redistributable access modes.
- Designed source independence/corroboration, preserved disagreements, typed thesis
  graph and claim/risk/evidence-family outcome evaluation. No opaque AI score and
  no silent production self-modification. Alternative roadmap covers all requested
  categories; cost/usefulness are preliminary hypotheses, all feeds unqualified.
- Updated README, blueprint, architecture summary, platform comparison, event plan,
  prediction/outcome ledger, validation/learning controls, external review handoff
  and this state. Local file/anchor checks passed across 12 Markdown files.
- Documentation-only: no application code, migrations, tests, formulas, scraping,
  datasets, backup scripts or production connectors changed. Existing R1–R9 gates
  and strategic CONDITIONAL PASS remain; this extension does not remediate them.
- **Phase 2.5 remains IMPLEMENTATION COMPLETE / READY FOR INDEPENDENT REVIEW /
  NOT YET FROZEN. Phase 2.6 NOT STARTED.** Existing phase numbering retained.
- Commit locator:
  `git log -1 --format='%H' --grep='^Document qualitative and alternative research architecture$'`.
  Publication requires push and verification that HEAD equals origin/master and
  the working tree is clean; this file cannot embed its own final commit hash.
- Final validation: `python -m pytest -q` — **378 passed, 27 existing deprecation
  warnings, zero failures**, 101.76s. `git diff --check` passed. No separate focused
  suite, live-source qualification, empirical research test or migration run claimed.

### Foundation remediation checkpoint 2 — raw/build/publication and macro

Content-addressed atomic raw storage and verified replay implemented; connector
request metadata retained without credentials. Feature conflicts now require a
new version/build; PostgreSQL rejects UPDATE. Portable deterministic research
manifest and fail-closed replay source policy added. Canonical macro vintages and
synthetic adapter prove initial/revised as-of selection. Yahoo adjusted-close
parsing and UTC/receipt boundary repaired; provider fields explicitly unqualified.
No frozen formula changed. Focused 36 foundation/market and 22 integration/cache
checks passed; migrations current/head fnd003. Full regression and disposable
recovery verification follow. Live ALFRED and market providers remain unqualified;
Phase 2.6 NOT STARTED. Next: durability scripts, restore drill and final report.

### Foundation remediation checkpoint 3 — local durability and final safeguards

Implemented consistent exported-snapshot compressed PostgreSQL backup plus raw
inventory, SHA256/size checks, manifest and table/query hashes. Restore always uses
a new generated DB/raw root; verify checks Alembic, counts/content, raw provenance
and a nonempty deterministic FCF result before dropping its own DB. Disposable
base→head→25e001→head and restore drills passed at fnd003 and fnd004. Off-host storage
is planned, not configured. Existing local raw metadata includes unavailable files;
no fabricated repair or destructive cleanup was performed.

Additional guards: immutable raw metadata; monotone membership knowledge versions;
non-synthetic membership raw provenance; membership evidence in market output;
exact provider qualification per manifest; ambiguous feature builds require pinning.
Preserved the estimate raw-integrity error contract after regression exposed an
exception-type mismatch. Formula mathematics remains unchanged across all phases.
33 new foundation tests now pass; final full suite and final drill pending below.
README, field-level trust contract, provider register, machine/recovery guides,
validation plan, review handoff and detailed audit updated. Overall CONDITIONAL PASS;
remaining source/coverage/legacy-data/operations gates mean MORE FOUNDATION
REMEDIATION REQUIRED. Phase 2.6 NOT STARTED. Phase 2.5 NOT YET FROZEN.


### Foundation remediation final validation and publication

- Starting checkpoint: 125e780c2731dd831ebbdd69abf92332e5578dde, clean master/origin.
- Implementation checkpoints pushed: f8f83ea (universe PIT), 6ce5ea6 (raw/build,
  feature and vintage foundation), 36980d9 (durability, guards and documentation).
- Final full suite: 411 passed, 8 existing deprecation warnings, 0 failures, 103.45s.
  Warning reduction comes from removing the generic RawStorage utcnow path.
- Final disposable drill on committed 36980d9: fresh base→fnd004→25e001→fnd004;
  33 foundation tests passed; compressed snapshot backup and separate disposable
  restore passed exact table counts/content, raw inventory/SHA256/size, Alembic
  current and one nonempty as-of FCF=80 result. Tampered dump rejected. Only created
  databases were dropped; no persistent database downgrade/restore took place.
- Synthetic local report: /tmp/irs-foundation-drill-7lmfyqq7/restored_raw/restore_report.json.
  The deliberately tampered dump in that drill directory is not a recovery backup.
- Persistent Alembic current/head fnd004. Metadata inspection found two pre-existing
  HistoricalUniverse sector/industry index omissions, unchanged from the baseline;
  no new foundation column/constraint/index discrepancy was found.
- Frozen calculator, adjustment engine, temporal price/action resolver and all
  research/estimates files have no diff from starting checkpoint. Market changes
  are membership selection/provenance and persistence; fundamental changes are
  persistence and explicit build selection. Estimate normalization only preserves
  receipt/request provenance and the established integrity-error contract.
- All changed Markdown local file links pass; git diff --check passes. No dumps,
  raw datasets, secrets or proprietary material added to Git. No new investment
  feature families, backtester, factors, signals, ML or Phase 2.6 implementation.
- Final audit verdict CONDITIONAL PASS with the seven requested risk assessments
  and R2/R3/R8 remaining gates in FOUNDATION_REMEDIATION_AUDIT.md. Recommendation:
  MORE FOUNDATION REMEDIATION REQUIRED. Phase 2.5 remains NOT YET FROZEN.
- Final documentation commit locator:
  `git log -1 --format='%H' --grep='^Record foundation remediation validation and remaining gates$'`.
  Publication is complete only after HEAD/origin equality and a clean working tree.
