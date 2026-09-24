# Phase 2.9 historical replay / prediction ledger / backtester architecture review

Review date: 2026-09-23

Approved starting checkpoint: `4feb3dff98da858710082d7803719ac860ce3fb5`
on `master`, equal to `origin/master` with a clean worktree before this review.
Alembic current/head/check is `p28002`. The baseline reproduced **905 passed,
8 existing warnings, zero failures**.

This is an architecture review only. It adds no application code, model,
migration, replay row, prediction, outcome, backtest result, provider data, or
Phase 2.10 behavior. Foundation and Phases 2.1, 2.2, 2.4, 2.5, 2.6, 2.7, and
2.8 remain frozen. In particular, this design consumes the existing immutable
feature, factor, signal, universe, provider-admission, and build-manifest
artifacts without changing any frozen formula, threshold, proof rule, quality
rule, or signal meaning.

## A. Verdict

**CONDITIONAL PASS**

The Phase 2.9 architecture is exact enough to implement. The conditions are
external data and release gates, not unresolved replay, ledger, episode, entry,
outcome, or aggregation semantics:

- No live cataloged provider currently has the combined qualified raw price,
  action, historical universe/security, delisting, lifecycle, survivorship, and
  licensing capabilities needed for a production historical backtest. Yahoo
  and the current `market_prices` contract remain development-only.
- The persistent development database at review time contains no published
  `FactorValue`, `SignalResult`, build-manifest, sealed historical-universe, or
  peer-group replay corpus. Existing synthetic tests prove mechanics, not a
  ready historical dataset.
- Phase 2.9 may be implemented and validated with deterministic synthetic
  providers and explicit `DEVELOPMENT` provenance. Any request for
  `PIT_QUALIFIED`, `SURVIVORSHIP_QUALIFIED`, or `PRODUCTION_RESEARCH` must fail
  closed until every required source and the complete lookback interval pass the
  existing catalog gates.
- A historical replay created today is labeled `HISTORICAL_REPLAY`; it is never
  represented as a prediction the system actually published in the past.
- Outcome data is isolated from replay inputs. It can evaluate a sealed
  prediction after T, but it cannot reconstruct, repair, select, or rerank the
  prediction at T.

Subject to those conditions, section AA recommends **SAFE TO IMPLEMENT PHASE
2.9**.

## B. Phase 2.9 scope

### Exact v1 scope

- One chronological `HistoricalReplayRunner` over a versioned weekly schedule.
- A fail-closed target research-quality gate using the repository's existing
  quality lattice and provider catalog.
- Bitemporal reconstruction of the complete screened universe, including every
  included and excluded candidate and its reason.
- Exact invocation or reuse of manifest-matching frozen feature, peer, factor,
  and signal artifacts at T.
- One compact immutable research snapshot per screened security and per
  evaluated industry-node subject. Values remain in their authoritative tables;
  the snapshot pins their exact builds and set fingerprints.
- An immutable prediction only when a signal has a proven weekly
  `VALID/fired=false -> VALID/fired=true` transition.
- The existing Phase 2.8 `SignalResult` population as the authoritative record
  of every fired, valid-false, missing, and invalid signal observation.
- Separate outcome evaluation at 3, 6, 12, and 24 calendar months, plus a small
  versioned set of fiscal and estimate labels.
- Signal-event backtests whose primary statistical unit is a signal episode,
  with contemporaneous valid-false controls and explicit coverage accounting.
- Descriptive aggregate result storage: counts, mean, median, quantiles, and
  direction rates. No automatic tuning or promotion follows from a result.
- Deterministic checkpoint/resume, per-date atomic publication, immutable
  manifests, and exact audit lineage.
- The same snapshot, prediction, outcome, and evaluation contracts for
  historical backfill and later live-forward observation.

### Explicitly out of scope

- Any change to a frozen feature, factor, signal, threshold, detector, proof,
  severity, coverage, or quality formula.
- Cross-sectional ranks, score-decile portfolios, long/short portfolios,
  portfolio weights, capital allocation, optimization, turnover, capacity,
  transaction costs, spread, slippage, fees, or strategy Sharpe/Sortino.
- Matched-control construction, causal inference, bootstrap confidence
  intervals, statistical significance, multiple-testing decisions, calibration,
  threshold fitting, ML, or champion/challenger promotion. These are Phase 2.10
  concerns.
- Sector benchmarks in v1. SPY and a separately defined PIT industry benchmark
  are sufficient for the first signal-event evaluation.
- A market-cap filter. The repository has no qualified PIT market-cap feature.
- A liquidity exclusion in v1. `dollar_vol_21` remains inspectable context, but
  an unvalidated cutoff must not redefine the historical universe.
- Qualitative AI thesis, catalyst, invalidation, news, regime, or retrospective
  LLM labels. The schema retains extension points, but no such label is admitted.
- Event-driven replay in v1. The schedule interface supports it later without
  changing snapshot or ledger identity.
- Phase 2.10 calibration or any automatic model self-modification.

### Exact boundary

```text
qualified as-of data and sealed universe at T
    -> frozen feature / peer / factor / signal runners at T
    -> complete immutable research snapshot
    -> episode-start prediction ledger

separately, after publication:
qualified future outcome build
    -> immutable continuous outcome labels
    -> descriptive signal-event backtest
    -> Phase 2.10 may later study calibration or challengers
```

The replay transaction never requires future data. Historical backfill may run
the outcome worker immediately afterward because those dates have already
matured, but it still uses a new service boundary, session, manifest, and query
path.

## C. Historical replay model

### C.1 Precise terms

| Term | Phase 2.9 meaning |
| --- | --- |
| **Research timestamp T** | The timezone-aware logical information cutoff for one research decision. An input is admissible only under its versioned policy and justified `available_at <= T`. T is not the later wall-clock time at which a historical reconstruction runs. |
| **Replay run** | One immutable experiment request plus operational checkpoints that reconstructs one ordered sequence of T values from pinned policies and builds. It may be created years later and is explicitly originated as `HISTORICAL_REPLAY`. |
| **Signal observation** | One existing immutable Phase 2.8 `SignalResult`: `VALID/true`, `VALID/false`, `MISSING`, or `INVALID`, with exact conditions and factor lineage. Every requested state is an observation; not every observation is a prediction. |
| **Research snapshot** | One sealed subject-at-T index over the exact universe evidence, identity/classification context, build manifests, factor values, signal results, warnings, quality, and set fingerprints. It references immutable artifacts instead of copying their bytes or formulas. |
| **Prediction** | One immutable, versioned, ex-ante-style assertion emitted by an episode policy from an exact fired signal observation. Historical predictions are reconstructions, not claims of actual past issuance. |
| **Outcome horizon** | A versioned rule mapping an actual entry session to a future calendar boundary and then to an eligible exit observation. It is not a vague label such as “about one quarter.” |
| **Outcome label** | One continuous or categorical value produced only by the isolated outcome evaluator under an immutable label definition and outcome build. Status and missingness are part of the label. |
| **Benchmark** | A versioned comparison wealth path over exactly the security's actual entry/exit interval, with PIT membership, action, terminal, currency, and quality rules. |
| **Evaluation** | Joining a frozen prediction or control observation to mature outcome labels and computing descriptive evidence under a preregistered definition. Evaluation cannot alter the prediction. |
| **Backtest** | A versioned historical experiment over published replay snapshots, predictions, controls, and outcome builds. V1 is a signal-event backtest, not a portfolio simulation. |
| **Calibration** | Mapping a score/bucket to empirical outcome frequencies for one named label and horizon using separate chronological data. Calibration is not implemented in Phase 2.9. |

The generic word “backtest” must not describe a feature regression, famous case,
synthetic runner test, prediction replay alone, or portfolio strategy that was
not actually constructed.

### C.2 Repository boundary actually available

The design depends on these existing contracts rather than imagined APIs:

- `ResearchBuildManifest.create(...)` requires the exact eleven semantic fields,
  includes a timezone-aware `research_timestamp`, and hashes canonical JSON to a
  64-character `build_id`. `research_build_manifests` already persists that
  canonical payload immutably.
- `HistoricalUniverseResolver` first selects the latest evidence version known by
  T and only then tests the effective interval `[start_date, end_date)`. Current
  `Security.is_active` and current ticker/classification are not historical
  selectors.
- Phase 2.6 sealed datasets separately pin classification, eligibility, and
  security-identity evidence. The canonical peer eligibility policy is
  `peer_primary_common_equity_v1`, one primary security per company, with no
  hierarchy fallback.
- `FactorValue` is identified by exact definition, subject, T, and build and
  stores value/status, coverage, quality, admission, provenance, and component
  lineage.
- `SignalRunner.calculate_signals(subjects, research_timestamp,
  research_build_manifest, signal_registry_keys)` reads only exact immutable
  `FactorValue` rows. `SignalResult` is identified by signal definition, subject,
  T, and signal build; it stores all four relevant states through
  `VALID/true`, `VALID/false`, `MISSING`, and `INVALID` plus every condition.
- Phase 2.8 already rejects future, other-T, other-build, other-subject, and
  present-day industry fallbacks. Phase 2.9 must not weaken those checks.
- The quality order is exactly `DEVELOPMENT < PIT_PARTIAL < PIT_QUALIFIED <
  SURVIVORSHIP_QUALIFIED < PRODUCTION_RESEARCH`.
- The current `MarketPrice` identity omits provider/build/version and its raw
  OHLCV basis is unqualified. It may support explicit development fixtures, not
  production outcome claims.

### C.3 Runner contract

Conceptual API:

```text
HistoricalReplayRunner.run(
    replay_definition,
    start_date,
    end_date,
    schedule_policy="weekly_friday_2000_new_york_v1",
    target_quality,
    universe_policy,
    provider_qualifications,
    source_dataset_builds,
    feature_versions,
    factor_registry_keys,
    signal_registry_keys,
    development_override=None,
) -> ResearchReplayRun
```

For each scheduled T, in chronological order, the runner must:

1. Rebuild and independently hash every manifest. Pin Git commit, Alembic
   revision, dependency versions, provider/catalog versions, raw inventory,
   source datasets, normalization/feature/calculation versions, and every policy.
2. Calculate each source family's complete required coverage interval, including
   the longest feature lookback. Call `require_historical_replay` for every
   pinned provider/version over that interval, not merely at T.
3. Enforce the requested quality. Any unmet capability or coverage interval
   fails the checkpoint; a stronger request is never downgraded.
4. Resolve the bitemporal candidate universe and all eligibility exclusions from
   the sealed classification, eligibility, and identity datasets.
5. Reuse an upstream artifact only when T, definition/version/hash, subject,
   effective date, dataset/build, origin, admission, and manifest match exactly.
   Otherwise invoke the frozen runner through its supported entry point. Never
   copy a current value backward or choose a generic “latest” artifact.
6. Publish every requested Phase 2.8 signal state and its condition graph.
7. Seal compact research snapshots, derive transitions against the immediately
   preceding completed schedule checkpoint, and append any episode predictions.
8. Atomically mark the date checkpoint complete with counts and a canonical
   publication fingerprint. A failure rolls back that date and stops the run.

The first date in the requested evaluation range has one prior scheduled
**warm-up checkpoint**. Its artifacts may establish transition state, but its
predictions/outcomes are excluded from the requested result window. This prevents
an arbitrary start-date boundary from turning an already-active signal into a
new episode.

### C.4 Research-quality gate

The request names one exact existing `ResearchQualityLevel`.

| Target | V1 admission rule |
| --- | --- |
| `DEVELOPMENT` | Allowed only with repository-owned synthetic providers or a structured `DevelopmentOverride(requested_by, reason, reference)`. The override and every limitation are persisted. |
| `PIT_PARTIAL` | Not a valid target for historical performance claims. It remains a provenance state for current/prospective research but cannot masquerade as historical replay. |
| `PIT_QUALIFIED` | Every source must pass `HISTORICAL_REPLAY`, trust, version, evidence, and full-coverage checks. |
| `SURVIVORSHIP_QUALIFIED` | In addition, require historical universe/security coverage, PIT actions, delisted history, lifecycle history, and `SURVIVORSHIP_SAFE`. |
| `PRODUCTION_RESEARCH` | Require both the production/raw/licensing gate and all survivorship-safe capabilities. Passing only one is insufficient. |

A development override is valid only when the requested target itself is
`DEVELOPMENT`. It cannot waive a `PIT_QUALIFIED`, survivorship, or production
claim. The run stores requested quality and realized weakest quality separately.

### C.5 Historical universe

At T, the universe is the intersection of exact, sealed evidence:

```text
latest membership version known by T
AND economically effective on the schedule's effective session
AND eligibility universe = peer_primary_common_equity_v1
AND one unambiguous primary security per company
AND effective SecurityIdentityHistory says ACTIVE
AND exact historical classification/taxonomy release is usable
AND required provider coverage is admitted
```

Security type, ticker, exchange, lifecycle, sector, industry, and sub-industry
come from the historical evidence rows and release, not mutable current master
fields. Delisted, acquired, merged, and bankrupt securities remain candidates at
dates when their effective history says they were eligible. Every candidate gets
an included or excluded snapshot row and a stable reason. An empty or ambiguous
universe fails the checkpoint rather than silently falling back to today's list.

No liquidity or market-cap screen is added in v1. If a future liquidity policy is
introduced, it must be a new version using an exact PIT feature/build and it must
preserve pre-filter counts and exclusions.

### C.6 Replay is not recalculation with today's knowledge

These operations are prohibited:

- downloading current SEC Company Facts and assigning its old period rows to an
  earlier T;
- treating a 2026 Yahoo download of a 2021 close or adjusted close as evidence
  admitted to a 2021 feature build;
- applying a taxonomy correction before its own `available_at`;
- building the universe from current `Security.is_active`, current ticker, or
  current surviving companies;
- selecting a future estimate correction, alias, reporting event, macro vintage,
  factor value, signal build, or outcome because its economic observation date is
  old;
- precomputing with the full future panel and slicing rows labeled T afterward.

A qualified historical archive retrieved later may support an explicitly
reconstructed provider/build only when its publication, revision, identity, and
coverage semantics are independently certified. It still does not become an
actual “seen by this system in 2021” receipt. Existing frozen runners that require
receipt by T may only use artifacts satisfying their actual contract; Phase 2.9
cannot bypass that check.

## D. Prediction ledger

### D.1 Exact record policy

`research_predictions` contains **episode-start predictions only**. It does not
duplicate every Phase 2.8 observation. A record is emitted only from an exact
fired `SignalResult` that satisfies section E's transition rule.

The broader evaluated population is still complete:

- every eligible subject-at-T has a sealed research snapshot;
- every requested signal has its existing `SignalResult` and condition rows;
- valid false is a control candidate;
- missing and invalid remain explicit coverage failures;
- excluded securities remain in the checkpoint snapshot population with reasons.

This avoids selection bias without adding a second copy of tens of millions of
signal-state rows.

### D.2 Prediction fields

Each immutable prediction must contain or reference:

- `prediction_id` and a canonical content fingerprint;
- `research_snapshot_id`, `signal_result_id`, and
  `prior_signal_result_id` (the exact valid-false predecessor);
- replay run/checkpoint for a reconstruction, or live research run for a forward
  observation;
- subject scope and exact permanent security or industry-node/group identity;
- research timestamp, effective date, and prediction origin;
- exact `ResearchSignal` ID/version/hash and detector/policy versions through the
  signal result;
- exact signal build, upstream factor build, and full build-manifest chain;
- `prediction_type`, `objective_family`, and prediction-policy version;
- the canonical expected horizon/label set definition;
- snapshotted `VALID/true`, evidence strength, severity, coverage, and signal
  quality, with database validation against the referenced signal result;
- prediction quality as the weakest replay/snapshot/signal quality;
- created/published time distinct from historical T.

The snapshot makes all co-occurring opportunity, context, contradiction, and
warning results at T queryable. The prediction row does not copy their factor
graphs.

Origins are explicit: `HISTORICAL_REPLAY`, `LIVE_SHADOW`, `LIVE_CHAMPION`, or
`EDUCATIONAL`. V1 historical implementation uses `HISTORICAL_REPLAY` or
`EDUCATIONAL`; it does not create a false live origin.

### D.3 Research snapshot

One compact snapshot row per checkpoint subject stores:

- security or industry-node/group identity and ticker/classification as known at
  T;
- `ELIGIBLE`, `EXCLUDED`, or `EVALUATED_NODE` state and exclusion reason;
- exact membership, eligibility, identity, classification, taxonomy-release, and
  peer-group evidence references where applicable;
- T, effective session/date, origin, replay checkpoint, signal build, upstream
  factor build, and canonical manifest IDs;
- exact `decision_ready_at`; all predictions published by one snapshot share this
  anchor, and adding a later live prediction requires a new snapshot/run rather
  than changing the old anchor;
- requested and realized quality plus limitations;
- expected/actual factor and signal counts;
- ordered factor-value-ID and signal-result-ID set hashes, warnings/counts, and a
  complete snapshot fingerprint;
- creation time, which may be years after T for a reconstruction.

Factor values, signal results, conditions, admissions, and component evidence
remain in their authoritative immutable tables. A snapshot does not duplicate
raw source bytes, formulas, factor graphs, or every JSON provenance document.
Exact builds plus ordered relational set hashes make omission or substitution
detectable.

Published snapshots and predictions reject `UPDATE` and `DELETE`. An error or
new model version creates a new run/build and optional explicit supersession
reference; it never repairs old history in place.

## E. Signal episode policy

The v1 policy is `valid_false_to_true_weekly_v1`:

1. Compare the same signal definition/version and same permanent subject across
   two immediately consecutive **completed** schedule checkpoints.
2. Emit one prediction when the predecessor is exactly `VALID/fired=false` and
   the current observation is exactly `VALID/fired=true`.
3. Repeated `VALID/true` observations are episode continuation and create no new
   prediction, even if severity or evidence strength changes materially.
4. The first observed true state, a true state after the subject enters the
   universe, or a true state after `MISSING`, `INVALID`, an incomplete checkpoint,
   or a schedule gap is left-censored. It remains visible as a signal observation
   but is excluded from the primary episode-start prediction population.
5. The first subsequent `VALID/false` closes the observable episode. End state is
   derived from immutable signal observations; v1 does not mutate the prediction
   or need a separate `signal_episodes` table.
6. A later immediately consecutive false-to-true transition begins a new episode.
   There is no cooldown, hysteresis, minimum-off duration, or tunable reset.
7. The same rule applies to security and industry-node signals.

Thus the prediction itself is the immutable episode identity. A future need for
typed `FIRED_CONTINUE` or `FIRED_END` events can add an append-only episode-event
table without changing v1 predictions. Strength-based episode restarts are
deferred because they would introduce an unvalidated threshold.

## F. Replay schedule

V1 cadence is **weekly** under
`weekly_friday_2000_new_york_v1`:

- T is each Friday at `20:00:00 America/New_York`, converted and stored as UTC.
- The market effective date is the latest qualified regular US equity session on
  or before that local Friday.
- The requested range includes T values whose local calendar date is within the
  inclusive run bounds, plus one prior state-only warm-up checkpoint.
- The schedule and calendar versions are manifest pins. DST, holidays, unexpected
  closures, and session clocks are resolved by that version; naive UTC constants
  are prohibited.

Weekly sampling keeps a ten-year 5,000-security candidate near 2.6 million
security-date snapshots while exercising temporal state, estimate ageing, and
signal transitions. Daily replay would multiply the already-wide factor/signal
lineage by roughly five without first adding evidence proportional to its cost.
Month-end would be cheaper but too coarse for short-lived revision/market
transitions. Quarterly replay is inconsistent with the detector's market and
estimate horizons.

`ReplaySchedulePolicy` must yield ordered `(research_timestamp, schedule_key,
context)` values. Later event-driven policies may yield earnings releases, major
estimate revisions, or signal-transition candidates, but they use the same
quality, snapshot, prediction, entry, and outcome contracts. Event-driven replay
is not implemented in v1 and cannot retroactively change weekly episodes.

## G. Execution / entry policy

V1 uses `next_regular_session_close_v1`. It is an evaluation convention, not an
assertion that a trade was placed.

### G.1 Timestamps

- `research_timestamp`: evidence cutoff T.
- `prediction_published_at`: actual immutable-ledger publication time.
- `decision_ready_at`: T for `HISTORICAL_REPLAY`/`EDUCATIONAL`; for live origins,
  the later of T and actual publication time.
- `eligible_execution_session`: the first qualified regular session whose
  official open is **strictly after** `decision_ready_at`.
- `entry_timestamp`: that session's official close.
- `entry_price`: the exact qualified raw-as-traded close at that timestamp,
  linked to its observation/build. Provider adjusted close is prohibited.

Choosing the close of a session that begins after the decision creates a full,
auditable no-same-bar lag and uses the daily data surface the project actually
has. It avoids pretending that a research signal achieved an exact opening fill.

### G.2 Boundary examples

| Signal availability | Eligible v1 entry |
| --- | --- |
| May 20, 16:05 New York, after the close | Close of the next regular session. May 20 close is forbidden. |
| May 20, 07:00 New York, before the open | May 20 close, because its regular open is strictly after the decision. |
| Exactly at the regular open | The following regular session's close; strict `>` removes boundary ambiguity. |
| Saturday | Monday/next regular session close. |
| Exchange holiday | Close of the next qualified regular session. |

If the intended entry session has no exact valid price, the evaluator checks at
most that session plus the next two scheduled regular sessions. It selects the
first exact valid close and records the delay. It never carries a stale last price
or searches indefinitely for a convenient entry. No valid observation within
three sessions yields `MATURE_MISSING/ENTRY_PRICE_UNAVAILABLE` for all price-path
labels.

## H. Outcome horizons

V1 canonical market horizons are exactly **3m, 6m, 12m, and 24m** under
`calendar_month_from_entry_v1`.

1. Start from the actual entry session's New York local date.
2. Add 3, 6, 12, or 24 calendar months using end-of-month clamping: for example,
   November 30 plus three months is the last valid day of February.
3. Select the first qualified regular-session close on or after that boundary.
4. As at entry, search no more than three scheduled sessions for an exact valid
   security observation. Record the actual exit and delay.

The horizon is therefore not 63/126/252/504 trading sessions. Actual entry and
exit timestamps are always stored. Fundamental and estimate labels have their
own fiscal or calendar-day maturity rules in section J; a mature 12m return does
not imply a future filing label has matured.

## I. Market outcome labels

All v1 market labels are continuous. No result is collapsed into a universal
success flag.

### I.1 Versioned definitions

| Label definition | Exact v1 meaning |
| --- | --- |
| `security_total_return_v1` | Simple total return over the actual entry/exit interval, including qualified ordinary distributions and terminal proceeds. |
| `security_price_return_v1` | Split-continuous price-only simple return over the same interval; a diagnostic, not a substitute for total return. |
| `spy_total_return_v1` | SPY simple total return over exactly the security's actual entry/exit sessions. |
| `spy_excess_return_v1` | `security_total_return_v1 - spy_total_return_v1`. Simple returns are subtracted; log returns are not mislabeled as excess simple return. |
| `pit_industry_total_return_v1` | Investable-style daily-rebalanced equal-weight simple-return wealth for the prediction-time industry node under the PIT membership/action policy in section N. |
| `pit_industry_excess_return_v1` | Security total return minus the aligned PIT industry total return. |
| `maximum_drawdown_from_entry_v1` | Worst decline from the running peak of the subject's total-return wealth path, including qualified terminal losses. |
| `maximum_favorable_excursion_from_entry_v1` | Greatest total-return gain relative to entry wealth reached before the exit. It is not return from a later trough. |
| `forward_realized_volatility_v1` | Sample standard deviation of daily log changes in positive total-return wealth, annualized by `sqrt(252)`, with at least 20 valid intervals. |

For an industry-node prediction, the subject path is
`pit_industry_total_return_v1`; its excess label is that path minus aligned SPY,
and drawdown/MFE/volatility operate on the industry wealth path. A node result is
not copied to every constituent as a company prediction.

### I.2 Total-return path

Use exact finite Decimal arithmetic under a pinned policy. Let `P'_s` be the
split-continuous raw close and `D_s` qualified distributions for session `s`.
Entry wealth is one at the entry close. For later sessions:

```text
W_entry = 1
W_s = W_(s-1) * (P'_s + D_s) / P'_(s-1)
TotalReturn = W_exit - 1
PriceReturn = P'_exit / P'_entry - 1
```

Only distributions economically occurring after the entry close and through the
exit close enter the holding path. Split adjustment preserves units; a split is
not a return. The evaluator consumes a qualified versioned raw-price/action build
and never uses provider adjusted close as an input.

For an industry basket, set equal weights at each prior qualified session close
using only memberships, classifications, identities, and lifecycle evidence known
at that close. Apply those weights to each constituent's next close-to-close
qualified simple total return, including terminal outcomes, and then rebalance at
the new close. Evidence first learned at the new close can affect only the next
interval. For the first interval after entry, use the members known at the entry
close. Compound:

```text
r_industry,s = sum(r_i,s) / N_s
W_industry,s = W_industry,s-1 * (1 + r_industry,s)
```

Require at least three valid constituents each session. This is deliberately
distinct from Phase 2.4's descriptive mean-log-return research benchmark and is
versioned separately.

### I.3 Path risk

Include `W_entry = 1` in both paths:

```text
running_peak_s = max(W_entry ... W_s)
drawdown_s     = W_s / running_peak_s - 1
MaxDrawdown    = min(drawdown_s)              # in [-1, 0]
MFE            = max(W_s / W_entry - 1)       # >= 0
```

Forward realized volatility uses consecutive positive wealth observations:

```text
g_s = ln(W_s / W_(s-1))
ForwardVol = sample_stddev(g_s) * sqrt(252)
```

A qualified terminal conversion to cash extends the path with zero nominal cash
return until the horizon. Transaction costs, financing, interest on terminal
cash, taxes, spread, slippage, and capacity are excluded. Results therefore
describe event outcomes, not realizable portfolio alpha.

## J. Fundamental outcome labels

V1 has a deliberately small first-reported set. It does not attempt every future
fundamental or thesis label.

### J.1 Delivered fundamentals

At T, freeze the latest eligible explicit quarterly fiscal identity as the
baseline and its evidenced next-quarter successor as the target. Calendar dates
never infer fiscal identity. The target label uses the first qualified primary
reporting version available after T; a later restatement may be a new label
definition/build but cannot replace it.

| Label definition | Value and unit |
| --- | --- |
| `next_q_revenue_yoy_growth_first_reported_v1` | Frozen `revenue_yoy_growth` semantics for the explicit next fiscal quarter; ratio. |
| `next_q_revenue_growth_acceleration_first_reported_v1` | Frozen `revenue_growth_acceleration` semantics for that quarter; percentage points. |
| `next_q_operating_margin_change_from_baseline_first_reported_v1` | First-reported target-quarter operating margin minus the exact baseline-quarter operating margin known at T; percentage points. |
| `next_q_fcf_margin_change_from_baseline_first_reported_v1` | First-reported target-quarter FCF margin minus the exact baseline-quarter FCF margin known at T; percentage points. Missing CapEx remains missing. |

The label points to the exact target fiscal period, baseline/target feature or
fact evidence, first-reporting event, formula/version, and source build. If the
fiscal successor is unresolved at T, the prediction can still exist but this
label is `MATURE_MISSING/FISCAL_TARGET_UNRESOLVED`; the evaluator may not choose a
convenient future quarter after seeing results.

### J.2 Estimate continuation

Freeze at T the exact FQ1 absolute target, provider/profile, metric, adjusted or
reported basis, diluted/share basis, currency, scope, and consensus series used
by Phase 2.5. Compare that same series at T with the latest consensus state whose
`state_available_at <= T + 30 days` or `T + 90 days`. The selected state itself
must be `VALID`; a latest `MISSING`/`INVALID` state remains unavailable and the
evaluator may not resurrect an older valid state. The existing no-arbitrary-age-
expiry rule remains in force, while complete source coverage through the boundary
is mandatory:

| Label definition | Formula |
| --- | --- |
| `same_target_eps_consensus_scaled_change_30d_v1` | `(mean_T+30 - mean_T) / max(abs(mean_T), abs(mean_T+30))`; both zero maps to zero. |
| `same_target_eps_consensus_scaled_change_90d_v1` | Same formula at T+90 calendar days. |
| `same_target_revenue_consensus_pct_change_30d_v1` | `(mean_T+30 - mean_T) / mean_T`, requiring a positive baseline and nonnegative future revenue. |
| `same_target_revenue_consensus_pct_change_90d_v1` | Same formula at T+90 calendar days. |

These are consensus-change labels, not necessarily analyst economic activity;
the outcome preserves membership, corrections, withdrawals, and activity
evidence separately. If the target is first reported before the boundary, the
fixed-window label is `CENSORED/TARGET_REPORTED_BEFORE_ESTIMATE_HORIZON`, not
silently rolled to the new FQ1 and not compared over a shorter window. A future
pre-report label can be added with a new definition.

No v1 qualitative catalyst/invalidation, future EPS growth, annual FCF, or
latest-restated label is implied. Continuous values are preserved; favorable or
unfavorable thresholds may only be added as separately versioned derived labels.

## K. Signal-to-outcome mapping

The mapping is code-owned, versioned as `phase_2_8_signal_objectives_v1`, and
pinned by every prediction/backtest definition. “Primary” means the endpoint the
signal is intended to illuminate; secondary values are diagnostics and may not
replace a missing primary endpoint after results are seen.

| Frozen signal | Objective family | Primary v1 outcomes | Secondary v1 diagnostics |
| --- | --- | --- | --- |
| `fundamental_acceleration_v1` | `GROWTH_DELIVERY` | Next-quarter revenue YoY growth and revenue-growth acceleration | 6m/12m/24m SPY and industry excess return; drawdown |
| `margin_inflection_v1` | `MARGIN_DELIVERY` | Next-quarter operating-margin change from baseline | Next-quarter FCF-margin change; 6m/12m excess return and drawdown |
| `estimate_confirmation_v1` | `EXPECTATION_PERSISTENCE` | Same-target EPS and revenue consensus changes at 30d and 90d | 3m/6m excess return; target-report censoring rate |
| `market_confirmation_v1` | `MARKET_CONTINUATION` | 3m/6m/12m total, SPY-excess, and industry-excess return | MFE, drawdown, and forward volatility; 24m decay diagnostic |
| `industry_tailwind_v1` | `INDUSTRY_STRENGTH` | Node industry total and SPY-excess return at 3m/6m/12m | Node drawdown, MFE, volatility, and 24m decay |
| `industry_headwind_v1` | `INDUSTRY_WEAKNESS` | Node SPY-excess return, drawdown, and volatility at 3m/6m/12m | Node total return, MFE, and 24m decay; no binary “headwind succeeded” flag |
| `cash_flow_contradiction_v1` | `CASH_FLOW_RISK` | Next-quarter FCF-margin change; 3m/6m/12m drawdown and SPY-excess return | Revenue growth/acceleration, MFE, and forward volatility |
| `fragility_warning_v1` | `DOWNSIDE_RISK` | 3m/6m/12m maximum drawdown and forward realized volatility | Total/excess return and MFE, which quantify false alarms or missed upside; 24m diagnostic |
| `early_growth_candidate_v1` | `EARLY_GROWTH` | Next-quarter revenue growth/acceleration and 6m/12m/24m SPY/industry excess return | 30d/90d estimate continuation, 3m return, margin/FCF change, MFE, drawdown, and volatility |

All common market horizons may be materialized for decay analysis, but the table
predeclares interpretation. A warning is not judged by the same criterion as an
opportunity, and business delivery is not equated with stock performance.

## L. Outcome maturity

Outcome rows are append-only observations with exactly one status:

| Status | Exact meaning |
| --- | --- |
| `PENDING` | The label's scheduled boundary or required publication has not been reached in the outcome build. Numeric value is null. |
| `MATURE_VALID` | The boundary is reached and all definition-required qualified evidence is complete. |
| `MATURE_MISSING` | The boundary is reached, but required evidence is absent, stale, outside the fixed search window, or otherwise unavailable. Numeric value is null and reason is mandatory. |
| `CENSORED` | A defined economic event prevents ordinary observation of the full label, such as unresolved terminal value or target reporting before a fixed estimate horizon. It is not ordinary missingness. |
| `DISPUTED` | Qualified sources or corrections conflict and the policy cannot choose one result. |

Each row stores scheduled maturity, actual entry/exit or fiscal target, outcome
dataset cutoff, `evaluated_at`, `matured_at` when applicable, value/unit, reason,
quality, evidence/build references, fingerprint, and optional superseded outcome
ID. `PENDING` has null `matured_at`; every other status has the instant at which
its terminal evidence decision became available. Pending is never zero and is
excluded from mature-value denominators.

An unchanged evaluation against the same outcome build is idempotent. New data,
provider corrections, or a changed label/entry/benchmark policy requires a new
outcome build or definition and appends a row. The earlier pending, mature, or
disputed row remains queryable.

Evaluation completeness is always reported as:

```text
eligible population
  = mature valid + mature missing + censored + disputed + pending
```

Near the end of a dataset, 12m/24m pending observations are not failures and do
not enter a hit-rate denominator.

## M. Delisting / censoring policy

`terminal_total_return_v1` distinguishes economic termination from missing data:

- A qualified cash acquisition, liquidation distribution, bankruptcy recovery,
  or other terminal proceeds becomes part of total-return wealth on its effective
  session. Wealth is then held as nominal cash at 0% through the horizon.
- A qualified stock conversion is valued from the exact conversion ratio and
  successor observation on the effective session, then notionally liquidated to
  cash for v1. V1 does not silently stitch successor performance beyond that
  point.
- A legally evidenced cancellation with zero recovery produces terminal wealth
  zero and return/drawdown of `-1`.
- A delisting, suspension, or disappearance without qualified proceeds is
  `CENSORED/TERMINAL_VALUE_UNKNOWN`. The last quote is not carried forward and
  missing data is not assumed to be a 100% loss.
- A still-listed security with no valid exit inside the fixed three-session
  window is `MATURE_MISSING/EXIT_PRICE_UNAVAILABLE`; a lifecycle termination with
  unresolved value is censored instead.
- Sensitivity bounds may be stored only when evidence supports them. They never
  replace the null primary value.

Censored and missing rows stay in coverage counts. A run with unresolved terminal
cases cannot claim survivorship-qualified results unless its qualification and
predeclared tolerance policy explicitly permit and disclose them; v1's default is
fail closed for the survivorship claim.

## N. Benchmark policy

V1 provides exactly two benchmark families:

1. `SPY_total_return_aligned_v1`: qualified SPY raw close/action history over the
   security's actual entry and exit sessions.
2. `pit_dynamic_industry_equal_weight_v1`: the prediction-time stable industry
   node interpreted through a pinned taxonomy-lineage policy. For each future
   close-to-close interval, equal-weight the primary securities whose membership/
   classification is effective and known at the **start** close, with qualified
   delisting and terminal treatment during the interval. Rebalance only after the
   interval. Require three valid constituents at each weight-setting close.

The security's prediction-time taxonomy/node identity is frozen; today's sector
or industry never substitutes for it. A future taxonomy release needs explicit
stable-node lineage. No lineage, membership, or terminal coverage means the
industry label is missing or lower quality, not replaced with SPY.

Benchmark start and end observations align to the security's **actual** delayed
entry and exit sessions. Currency, calendar, action, and terminal conventions are
the same. Benchmark quality is separate and propagates to the excess label.

Sector ETF excess return is deferred, not open: it adds a coarser overlapping
benchmark and another qualified-history requirement without resolving the first
research question. It can be introduced as a new benchmark definition later.

## O. Backtest definition

A replay reconstructs historical decisions. A backtest evaluates a frozen
population selected from published replay artifacts. They are separate objects.

An immutable `backtest_definition` pins at least:

- definition ID/version/hash and description;
- exact replay run and required completeness;
- date range and temporal partition (`DEVELOPMENT`, `VALIDATION`, or `HOLDOUT`);
- signal IDs/versions/hashes and `phase_2_8_signal_objectives_v1`;
- `valid_false_to_true_weekly_v1` episode policy and warm-up rule;
- universe/schedule/quality/origin policies;
- prediction and control selection rules;
- entry, horizon, label, terminal, benchmark, and aggregation versions;
- exact outcome build and required label set;
- requested/realized quality filters and coverage treatment;
- all slice dimensions and minimum-reporting rules;
- Git commit, schema revision, dependencies, and canonical fingerprint.

Changing schedule, universe, quality gate, signal version, transition rule,
entry, horizon, label, benchmark, terminal treatment, filter, partition, or
aggregation creates a new definition. It does not revise a completed backtest.

V1's statistical unit is one **signal episode start**. It produces conditional
outcome distributions, not positions or a capital-weighted return stream.
Cross-sectional ranks and portfolios are explicitly deferred until Phase 2.10
can use calibrated evidence and an independent portfolio contract.

Partition boundaries are stored now, but actual years are not frozen until a
coverage audit is complete and before outcomes are inspected. The database can
enforce that a `HOLDOUT` definition has no parent tuning run; formal promotion and
holdout-spending policy remains Phase 2.10.

## P. Evaluation metrics

For every signal, outcome definition, horizon/target, cohort, and permitted slice,
v1 stores:

- `eligible_count`;
- `mature_valid_count`, `mature_missing_count`, `censored_count`,
  `disputed_count`, and `pending_count`;
- arithmetic mean of mature valid continuous values;
- `P10`, `P25`, median, `P75`, and `P90` using one pinned percentile policy;
- positive-value rate where the outcome definition declares zero meaningful;
- for raw return, win rate `P(return > 0)`;
- for excess return, positive excess-return rate `P(excess > 0)`;
- coverage/maturity rate with its numerator and denominator;
- descriptive case-minus-control differences in mean, median, and positive rate.

Drawdown and MFE are summarized by the same distribution fields; signs retain
their definitions. Means are shown but never alone. Counts always name the unit
(`episode`, `control snapshot`, `security`, or `research date`).

V1 does **not** persist p-values, “statistically significant” flags, bootstrap
confidence intervals, multiple-testing-adjusted results, Sharpe/Sortino, alpha,
information coefficient, or rank-decile spreads. Confidence intervals and
dependence-aware date/company block resampling belong to Phase 2.10 after the
episode/control population is stable. Raw trials and endpoints remain registered
so unsuccessful tests cannot disappear.

### P.1 Initial slices

Persist overall results and single-dimension slices only:

- signal;
- horizon/label;
- fired severity (`LOW`, `MEDIUM`, `HIGH`), not a new strength bucket;
- prediction/research quality;
- historical sector at T.

Exact industry/sub-industry, liquidity, regime, and co-occurring-warning fields
remain on the immutable population and can be queried, but v1 does not precompute
their potentially sparse Cartesian combinations. No multi-slice combination is
created unless a later definition preregisters it.

## Q. Control population

The authoritative control source is Phase 2.8 `SignalResult`, not a duplicate
prediction row.

For each episode-start prediction at T:

- cases are the one prediction episode for the exact signal/version;
- controls are all same-scope, same-signal/version observations in the same
  completed checkpoint that are `VALID/fired=false`, pass the same research
  quality gate, and belong to eligible snapshots;
- `MISSING` and `INVALID` are never negatives. They are separately counted as
  unavailable coverage;
- fired continuations are not new cases or controls;
- excluded securities/nodes remain in universe coverage but not the valid control
  distribution;
- control outcomes use the same entry, horizon, label, benchmark, and outcome
  build as cases.

Within one backtest, controls are a set union keyed by exact `SignalResult`: the
same valid-false signal observation appears at most once for its signal/T even if
several securities start episodes that date. This prevents dates with more cases
from multiplying an identical control panel.

This full contemporaneous comparison preserves base rates and market-date
conditions without outcome-based sampling. A security can appear as a control on
more than one episode date, so counts are not independent experiments; Phase 2.9
states that limitation and Phase 2.10 handles dependence-aware inference.

Matched controls by date, industry, market cap, and liquidity are **deferred**.
The PIT classification and `dollar_vol_21` lineage needed for later matching are
preserved, but there is no robust PIT market cap and no justified distance or
caliper policy today. V1 never labels its unadjusted controls “causal matches.”

## R. Storage model

The smallest normalized v1 model is eleven new logical tables. Exact DDL belongs to
the later implementation specification; this review creates no migration.

### R.1 Replay and snapshots

1. `research_replay_runs`
   - One canonical replay definition/attempt: range, schedule, target quality,
     origin, policy versions, source/build template, status, fingerprint, and
     timestamps.
   - Status follows the controlled FSM in section T; semantic configuration is
     immutable from creation.

2. `research_replay_checkpoints`
   - One scheduled T per run, including warm-up: ordinal, T, effective date,
     status, exact universe/factor/signal build IDs, attempt/error metadata,
     included/excluded/evaluated/result/prediction counts, and publication hash.
   - Unique `(replay_run_id, ordinal)` and `(replay_run_id, research_timestamp)`.

3. `research_snapshots`
   - One compact subject-at-T row with the fields in section D.3.
   - Security rows represent every candidate, including exclusions; node rows
     represent evaluated formal nodes.
   - Unique checkpoint/subject identity and indexes on
     `(security_id, research_timestamp)`, `(industry_node_id,
     research_timestamp)`, checkpoint/status, and exact builds.

4. `research_predictions`
   - One episode-start row with exact current/prior signal result and snapshot
     FKs, objective/policy metadata, inherited strength/severity/quality, origin,
     fingerprint, and created time.
   - Unique current `signal_result_id`; unique semantic episode fingerprint; index
     by signal, subject, T, quality, and maturity-selection fields.
   - No dedicated v1 `signal_episodes` table.

### R.2 Outcomes

5. `outcome_definitions`
   - Immutable code-owned label ID/version, subject scope, value/unit/domain,
     direction metadata, maturity rule, entry/horizon/terminal/benchmark formula
     versions, canonical payload, and definition hash.

6. `research_outcomes`
   - One scalar label observation per `research_snapshot_id`, outcome definition,
     exact `decision_ready_at`, horizon or fiscal target, and outcome
     `ResearchBuildManifest` ID. Snapshot identity plus the decision anchor is
     primary so the same security-date outcome is reused by several signal cases/
     controls only when its executable window is identical.
   - Stores status/reason, scheduled/actual window, entry/exit price observation
     references, benchmark subject/evidence references, numeric/text value,
     unit, maturity/evaluation time, outcome quality/admission, compact evidence
     provenance, path/evidence hash, supersession, and fingerprint.
   - Index latest-status/maturity lookup by `(status, scheduled_maturity_at,
     outcome_definition_id)` and exact snapshot/definition/horizon/build lookup.

7. `prediction_outcome_links`
   - Immutable many-to-many audit link from each prediction to every applicable
     shared outcome row, including pending and superseding builds.
   - Unique `(prediction_id, research_outcome_id)` with database checks that
     snapshot, decision anchor, definition/horizon set, and subject agree. This
     preserves direct `prediction_id -> outcome` traversal without duplicating
     price paths or scalar labels for co-occurring signals.

Outcome builds reuse the existing content-addressed
`research_build_manifests`; no second generic build table is invented. A manifest
has a versioned `configuration.outcome` payload that pins price/action/universe,
calendar, entry, horizon, label, benchmark, terminal, and evaluator versions.

### R.3 Backtests

8. `backtest_definitions`
   - Immutable canonical policy from section O with unique definition hash.

9. `backtest_runs`
   - One execution of a definition against exact replay and outcome builds;
     operational status, completeness counts, population/result hashes, and
     timestamps.

10. `backtest_population_members`
   - The audit set: run, cohort (`EPISODE_CASE` or `VALID_FALSE_CONTROL`), exact
     signal result, snapshot, nullable prediction, episode date, partition, and
     inclusion/exclusion reason. One membership is shared across labels/horizons.
   - It prevents an aggregate from depending on an unrecoverable ad hoc query and
     supplies the exact prediction/control IDs to an auditor.

11. `backtest_results`
    - One immutable aggregate row per run, cohort, signal, outcome/horizon and
      optional single slice dimension/value. It stores all section P counts and
      statistics, aggregation version, ordered member/outcome-set hash, quality,
      and fingerprint.
    - A separate `backtest_result_slices` table is unnecessary; baseline and slice
      rows use the same schema. No copied source observations are stored.

### R.4 Integrity and immutability

- Definitions, snapshots, predictions, population members, outcomes, and results
  reject `UPDATE` and `DELETE` after insert.
- Run/checkpoint status is the only controlled mutable state and is guarded by an
  explicit transition function. After `COMPLETE`, all fields are immutable.
- FKs and insert guards prove subject/T/build/signal/snapshot consistency and
  validate copied strength, severity, and quality against `SignalResult`.
- Exact replays use conflict-do-nothing followed by full content verification.
  Same identity/different content fails with a stable “use new build/version”
  error.
- Corrections append a new build/artifact with `supersedes_id`; they never
  cascade-rewrite published history.

## S. PIT / leakage controls

### S.1 Structural isolation

Research reconstruction and outcome evaluation use separate packages,
repositories, sessions, manifests, and database privileges:

```text
ReplayInputRepository
  may read: qualified source/universe/feature/factor/signal artifacts
  may write: upstream artifacts only through their existing immutable writers,
             plus replay checkpoints, snapshots, predictions
  cannot read: research_outcomes or backtest results

OutcomeRepository
  may read: published snapshots/predictions and qualified future datasets
  may write: research_outcomes
  cannot update: universe, features, factors, signals, snapshots, predictions

BacktestRepository
  may read: sealed replay and outcome artifacts
  may write: backtest population/results
  cannot update either ledger
```

Production deployment grants the replay database role no `SELECT` privilege on
outcome/backtest tables. Development tests additionally enforce an import and SQL
allowlist so feature, factor, signal, and replay modules cannot import or query
outcome models. A historical-backfill orchestrator must commit and close the
replay transaction before opening the outcome transaction. Passing an outcome
object into a replay runner is not an API option.

### S.2 Required leakage attacks

| Attack | Expected fail-closed behavior |
| --- | --- |
| Future filing or comparative fact | `available_at > T` is absent from the exact fundamental build; no fallback to the later row. |
| Future estimate revision/correction/alias | Exact dataset prefix through T only; current consensus cannot reconstruct the old state. |
| Future taxonomy correction or reclassification | Latest evidence **known by T** is selected before effective-interval filtering; future version is excluded. |
| Future split/dividend/merger action | It cannot enter a research feature at T. Outcome use is allowed only in the separate future outcome build. |
| Future factor or signal build | Exact manifest/T/build/hash mismatch; no global-latest lookup. |
| Future outcome row | Replay role/import boundary denies access; sealed snapshot hash must be identical before and after outcome publication. |
| Today's surviving universe/current security flags | Current `Security.is_active`, current ticker, and current membership are not queried by universe reconstruction. |
| Today's revised macro value | Only a qualified as-of vintage can enter a research manifest; ordinary FRED/latest view is rejected. |
| Same-day close before signal availability | Entry policy requires a session whose open is strictly after the decision; the already-finished close cannot be used. |

An adversarial regression must add each forbidden future row, rerun the exact T
checkpoint, and prove either an identical snapshot/prediction fingerprint or a
controlled failure with zero publication. Merely asserting a date filter in a
unit mock is insufficient.

### S.3 No hindsight optimization

- Outcome labels never select the universe, source row, feature, factor proof,
  signal, episode boundary, entry date, benchmark, missing-data treatment, or
  slice.
- Famous winners, failures, and false positives may be golden regression cases,
  but no threshold or endpoint is chosen because those cases look attractive.
- All backtest definitions and primary endpoints are sealed before opening a
  validation/holdout outcome build.
- A poor result creates a research finding and possibly a future v2 challenger.
  Phase 2.9 cannot change factor weights, signal thresholds, detector rules, or
  champion status.

## T. Replay checkpoint / resume

### T.1 Run state

Allowed run states and transitions are:

```text
PENDING -> RUNNING
RUNNING -> COMPLETE
RUNNING -> FAILED         # attempt stops before any checkpoint completes
RUNNING -> PARTIAL        # attempt stops after >=1 but before all checkpoints
RUNNING -> CANCELLED      # explicit operator cancellation
PARTIAL -> RUNNING        # explicit deterministic resume
FAILED  -> RUNNING        # explicit deterministic retry
```

`COMPLETE` and `CANCELLED` are terminal. `FAILED` therefore has a zero completed
watermark; `PARTIAL` has a nonzero, nonfinal watermark. Neither is eligible for a
complete backtest, and no failed checkpoint may be silently skipped.

Checkpoint states are `PENDING`, `RUNNING`, `COMPLETE`, or `FAILED`. A process
restart detects an expired `RUNNING` lease, records an append-only failed attempt,
rolls back the incomplete date transaction, and sets the run to `FAILED` or
`PARTIAL` according to its watermark. An explicit resume starts a new attempt for
that same deterministic checkpoint. Wall-clock leases and errors are operational
metadata, not part of the semantic snapshot.

### T.2 Transaction boundary

One scheduled research timestamp is the atomic publication unit:

```text
BEGIN date transaction
  validate exact manifests and quality
  reconstruct the full date universe
  run/reuse bounded feature/peer/factor/signal artifacts
  insert all date snapshots
  insert all determinable episode predictions
  verify counts, identities, and content hashes
  mark checkpoint COMPLETE
COMMIT
```

Existing runners may use nested savepoints and bounded 250-security/parameter-
safe batches, but the outer date transaction is all-or-nothing. The ten-year run
is not one transaction. A crash can at worst require recomputing the current
date; prior completed dates stay committed.

On resume, recompute the deterministic checkpoint identity from run fingerprint
and T, verify any complete checkpoint's manifests/counts/hash, and start at the
first non-complete date. Exact inserts reuse existing IDs after full-content
comparison. A content conflict fails the run and requires a new build/definition;
resume never “repairs” a row. Chronological order is mandatory because episode
state depends on the previous completed checkpoint.

A failed date stops later dates. `COMPLETE` requires every scheduled checkpoint,
including warm-up, to be complete and gap-free; the run stores the first/last T,
expected/completed counts, and ordered checkpoint hash.

## U. Quality propagation

Quality is a vector first and a weakest-level summary second.

| Artifact | Quality rule |
| --- | --- |
| Replay checkpoint | Weakest admitted provider/universe/identity/classification/raw/build quality over every required source and lookback. Must meet the requested target exactly or better. |
| Research snapshot | Weakest checkpoint admission and subject-specific universe/identity/classification/peer/factor/signal admission. Coverage counts remain separate from quality. |
| Prediction | Weakest snapshot research quality and referenced valid signal-result quality. Future evidence can never upgrade it. |
| Outcome | Weakest label source, market/action/terminal or filing/estimate, benchmark, calendar, and raw-reproducibility quality in the outcome build. |
| Backtest member | Preserves prediction/control research quality, outcome quality, benchmark quality, and survivorship status separately. |
| Backtest result | Weakest included member across those dimensions, plus the full quality distribution and limitations. |

An industry-excess label can be lower quality than its raw security return. A
qualified future price outcome does not improve a development prediction. A
survivorship-safe universe does not make an unqualified market path safe, and a
licensed market feed does not cure incomplete delisting history.

If any required member or label lacks survivorship qualification, the result must
say so and cannot be titled “survivorship-safe.” Development overrides appear in
every downstream provenance document and force the relevant artifact/result to
`DEVELOPMENT`.

## V. Provenance

Every published statistic must support this walk without consulting mutable
current state:

```text
backtest_result
  -> backtest_run + immutable definition
  -> exact population member IDs and ordered population hash
  -> prediction or valid-false SignalResult
  -> research_snapshot + replay checkpoint/run
  -> SignalResult conditions
  -> exact FactorValue / component / FeatureValue artifacts
  -> universe, identity, classification, taxonomy and peer evidence
  -> ResearchBuildManifest chain
  -> provider qualification/admission and verified raw evidence

backtest_result
  -> exact research_outcome IDs and ordered outcome hash
  -> outcome definition + outcome build manifest
  -> actual entry/exit/benchmark observations
  -> price/action/terminal or fiscal/estimate evidence
  -> provider qualification/admission and raw evidence
```

Every relevant manifest pins:

- full Git commit and clean-publication requirement;
- Alembic revision;
- dependencies;
- provider versions and qualification catalog version;
- raw inventory and source dataset/build IDs;
- universe, identity, taxonomy, feature, factor, signal, schedule, episode,
  entry, horizon, outcome, benchmark, terminal, and aggregation policy versions;
- T or outcome cutoff and explicit origin.

The snapshot stores exact relational references and canonical set hashes, not
every source byte. Large daily price paths are reconstructed from the pinned
immutable outcome dataset and protected by ordered observation/action hashes;
entry, exit, terminal, and benchmark anchors retain direct references. Raw bytes
remain in the qualified archive under its retention rights.

Same manifest, date range, schedule, and policy must reproduce the identical
ordered prediction fingerprint set. A different code/data/model/outcome version
produces a separate run. A hash is an integrity check, not protection against an
operator rewriting both content and hash; operational backups/independent
anchoring remain applicable.

## W. Scale strategy

### W.1 Row-count estimate

For 5,000 eligible securities, 52 checkpoints/year, and 10 years:

| Artifact | Approximate count before exclusions/reuse |
| --- | ---: |
| Security-date snapshots | `5,000 * 52 * 10 = 2.6M` |
| Security-scoped signal results | `2.6M * 7 = 18.2M` |
| Signal condition rows | about `110,002 * 520 = 57.2M`, using the measured 5k Phase 2.8 shape |
| Security factor values | about `40,001 * 520 = 20.8M`, including small node overhead |
| Factor subfactor rows | about `80,003 * 520 = 41.6M` from the frozen scale shape |
| Factor component decisions | about `195,008 * 520 = 101.4M` from the frozen scale shape |
| Predictions | Data-dependent episode starts; necessarily much smaller than all fired continuations |
| Outcomes | Materialized once per needed snapshot/label/horizon and reused across signals; not all labels for every weekly row by default |

The frozen 5,000-security scale measurements are 647.026 seconds for factors and
154.791 seconds for signals per snapshot on the designated 2-vCPU/8-GB host.
Naively repeated for 520 dates, those two stages alone are about **116 serial
hours**, before source features, peers, outcomes, validation, or I/O. This is a
planning lower bound, not an SLA.

Wide JSON provenance and indexes make a byte estimate from row count alone
unreliable. More than 200 million factor/signal lineage rows can plausibly require
tens to hundreds of gigabytes once indexes and operational headroom are included.
Implementation must measure `pg_total_relation_size`, bytes/row, index size, WAL,
temp I/O, runtime, RSS, and outcome-cache hit rate on the staged scale tests before
authorizing a decade-wide run.

### W.2 Persistence and caching policy

- Persist every frozen runner artifact its contract requires. Phase 2.8's complete
  valid-false/missing/invalid result and condition graph is not pruned.
- Persist one compact snapshot for every screened candidate, including exclusions.
- Persist predictions only for episode starts.
- Persist outcome rows only for prediction cases and controls required by a sealed
  backtest/live maturity definition. Cache by snapshot/definition/horizon/outcome
  build so co-occurring signals reuse one path.
- Reuse upstream artifacts only on exact manifest identity. A wrong build is a
  miss, not a cache candidate.
- Process set-based subject batches using the existing 250-security runner shape;
  bulk-insert within PostgreSQL parameter limits. Prefetch definitions/builds and
  avoid security/date/signal N+1 queries.

### W.3 Development scale and parallelism

First implementation qualification is:

```text
1 year weekly * 1,000 securities
```

The measured factor+signal lower bound for that matrix is about 2.25 serial hours
before upstream work. Measure SQL statements, runtime, peak RSS, PostgreSQL temp
I/O, row counts, relation/index bytes, prediction count, and outcome count. Only
then expand to multiple years/5k and, if justified, 10k.

On the current 2-vCPU/8-GB machine, process one research date at a time. Bounded
in-date computation may use at most modest worker parallelism, but one publisher
owns the date/checkpoint transaction. Later hardware may partition read-only
calculation by subject batch; chronological date publication and episode derivation
remain serialized. No Spark, Kafka, external feature store, or distributed control
plane is introduced.

PostgreSQL remains authoritative. Physical range partitioning of new snapshot/
outcome tables may be selected only after the 1k-year measurement; frozen factor
and signal tables are not redesigned speculatively.

## X. Worked examples

All names, IDs, and prices below are fictional.

### X.1 Early Growth episode that matures unevenly

At the scheduled research cutoff `2021-06-26 00:00:00Z` (Friday June 25 at 20:00
New York), Atlas Compute has an immediately preceding completed observation of
`early_growth_candidate_v1 = VALID/false`. Its exact T result becomes
`VALID/true`, strength `0.60`, severity `MEDIUM`, and `PIT_QUALIFIED`. The signal
result references its immutable factor build and selected Market confirmation;
Fragility Warning also fires and stays visible.

The replay appends one `FIRED_START` prediction. Because Friday's session had
already opened before T, `next_regular_session_close_v1` selects Monday June 28's
close:

```text
research_timestamp:       2021-06-26 00:00:00Z
prediction origin:        HISTORICAL_REPLAY
prior signal:             VALID false
current signal:           VALID true
entry session/close:      2021-06-28 / 100.00
3m boundary/exit:         2021-09-28 / 118.00 total-return wealth basis
6m boundary/exit:         2021-12-28 / 110.00
12m boundary/exit:        2022-06-28 / 95.00
24m boundary/exit:        2023-06-28 / 140.00
```

Fictional labels:

| Horizon | Total return | SPY excess | Industry excess | Max drawdown | MFE |
| --- | ---: | ---: | ---: | ---: | ---: |
| 3m | `+18%` | `+14%` | `+10%` | `-12%` | `+25%` |
| 6m | `+10%` | `+4%` | `+1%` | `-22%` | `+30%` |
| 12m | `-5%` | `-3%` | `+3%` | `-35%` | `+30%` |
| 24m | `+40%` | `+15%` | `+9%` | `-35%` | `+52%` |

The next fiscal-quarter revenue-acceleration label matures when that primary
report is available and records `+0.08` percentage points. The 3m row can become
`MATURE_VALID` while 6m/12m/24m and the filing label are still `PENDING`. Each
transition appends an outcome under its data build. The later 12m disappointment
does not rewrite the signal, prediction, strength, or positive 3m outcome.

### X.2 False positive

Beacon Devices makes the same valid false-to-true Early Growth transition with
entry close `50.00`. At 3m its qualified total-return wealth is `40.00`, SPY is
`+3%`, industry is `+1%`, maximum drawdown is `-38%`, and next-quarter revenue
acceleration is `-0.12` percentage points:

```text
total return       = -20%
SPY excess return  = -23%
industry excess    = -21%
max drawdown       = -38%
```

Beacon remains an episode and a mature adverse observation. It cannot be deleted,
reclassified as “not really fired,” or excluded because the result is poor. A
future signal v2 hypothesis may cite it; v1 remains frozen.

### X.3 Terminal outcome

Cascade Systems fires Fragility Warning, later delists, and has a qualified final
cash liquidation worth 20% of entry wealth. The path records total return `-80%`
and max drawdown `-80%`, then holds cash to each later horizon. If that terminal
proceeds evidence were absent, the same label would be `CENSORED`, not dropped
and not assumed `-100%`.

## Y. Test matrix

All tests use deterministic synthetic fixtures or repository-owned golden data.
No test requires internet access or a live provider.

### Y.1 Definitions, manifests, and quality

- Exact schedule, episode, entry, horizon, label, benchmark, terminal,
  aggregation, and signal-objective registries and hashes.
- Manifest canonicalization, Git/schema/dependency/source/build pins, dirty or
  malformed payload rejection, and changed-policy/new-build isolation.
- Every quality target and lattice boundary; full lookback coverage; catalog
  mismatch; synthetic opt-in; structured development override; no silent
  downgrade; production plus survivorship joint gate.

### Y.2 Universe and replay

- Effective-before/at/end-exclusive membership, future-known changes, corrected
  versions, ticker reuse, re-entry, primary-security ambiguity, security type,
  delisted historical inclusion, classification release and stable-node lineage.
- Current `is_active`, ticker, sector, industry, and present-day universe cannot
  affect T.
- Exact feature/factor/signal T/build reuse; wrong/future artifacts rejected;
  no generic latest fallback; missing remains missing.
- Chronological ordering, warm-up exclusion, exact rerun fingerprints, different
  build isolation, and no outcome-table access from replay.

### Y.3 Episodes and controls

- `false->true` emits once; `true->true` emits none; `true->false` closes by
  derivation; `false->true` after close emits a new episode.
- Initial true, new-universe true, missing/invalid gap, failed checkpoint, and
  schedule gap are left-censored and do not emit primary predictions.
- Strength/severity changes during true do not restart an episode.
- Node and security identities cannot cross; version changes are separate.
- Controls include only same-date eligible valid false; fired, missing, invalid,
  and excluded states are separately counted.

### Y.4 Entry, market path, and benchmarks

- After-close, pre-market, exact-open, weekend, DST, holiday, early-close, and
  unexpected-closure cases under the pinned calendar.
- Strict no-same-bar behavior and three-session entry/exit search limit; stale
  last-price rejection and suspension handling.
- Splits, reverse splits, dividends, multiple actions, action corrections,
  incompatible share basis, provider adjusted-close trap, and exact Decimal path.
- Aligned SPY/industry interval, min-three industry membership, daily membership
  changes, delisting, acquisition cash, stock conversion, zero bankruptcy
  recovery, unknown terminal censoring, and missing benchmark quality.
- Hand calculations for total/price/excess return, drawdown, MFE, volatility, and
  calendar-month end clamping.

### Y.5 Fundamental and estimate labels

- Explicit next fiscal successor; non-calendar year; unresolved successor;
  after-hours first report; amendment/restatement after first report; first-report
  definition never substitutes latest-restated data.
- Revenue growth/acceleration, operating-margin delta, FCF-margin delta, missing
  CapEx, zero/negative denominator, and source-build lineage.
- Exact same estimate target/profile/series at 30d/90d, FQ1 rollover rejection,
  correction/withdrawal/member changes, report-before-boundary censoring, and
  incomplete history.

### Y.6 Maturity, immutability, and corrections

- Pending is null and excluded from mature denominators; boundary transition to a
  new mature row; missing/censored/disputed distinction; partial future coverage.
- Exact same-build idempotency; corrected source creates a new outcome build and
  superseding row; old prediction/outcome/result remains unchanged.
- UPDATE/DELETE rejection, conflicting content failure, atomic parent/child
  publication, injected rollback, and concurrent one-winner publication.

### Y.7 Deterministic three-company golden replay

The foundational fixture has companies A, B, and C over four weekly T values:

| Subject | T0 | T1 | T2 | T3 | Expected episode |
| --- | --- | --- | --- | --- | --- |
| A | false | false | true | true | One start at T2 |
| B | false | false | false | false | No prediction; valid control at T1/T2 |
| C | false | true | true | terminated | One start at T1; qualified terminal outcome |

Use tiny raw closes/actions, SPY, a three-member industry, one dividend, and C's
documented terminal proceeds. Hand-compute A's positive return/excess and path
risk, B's control outcome, and C's terminal return. Assert snapshot/prediction/
outcome/result IDs and hashes, control counts, pending versus mature counts, one
episode per transition, and no dropped delisting. Add a fourth unavailable signal
state without changing the three-company universe denominator.

### Y.8 Leakage falsification

Run every attack in section S.2, including a future filing, estimate, taxonomy
correction, action, signal build, outcome, surviving-universe edit, revised macro
vintage, and pre-signal same-day close. Each must preserve the sealed T artifact
or fail before publication.

### Y.9 Checkpoint, concurrency, and scale

- Failure before/after each publication stage, process interruption, expired
  lease, restart, exact resume, no duplicates, no skipped date, and complete-run
  gap detection.
- Two writers for one checkpoint converge or one fails cleanly; readers never see
  a completed marker with a partial date graph.
- `1 year weekly * 1,000 securities`: SQL count, runtime, RSS, temp I/O, relation/
  index/WAL bytes, snapshot/signal/prediction/outcome counts, query plans, and
  cache reuse. Expand only after measured evidence.

## Z. Open decisions

Only these external or measurement-dependent decisions remain:

1. **Production providers and rights.** Select and qualify the price/action/
   terminal, universe/security/classification, estimates, fundamentals, and
   calendar sources over the intended historical interval. No current live
   provider set satisfies the production/survivorship contract.
2. **Versioned production market-observation store.** Choose the canonical
   provider/build-aware raw OHLCV correction identity and action/terminal feed
   that can satisfy the outcome contract. The existing `market_prices` uniqueness
   and Yahoo basis cannot do so.
3. **Exchange-calendar evidence source.** The policy semantics are fixed, but a
   production calendar must qualify official opens/closes, early closes, DST,
   and exceptional closures. The current deterministic calendar remains suitable
   for development fixtures only.
4. **Physical storage layout.** Decide whether new replay/outcome tables require
   yearly range partitions and what retention/index layout is affordable only
   after the required 1k-year size/I/O measurements.
5. **Actual development/validation/holdout years.** Freeze them after a blind
   source-coverage audit and before label inspection. The example years in prior
   documents are not adopted here.

Sector benchmarks, daily cadence, matched controls, market-cap/liquidity filters,
event-driven replay, portfolio simulation, confidence intervals, significance,
regimes, qualitative labels, and calibration are resolved as deferred scope, not
open v1 choices.

## AA. Recommendation

**SAFE TO IMPLEMENT PHASE 2.9**

Implementation must begin with the frozen implementation specification and
synthetic golden replay, preserve every fail-closed gate above, and make no
production or survivorship claim until the open provider qualifications are
actually satisfied. Phase 2.9 must stop after replay, immutable ledgers, outcome
calculation, and descriptive signal-event evaluation. It must not modify Phase
2.8 signals or begin Phase 2.10 calibration.
