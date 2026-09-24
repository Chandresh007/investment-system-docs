# Phase 2.9 — Historical Replay, Prediction Ledger, and Outcome Evaluation Contract

Status: **ARCHITECTURE APPROVED; IMPLEMENTATION SPEC FROZEN**.

This is the authoritative implementation contract for Phase 2.9. It preserves
the approved decisions in
[`phase-2.9-historical-replay-backtest-architecture-review.md`](phase-2.9-historical-replay-backtest-architecture-review.md).
If a detail is not repeated here, that review remains authoritative. Foundation
and Phases 2.2, 2.4, 2.5, 2.6, 2.7, and 2.8 remain frozen. Phase 2.9 consumes
their immutable artifacts and must not change a source formula, factor weight,
signal threshold, typed rule, proof, coverage rule, or quality rule.

The implementation starts from approved checkpoint
`67b6e36a4f77de4648b67e3468859218f3f653e8`, with Alembic head `p28002` and
the reproduced baseline of 905 passing tests and 8 existing warnings.

## 1. Scope and separation

Phase 2.9 v1 implements:

```text
historical state at T
  -> frozen upstream artifacts
  -> full-population research snapshot
  -> false-to-true episode-start prediction

published snapshot/prediction
  -> separately qualified future data
  -> versioned outcome
  -> descriptive signal-event backtest
```

Replay/prediction code and outcome code are separate packages, repositories,
sessions, manifests, and transaction lifetimes. A replay transaction cannot read
an outcome or backtest table. An outcome evaluator cannot update an upstream
artifact, replay checkpoint, snapshot, or prediction. A historical-backfill
orchestrator commits and closes replay work before opening outcome work.

V1 includes weekly chronological replay, a complete snapshot population,
episode-start predictions, market/fundamental/estimate outcomes, descriptive
case/control aggregation, deterministic resume, and synthetic/development-scale
qualification. It excludes portfolios, weighting, optimization, transaction
costs, significance tests, bootstrap inference, multiple-testing decisions,
calibration, threshold changes, ML, promotion, and every Phase 2.10 behavior.

Historical reconstructions use origin `HISTORICAL_REPLAY` or `EDUCATIONAL` and
never claim that the system actually published them in the past. The same
snapshot, prediction, and outcome identities also permit later `LIVE_SHADOW` and
`LIVE_CHAMPION` publication; no parallel live ledger is allowed.

## 2. Frozen policy identities

The exact v1 policy identifiers are:

```text
schedule_policy                 = weekly_friday_2000_new_york_v1
calendar_policy                 = us_equity_regular_sessions_v1
universe_policy                 = peer_primary_common_equity_v1
episode_policy                  = valid_false_to_true_weekly_v1
prediction_policy               = phase_2_9_episode_prediction_v1
signal_objective_policy         = phase_2_8_signal_objectives_v1
entry_policy                    = next_regular_session_close_v1
horizon_policy                  = calendar_month_from_entry_v1
terminal_policy                 = terminal_total_return_v1
spy_benchmark_policy            = spy_total_return_aligned_v1
industry_benchmark_policy       = pit_dynamic_industry_equal_weight_v1
market_path_policy              = qualified_tri_path_v1
fundamental_outcome_policy      = next_quarter_first_reported_v1
estimate_outcome_policy         = same_absolute_target_consensus_v1
aggregation_policy              = descriptive_signal_event_v1
percentile_policy               = decimal_linear_percentile_v1
```

Changing any semantic policy, schedule, universe, signal version, entry rule,
horizon, label, benchmark, terminal treatment, population filter, partition, or
aggregation creates a new version/build/definition. It never mutates a published
artifact.

All deterministic numeric work uses finite `Decimal` values under the existing
38-digit `ROUND_HALF_EVEN` policy. Binary floats, NaN, infinity, implicit
clipping, and provider adjusted-close inputs are prohibited.

## 3. Weekly replay schedule and research timestamp

`weekly_friday_2000_new_york_v1` yields one ordered checkpoint for each local
Friday whose local date is inside the inclusive requested date range, plus the
immediately preceding scheduled Friday as a state-only warm-up checkpoint.

- Research timestamp T is exactly `20:00:00 America/New_York` on the Friday,
  converted to and persisted as timezone-aware UTC.
- New York timezone conversion is authoritative. T is `01:00Z` during standard
  time and `00:00Z` the following UTC day during daylight time. A hard-coded UTC
  hour is invalid.
- `schedule_key` is the local date in `YYYY-MM-DD` form under this policy.
- The effective session/date is the latest qualified regular US equity session
  on or before the local Friday. A Friday holiday or full market closure keeps
  Friday T but uses the preceding qualified session as effective date.
- Early closes do not change T. Official session opens/closes, holidays, DST,
  and exceptional closures come from the pinned calendar build.
- The warm-up is processed and snapshotted normally, but its predictions and
  outcomes are excluded from the requested evaluation interval.
- Checkpoints are processed strictly chronologically. No later date may be
  processed while an earlier scheduled checkpoint is incomplete or failed.

T is an information cutoff, not the wall-clock reconstruction time. Every replay
input must satisfy its frozen PIT contract and justified `available_at <= T`.

## 4. Run identity, manifests, and quality admission

One replay run has an immutable semantic fingerprint over its date range,
warm-up, schedule/calendar/universe/episode policies, origin, requested quality,
provider qualifications and versions, source datasets/build templates, exact
feature/factor/signal versions and definition hashes, Git commit, Alembic
revision, dependencies, and all configuration versions.

Each date has exact content-addressed `ResearchBuildManifest` records. The
existing eleven-field manifest contract is reused; Phase 2.9 does not invent a
second generic build table. Replay-specific pins live in the canonical
`configuration` and existing dataset/version fields. Every manifest is rebuilt,
rehashed, persisted, and compared before use.

For every provider/version and every date, call the existing
`ResearchBuildManifest.require_historical_replay` over the complete required
coverage interval, including the longest source lookback—not merely at T.
Requested and realized quality are stored separately.

The exact admission rules are:

- `DEVELOPMENT` is allowed only for repository-owned synthetic sources or an
  explicit `DevelopmentOverride(requested_by, reason, reference)`. Its
  limitations and override are persisted downstream.
- `PIT_PARTIAL` is not an admissible target for a historical performance run.
- `PIT_QUALIFIED` requires every provider to pass historical-replay, trust,
  version, evidence, and full-interval coverage gates.
- `SURVIVORSHIP_QUALIFIED` additionally requires historical universe/security,
  PIT actions, delisted/lifecycle history, and `SURVIVORSHIP_SAFE` capabilities.
- `PRODUCTION_RESEARCH` requires both production/raw/licensing admission and all
  survivorship-safe capabilities.

A development override is legal only when the requested target is
`DEVELOPMENT`; it cannot waive a stronger claim. A request that cannot meet its
target fails before snapshot/prediction publication. No automatic downgrade is
allowed.

The quality order is exactly:

```text
DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED
            < SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH
```

## 5. Historical universe reconstruction

For a checkpoint, candidate history is resolved only from the exact sealed
classification, eligibility, and security-identity datasets pinned by the
manifest. The resolver must select the latest evidence version known by T before
testing its effective interval `[start_date, end_date)`.

A security is eligible only when all of these hold at the effective date/T:

```text
latest membership version known by T is economically effective
AND eligibility universe is peer_primary_common_equity_v1
AND exactly one primary security represents the company
AND exact SecurityIdentityHistory is ACTIVE
AND exact historical taxonomy release/classification is usable
AND every required provider/source admission passes
```

Ticker, exchange, lifecycle, sector, industry, and sub-industry come from
historical evidence, never mutable `Security.ticker`, `Security.is_active`, or
today's classification. Securities later delisted, acquired, merged, or bankrupt
remain candidates at dates when they were eligible. Future-known memberships,
identity changes, and taxonomy corrections are excluded.

The checkpoint records every candidate with an `ELIGIBLE` or `EXCLUDED` snapshot
and stable exclusion reason. Each evaluated formal industry node receives one
`EVALUATED_NODE` snapshot. Empty or ambiguous sealed universes fail the date.
There is no market-cap or liquidity screen in v1 and no fallback to a current
universe or broader taxonomy node.

## 6. Frozen upstream artifact use

For each eligible security and evaluated node, replay invokes or reuses the
existing supported feature, peer, factor, and signal runners. Reuse is permitted
only when definition/version/hash, subject, T, effective date, dataset/build,
origin, admission, and manifest all match exactly. A future, other-T,
other-subject, other-build, or generic-latest row is never a cache hit.

The authoritative Phase 2.8 population remains `SignalResult`:

```text
VALID / fired=true
VALID / fired=false
MISSING
INVALID
```

Replay publishes every requested state and complete condition graph. It never
recalculates a frozen factor/signal formula and never substitutes zero for
missing evidence.

## 7. Research snapshots

One immutable snapshot is published for every checkpoint candidate and every
evaluated node. Its subject state is exactly `ELIGIBLE`, `EXCLUDED`, or
`EVALUATED_NODE`.

A snapshot pins or references:

- checkpoint/run, T, effective date, origin, creation time, and
  `decision_ready_at`;
- exactly one security or industry-node/group subject;
- historical ticker/exchange/lifecycle/classification as known at T;
- exact membership, eligibility, identity, classification, taxonomy-release,
  and peer-group evidence IDs where applicable;
- signal build, upstream factor build, and complete research-manifest chain;
- requested and realized quality, subject admission, and limitations;
- expected/actual factor and signal counts;
- canonical ordered factor-value ID and signal-result ID lists plus independent
  ordered set hashes;
- warning/contradiction counts and a complete content fingerprint.

For `HISTORICAL_REPLAY` and `EDUCATIONAL`, `decision_ready_at = T`. For live
origins it is `max(T, actual publication time)`. All predictions from one
snapshot share that anchor; a later decision requires a new snapshot/run.

The snapshot references authoritative `FactorValue`, `SignalResult`, membership,
and classification rows. It does not copy factor components, signal conditions,
formula documents, raw bytes, or full upstream provenance. Exact ordered IDs,
builds, counts, and hashes make omission/substitution detectable.

An exact rerun reuses the snapshot after full content verification. Same identity
with different content fails and requires a new build/run. Published snapshots
reject `UPDATE` and `DELETE`.

## 8. Episode detection and prediction ledger

The v1 episode policy is `valid_false_to_true_weekly_v1`.

Compare the same permanent subject and same exact signal definition/version in
two immediately consecutive completed scheduled checkpoints of the same run:

- `VALID/false -> VALID/true` emits exactly one episode-start prediction.
- `VALID/true -> VALID/true` is continuation and emits none, regardless of
  strength/severity changes.
- The first later `VALID/false` closes the observable episode by derivation.
- A later immediately consecutive `VALID/false -> VALID/true` starts a new
  episode.
- Initial true, new-universe true, or true after `MISSING`, `INVALID`, excluded
  subject state, incomplete/failed checkpoint, or schedule gap is left-censored
  and emits no primary prediction.
- Security and industry-node identities never cross. Signal versions are separate
  series.

There is no cooldown, hysteresis, minimum-off duration, strength restart, or
mutable `signal_episodes` table. The immutable prediction is the v1 episode ID.

Each prediction references the exact current and prior `SignalResult`, snapshot,
run/checkpoint, `ResearchSignal`, signal/factor builds, origin, objective family,
episode/prediction policy, and expected outcome-set definition. It snapshots the
current `VALID/true` strength, severity, coverage, signal quality, and derived
prediction quality, all database-validated against the source artifacts.

Prediction quality is the weaker of replay/checkpoint, snapshot, and signal
quality. Future evidence never upgrades it. One current `SignalResult` may create
at most one prediction. Exact replay is idempotent; a new signal version or replay
build creates a separate artifact. Published predictions reject `UPDATE` and
`DELETE`.

The objective mapping is exactly `phase_2_8_signal_objectives_v1` from the
architecture review. Primary and diagnostic endpoints are fixed before outcomes
are opened; a missing primary endpoint cannot be replaced after inspection.

## 9. Entry policy

`next_regular_session_close_v1` is an evaluation convention, not a claimed fill.

- `decision_ready_at` is the snapshot's immutable anchor.
- The eligible execution session is the first qualified regular US equity
  session whose official open is strictly later than `decision_ready_at`.
- Entry time is that session's official close.
- Entry value is its exact valid, positive, qualified raw-as-traded close linked
  to the price observation/build. Provider adjusted close is forbidden.
- Search only the intended session and the next two scheduled regular sessions.
  Select the first exact eligible close and persist the delay/session count.
- Never use a stale last price, a same/finished bar, an intraday synthetic fill,
  or an unbounded future search.

Therefore an after-close decision enters at the next session close, a pre-open
decision may enter at that day's close, a decision exactly at the open enters at
the following session close, and a weekend/holiday decision enters at the next
eligible session close. Once the three-session window is complete, no entry
produces terminal `MATURE_MISSING/ENTRY_PRICE_UNAVAILABLE` for all price-path
labels. Before that decision can be made, those labels remain `PENDING`.

The current provider/build-unaware `MarketPrice` storage and Yahoo basis may be
used only in explicit `DEVELOPMENT` fixtures. It cannot support a PIT,
survivorship, or production outcome claim.

## 10. Market horizons and exit selection

Canonical horizons are exactly `3m`, `6m`, `12m`, and `24m` under
`calendar_month_from_entry_v1`.

1. Start from the actual entry session's New York local date.
2. Add the horizon's calendar months with end-of-month clamping.
3. Choose the first qualified regular-session close on or after that boundary.
4. Search only that session and the next two scheduled sessions for an exact
   valid security observation, recording actual exit and delay.

These are not 63/126/252/504-session horizons. Actual entry and exit timestamps
are always stored. A still-listed security with no valid exit inside the fixed
window is `MATURE_MISSING/EXIT_PRICE_UNAVAILABLE`. Benchmark paths use the
security's actual delayed entry and exit, never their own convenient interval.

## 11. Market outcome definitions and formulas

The code-owned immutable market definitions are:

```text
security_total_return_v1
security_price_return_v1
spy_total_return_v1
spy_excess_return_v1
pit_industry_total_return_v1
pit_industry_excess_return_v1
maximum_drawdown_from_entry_v1
maximum_favorable_excursion_from_entry_v1
forward_realized_volatility_v1
```

Let `P'_s` be the qualified split-continuous raw close and `D_s` qualified
ordinary distributions for session `s`. Only distributions economically after
entry close through exit close enter the holding path:

```text
W_entry = 1
W_s = W_(s-1) * (P'_s + D_s) / P'_(s-1)

TotalReturn = W_exit - 1
PriceReturn = P'_exit / P'_entry - 1
SPYExcess = SecurityTotalReturn - SPYTotalReturn
IndustryExcess = SecurityTotalReturn - IndustryTotalReturn
```

Splits preserve units and are not returns. The path uses the frozen Phase 2.4 TRI
corporate-action semantics through a qualified, versioned outcome price/action
build; it does not use adjusted close or change Phase 2.4.

Path-risk definitions include entry wealth one:

```text
running_peak_s = max(W_entry ... W_s)
drawdown_s = W_s / running_peak_s - 1
maximum_drawdown = min(drawdown_s)                 # [-1, 0]
maximum_favorable_excursion = max(W_s - 1)         # >= 0
g_s = ln(W_s / W_(s-1))
forward_volatility = sample_stddev(g_s) * sqrt(252)
```

Volatility requires at least 20 valid consecutive positive-wealth intervals;
otherwise it is mature missing with an explicit reason. Every numeric output is
finite and domain-checked.

For an industry-node prediction, the subject path is the PIT industry wealth
path. Its excess is industry return minus aligned SPY return, and its drawdown,
MFE, and volatility use that same node path. A node prediction is never copied to
constituent securities.

## 12. Benchmarks

V1 has exactly two benchmark families:

1. `spy_total_return_aligned_v1`: qualified SPY raw close/action history over
   the subject's exact actual entry/exit sessions.
2. `pit_dynamic_industry_equal_weight_v1`: the prediction-time stable industry
   node followed through explicit pinned node lineage. At each interval start
   close, equally weight primary securities whose membership, classification,
   identity, and lifecycle are effective and known then. Apply each next
   close-to-close qualified simple total return, including terminal outcomes,
   and rebalance only after the interval:

```text
r_industry,s = sum(r_i,s) / N_s
W_industry,s = W_industry,s-1 * (1 + r_industry,s)
```

At least three valid constituents are required at every weight-setting close.
Evidence learned at an interval's end can affect only the next interval. The
industry path is distinct from Phase 2.4's descriptive mean-log-return benchmark.
No present-day membership/classification substitutes for history. Missing stable
node lineage, membership, terminal coverage, or benchmark evidence makes the
label missing/lower-quality; it never triggers substitution with SPY. Sector
benchmarks are not in v1.

## 13. Terminal events, delisting, and censoring

`terminal_total_return_v1` applies without dropping failed securities:

- Qualified cash acquisition, liquidation, bankruptcy recovery, or other
  terminal proceeds enter wealth on the effective session; nominal cash then
  earns zero through the horizon.
- Qualified stock conversion uses the exact conversion ratio and successor
  observation on the effective session, then notionally liquidates to cash. V1
  does not stitch later successor performance.
- Legally evidenced cancellation with zero recovery sets terminal wealth zero,
  total return/drawdown `-1`.
- Delisting, suspension, or disappearance without qualified proceeds is
  `CENSORED/TERMINAL_VALUE_UNKNOWN`; the last quote is not carried and unknown is
  not assumed to be a total loss.
- A listed security lacking a valid fixed-window exit is
  `MATURE_MISSING/EXIT_PRICE_UNAVAILABLE`.
- Conflicting qualified terminal sources that the policy cannot resolve are
  `DISPUTED/TERMINAL_VALUE_CONFLICT`.

Censored, missing, and disputed cases remain in denominators and coverage. A
result containing unqualified terminal coverage cannot be called
survivorship-qualified. Sensitivity bounds, when independently evidenced, remain
diagnostics and never replace the null primary label.

## 14. Fundamental and estimate labels

V1 materializes only the approved small code-owned set.

At T, `next_quarter_first_reported_v1` freezes the latest eligible explicit
quarterly fiscal identity as baseline and its evidenced next-quarter successor as
target. Calendar dates never infer fiscal identity. The outcome uses the first
qualified primary reporting version available after T; a later amendment or
restatement cannot replace it under the same definition/build.

```text
next_q_revenue_yoy_growth_first_reported_v1                 ratio
next_q_revenue_growth_acceleration_first_reported_v1        percentage_points
next_q_operating_margin_change_from_baseline_first_reported_v1
                                                            percentage_points
next_q_fcf_margin_change_from_baseline_first_reported_v1    percentage_points
```

The first two reuse the frozen fundamental feature meanings for the explicit
target quarter. Margin labels subtract the exact baseline-quarter value known at
T from the target's first-reported value. Missing CapEx stays missing. If the
successor cannot be resolved at T, the fixed target is unavailable and the mature
label is `MATURE_MISSING/FISCAL_TARGET_UNRESOLVED`; the evaluator cannot choose a
future convenient quarter.

`same_absolute_target_consensus_v1` freezes at T the exact FQ1 absolute target,
provider/profile, metric, basis, diluted/share basis, currency, scope, and
consensus series. At T+30 or T+90 calendar days, use the latest state with
`state_available_at <= boundary` for that same series. The latest state must be
`VALID`; never resurrect an older valid state. Complete source coverage through
the boundary is mandatory.

```text
same_target_eps_consensus_scaled_change_30d_v1
same_target_eps_consensus_scaled_change_90d_v1
  = (future - baseline) / max(abs(baseline), abs(future)); both zero -> zero

same_target_revenue_consensus_pct_change_30d_v1
same_target_revenue_consensus_pct_change_90d_v1
  = (future - baseline) / baseline; baseline > 0 and future >= 0
```

If the fixed target is first reported before the estimate boundary, the outcome
is `CENSORED/TARGET_REPORTED_BEFORE_ESTIMATE_HORIZON`; it is not shortened and
does not roll to a new FQ1. Consensus movement and analyst event activity remain
distinct concepts.

## 15. Outcome maturity, versioning, and shared identity

`OutcomeEvaluator` operates only on published snapshots/predictions and qualified
future builds. Outcome rows have exactly one status:

```text
PENDING
MATURE_VALID
MATURE_MISSING
CENSORED
DISPUTED
```

- `PENDING`: the scheduled boundary/publication/search window has not completed;
  value and `matured_at` are null.
- `MATURE_VALID`: the boundary is reached and all definition-required qualified
  evidence is complete; numeric/text value is present as declared.
- `MATURE_MISSING`: maturity is reached but required evidence is absent, stale,
  outside its fixed search window, invalid, or coverage-incomplete; value is null
  and reason is required.
- `CENSORED`: a defined economic event prevents ordinary full observation.
- `DISPUTED`: admitted sources/corrections conflict and policy cannot choose.

Fundamental labels mature when their required qualified first report becomes
available or when a pinned coverage assertion proves it unavailable—not merely
when a price horizon passes. Estimate labels mature at their fixed boundary once
coverage through it and any report-before-boundary state are known. Market labels
do not mature before the exit boundary and three-session decision window.

Each row stores scheduled maturity; actual entry/exit or fiscal target; outcome
dataset cutoff; evaluation/maturity timestamps; value/unit; reason; outcome,
benchmark, and terminal quality; exact evidence/build references; ordered path
hash; optional `supersedes_id`; and fingerprint.

Outcome identity is snapshot + definition + decision anchor + horizon/fiscal
target + outcome manifest. A shared outcome is computed once per executable
snapshot/definition/build and linked to every applicable prediction. This avoids
duplicating a price path for co-occurring signals. Pending rows are real immutable
observations, never zero.

Same build/content evaluation is idempotent. New provider data, corrections, or
policy/definition changes require a new outcome build/row, optionally superseding
an earlier row; old pending/mature/disputed rows remain queryable. Published
outcome definitions, outcomes, and prediction links reject `UPDATE` and `DELETE`.

Outcome quality independently equals the weakest label source, raw reproducibility,
calendar, action/terminal, filing/estimate, and benchmark quality. It never
upgrades prediction quality.

## 16. Minimum storage boundary

The normalized v1 storage is exactly these eleven logical tables; no speculative
episode or slice table is added:

```text
research_replay_runs
research_replay_checkpoints
research_snapshots
research_predictions
outcome_definitions
research_outcomes
prediction_outcome_links
backtest_definitions
backtest_runs
backtest_population_members
backtest_results
```

Existing `research_build_manifests`, `FactorValue`, `SignalResult`, universe,
peer, price/action, filing/fundamental, and estimate artifacts remain authoritative.
Baseline and single-dimension slice aggregates share `backtest_results`; there is
no `backtest_result_slices` table. There is no v1 `signal_episodes` table.

Definitions, snapshots, predictions, outcomes, links, population members, and
results are insert-only. Run/checkpoint status and lease/attempt metadata are the
only controlled mutable operational state and become immutable at terminal
completion. Full-content verification follows conflict-do-nothing. Same semantic
identity with changed content fails with a stable use-new-build/version error.

Database checks and PostgreSQL insert guards enforce subject/T/build/signal/
snapshot consistency, copied prediction evidence, outcome decision anchors,
finite declared numeric domains, FK lineage, and quality non-upgrade.

## 17. Backtest definition and population

A replay reconstructs decisions; a backtest evaluates already published replay
artifacts. An immutable backtest definition pins:

- definition ID/version/hash/description;
- exact complete replay run and date range;
- temporal partition `DEVELOPMENT`, `VALIDATION`, or `HOLDOUT`;
- signal IDs/versions/hashes and objective mapping;
- schedule, warm-up, episode, universe, quality, origin, and selection rules;
- entry, horizons, outcomes, terminal, benchmarks, and outcome build;
- aggregation/percentile policies, slice dimensions, and minimum-reporting rules;
- Git/schema/dependency identity and canonical fingerprint.

The statistical unit is a signal episode start, not a position or portfolio.

For each episode-start prediction at T:

- the case is that exact prediction/current `SignalResult`;
- controls are the set union of same-checkpoint, same-scope, same exact signal
  definition/version observations that are `VALID/fired=false`, have eligible
  snapshots, and pass the same research-quality filter;
- each control `SignalResult` appears at most once per signal/T, regardless of
  how many cases start that date;
- fired continuations are neither cases nor controls;
- `MISSING` and `INVALID` are unavailable coverage, never negatives;
- excluded subjects remain universe coverage but not outcome controls;
- cases and controls use identical entry, horizon, label, benchmark, terminal,
  and outcome-build rules.

The immutable population table records every included/excluded candidate,
cohort, exact snapshot/signal/prediction, episode date, partition, decision, and
reason. It makes aggregation reproducible and does not claim controls are causal
matches. A security may be a control at multiple dates; dependence is disclosed.

## 18. Descriptive aggregation

For every cohort, exact signal, outcome/horizon or fiscal target, and permitted
slice, `descriptive_signal_event_v1` stores:

```text
eligible_count
mature_valid_count
mature_missing_count
censored_count
disputed_count
pending_count
terminal_count = valid + missing + censored + disputed
maturity_rate = terminal_count / eligible_count
coverage_rate = mature_valid_count / terminal_count  # null when terminal_count=0
mean
P10
P25
median (P50)
P75
P90
positive_value_count/rate, only where zero has declared meaning
positive raw-return count/rate for return > 0
positive excess-return count/rate for excess return > 0
```

Distribution values use only `MATURE_VALID` finite continuous values. Counts
always reconcile:

```text
eligible = mature_valid + mature_missing + censored + disputed + pending
```

Arithmetic mean is the Decimal sum divided by count. Under
`decimal_linear_percentile_v1`, sort ascending, let `h=(n-1)*p`, `i=floor(h)`,
and `f=h-i`; percentile is `x_i + f*(x_(i+1)-x_i)`, with the final element used
when `i=n-1`. One value returns itself. Empty distributions have null statistics.

Case-minus-control differences in mean, median, and positive rate are stored only
when both corresponding aggregates are defined and retain both source result
references/counts. No p-value, confidence interval, significance flag, Sharpe,
Sortino, alpha, IC, rank spread, portfolio return, or causal claim is produced.

V1 persists overall rows and only these single-dimension slices: signal,
horizon/label, fired severity, prediction/research quality, and historical sector
at T. It does not precompute multi-dimensional Cartesian slices.

## 19. Checkpoint, resume, and failure semantics

Replay run transitions are exactly:

```text
PENDING -> RUNNING
RUNNING -> COMPLETE
RUNNING -> FAILED       # zero completed checkpoints
RUNNING -> PARTIAL      # nonzero, nonfinal completed watermark
RUNNING -> CANCELLED    # explicit operator action only
FAILED  -> RUNNING      # explicit deterministic retry
PARTIAL -> RUNNING      # explicit deterministic resume
```

`COMPLETE` and `CANCELLED` are terminal. Checkpoint states are `PENDING`,
`RUNNING`, `COMPLETE`, and `FAILED`. A run may be `COMPLETE` only when every
expected checkpoint, including warm-up, is complete, chronologically contiguous,
and its ordered hash matches.

One scheduled T is the outer transaction:

```text
validate manifests and quality
reconstruct complete candidate universe
run/reuse bounded upstream artifacts
insert all snapshots
derive and insert eligible predictions
verify identities/counts/content hashes
mark checkpoint COMPLETE
commit
```

Any failure rolls back the entire date; a partial date is never complete. Later
dates stop. Existing runner savepoints/batches remain inside the date transaction.

Each checkpoint has deterministic identity from run fingerprint + ordinal + T.
Resume verifies all completed checkpoint manifests, counts, and publication
hashes, then restarts at the first non-complete date. It never skips a failure,
repairs a conflict, or creates duplicate predictions. Changed manifests require
a new run. An expired `RUNNING` lease records a failed attempt and sets the run to
`FAILED` or `PARTIAL` according to the completed watermark before explicit retry.

## 20. Quality and provenance propagation

Quality never upgrades:

- checkpoint = weakest required provider/universe/identity/classification/raw/
  build admission over complete lookbacks;
- snapshot = weakest checkpoint and subject-specific membership/identity/
  classification/peer/factor/signal admission;
- prediction = weakest snapshot and referenced signal result;
- outcome = weakest outcome source/calendar/price/action/terminal/filing/estimate/
  benchmark/raw admission;
- backtest member keeps research, outcome, benchmark, and survivorship dimensions
  separately;
- backtest result = weakest included required side, with distribution and
  limitations retained.

An industry-excess outcome may be weaker than raw return. Qualified future data
cannot improve a development prediction. A survivorship-safe universe cannot
cure an unqualified price path. Every development limitation propagates.

Every aggregate supports this immutable walk:

```text
backtest result -> run/definition -> population member
  -> prediction or valid-false SignalResult -> snapshot/checkpoint/replay run
  -> signal conditions -> FactorValue/components -> FeatureValue/source evidence
  -> universe/identity/classification/taxonomy/peer evidence -> manifests/raw

backtest result -> exact outcome rows -> outcome definition/manifest
  -> entry/exit/path/benchmark/terminal or filing/estimate evidence -> raw
```

Ordered population/outcome/path hashes protect completeness; direct anchor FKs
protect critical lineage. Hashes are integrity evidence, not a substitute for
backups or independent anchoring.

## 21. Mandatory leakage and survivorship failures

Tests must add each forbidden row/change and prove the T snapshot/prediction is
unchanged or publication fails with zero rows:

- future SEC filing/comparative fact;
- future estimate revision, correction, alias, or reporting evidence;
- future taxonomy correction/reclassification;
- future split, dividend, merger, or other corporate action in prediction data;
- future factor or signal build;
- current surviving universe, current ticker, or `Security.is_active` edit;
- ordinary revised FRED/latest macro history instead of a qualified vintage;
- same-day close from a session whose open is not strictly after decision time;
- future outcome/backtest table access from replay code;
- wrong or independently shifted benchmark interval.

No outcome can select or repair a universe member, source row, feature, factor,
signal proof, episode boundary, entry, benchmark, missing-data treatment, or
slice. Poor cases, false positives, delistings, missing outcomes, and unavailable
controls remain visible.

## 22. Scale and release boundary

The first structural qualification is exactly one year of weekly checkpoints and
1,000 securities. Measure security-date evaluations, snapshots, signal results,
predictions, outcomes, SQL statements, runtime, peak RSS, PostgreSQL temp I/O,
relation/index/WAL growth, and outcome-cache reuse. Inspect plans for N+1 access,
date/security loops, Cartesian signal/node multiplication, and unbounded lineage
loading. Process one date at a time with one publisher and bounded in-date batches.

Do not authorize a 10-year x 10,000-security run until this evidence is healthy.
Do not redesign frozen factor/signal tables or add distributed infrastructure for
benchmark vanity.

Development/synthetic replay produces only `DEVELOPMENT` evidence. No current
live provider set supports a production or survivorship-qualified historical
claim. Provider and rights gates remain hard external conditions; they never
permit weaker PIT, survivorship, entry, episode, outcome, quality, immutability,
or provenance semantics.

Phase 2.9 ends after its final internal audit as **IMPLEMENTATION COMPLETE — READY
FOR INDEPENDENT REVIEW — NOT YET FROZEN**. Phase 2.10 must not begin.
