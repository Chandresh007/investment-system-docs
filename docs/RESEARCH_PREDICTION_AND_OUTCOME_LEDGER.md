# Research prediction and outcome ledger

Status: FUTURE DESIGN, 2026-09-21. No tables or services implemented by this review.
The ledger must preserve what was asserted, including abstentions and failures,
without rewriting the prediction after the answer becomes known.

## Record boundaries

A research run owns a frozen eligible universe and a manifest. Each security's
snapshot produces one prediction packet, including watch/pass/insufficient-data
judgments. Record all screened names and exclusions, not just eventual winners.
Separate `LIVE_SHADOW`, `LIVE_CHAMPION`, `HISTORICAL_REPLAY`, and `EDUCATIONAL`
origins. A replay produced today is never represented as a prediction issued years ago.

| Proposed record | Required contents |
| --- | --- |
| Research run | Run ID, requested research timestamp/timezone, actual created_at, mode, parent run if reconstruction, universe build/hash, eligibility/exclusion manifest, execution policy, data cutoffs and publication completeness |
| Frozen snapshot | `research_snapshot_id`, stable security/company/share-class identity, ticker as known then, research timestamp, canonical payload hash, schema version, data build/version and raw manifest, git commit and environment/dependency manifest |
| Model identity | Feature versions, factor version, signal version, model version, calibration version, universe/benchmark/availability-policy versions, parameters and training cutoff |
| Evidence | Exact feature IDs, values, units, statuses/reasons, input timestamps and provenance; factor values and component contributions; signal and trigger evidence; score/rank and ranking-universe denominator when applicable |
| Judgment | Thesis, catalysts, risks, invalidation conditions, expected horizon, objective family, expected evidence, confidence/calibration bucket, human overrides with author/reason/time |
| Thesis assertion | Assertion ID, claim type, target fiscal identity/metric, baseline evidence, direction/threshold, deadline, acceptable evidence sources, adjudication rule/version, dependency on other assertions |
| Narrative artifact | Optional AI output, model/prompt/template version, supplied snapshot hash, citations, generation time; never an untracked replacement for deterministic judgment |

Snapshot values must be copied or referenced through immutable versioned artifacts,
not just linked to mutable legacy FeatureValue rows. Seal the payload atomically
with its manifest. Repeated identical requests are idempotent; conflicting content
at the same identity fails and requires a new run/build. No UPDATE/DELETE path for
published predictions. Corrections append a superseding record with reasons; the
original remains queryable. Sign or independently anchor manifests when operationally
appropriate; a hash alone cannot prevent an operator rewriting both payload and hash.

## Outcome records

Proposed key: `(research_snapshot_id, label_definition_version, horizon,
outcome_build_id)`. Each record includes observation-window start/end, scheduled
maturity, actual evaluation timestamp, latest evidence availability, value/unit,
status/reason, evidence IDs/hashes, benchmark definition, and correction lineage.
A later outcome revision creates a new record, never a changed original prediction.

Default horizon family: **3m, 6m, 12m, 24m**, measured as calendar anniversaries.
Version the rule for holidays and non-trading endpoints. Proposed daily policy:
entry is first executable session after snapshot publication plus processing lag;
exit is the first tradable session on/after the calendar horizon date. Store actual
entry/exit instants. Exact-close observations do not justify trading at that same
close. Halts, suspensions and unavailable exits need explicit handling.

| Label family | Definition and cautions |
| --- | --- |
| Raw stock return | Unbenchmarked holding-period total return including ordinary distributions and terminal proceeds; also store price-only return separately, never call adjusted-price return raw input data. |
| SPY excess return | Stock simple total return minus SPY simple total return over identical entry/exit; benchmark availability and currency must match. |
| Sector excess return | Same comparison against sector benchmark chosen by the frozen PIT policy; no present-day sector substitution. |
| Industry excess return | Explicit historical basket methodology with membership and missing/delisting policy; distinguish investable simple-return basket from market_v1's mean-log-return research benchmark. |
| Maximum drawdown | Minimum `wealth_t/running_peak_t - 1` over the forward holding path, including loss at delisting when known. |
| Risk-adjusted outcome | Optional daily excess-return mean / sample volatility, annualized under declared assumptions; require enough observations. A single holding-period return is not a Sharpe ratio. |
| Delivered fundamentals | Future revenue/EPS growth, margin expansion and FCF levels/growth for explicitly named fiscal periods and accounting bases. Zero/negative denominator domains remain explicit. |
| Future estimate revisions | Same absolute target and comparable series; record magnitude and economic activity separately, plus coverage limitations. |
| Catalyst / invalidation | OCCURRED / NOT_OCCURRED / UNKNOWN / NOT_YET_OBSERVABLE with event evidence and predeclared deadline. Absence of news is not proof of absence. |
| Thesis result | SUPPORTED / CONTRADICTED / MIXED / INDETERMINATE / PENDING from assertion rules; optional success/failure summaries require a predefined rubric. |

Maintain `PENDING`, `MATURE_VALID`, `MATURE_MISSING`, `CENSORED`, and `DISPUTED`
outcome states. A 24-month price horizon maturing does not mean a future accounting
report has been published. Each fundamental label has its own evidence maturity.
Acquisitions, bankruptcy, delisting and corporate conversions need terminal payout
or loss treatment; dropping such names biases results. Report unknown terminal
values and sensitivity bounds instead of silently excluding them or assuming zero.

For financial actuals keep two explicit label definitions where useful:
**first reported** and **latest restated as of evaluation cutoff**. Neither may be
silently substituted for the other. Outcome data may occur after research T by
design; feature inputs may not. Outcome workers cannot write to feature manifests.

## Define success before measuring it

| Objective family | Useful primary evidence | Complementary checks |
| --- | --- | --- |
| Early-growth detection | Delivered acceleration over next named quarters; 6/12m excess return | Estimate changes, false positives, valuation, downside |
| Compounder detection | Sustained growth, margins and cash generation; 12/24m excess return | Drawdown, reinvestment, business quality and entry valuation |
| Shorter-term catalyst | Event occurrence by deadline; 3/6m excess return | Announcement versus execution, slippage and event downside |
| Risk detection | Future drawdown, adverse events and thesis invalidation | Missed upside, false alarms, sector/regime exposures |

No universal winner label. Pre-register primary objective, secondary diagnostics,
benchmark, horizon, eligible cohort and downside guardrails for each experiment.
Report all objective families, even when the primary one looks good. Business
success can coexist with a falling stock if expectations were excessive.

## Synthetic thesis example

At T, a fictional company announces additional capacity. Freeze the claim:
“Revenue YoY growth will rise from 15% to at least 20% by explicit FY2027 Q2,
supported by capacity entering service by the stated date.” Expected evidence:
reported revenue acceleration, gross margin at least its recorded baseline, and
same-target estimate upgrades. Risks: execution delay and customer concentration.
Invalidation: documented capacity delay past deadline, customer loss, or gross
margin falling below a separately predeclared threshold.

Later, adjudicate each assertion from eligible reports/events. Record a price
loss independently of whether capacity and revenue improved. If causality cannot
be established, say so: co-occurrence does not prove that capacity caused growth.
Thresholds here illustrate a packet, not a proposed production signal.

## Acceptance gates for future implementation

Demonstrate append-only persistence, complete-universe recording, exact replay
hashes, outcome isolation, pending/late labels, correction lineage, independent
expected returns, delisting treatment, and concurrent publication atomicity.
Retain records for the project's lifetime subject to source retention rights;
if rights require deletion, preserve permitted derived audit metadata and an
explicit tombstone rather than promising unlawful perpetual raw retention.

## Qualitative claims, disagreements and evidence-family outcomes

Future extension: freeze a typed thesis graph with CLAIM, CAUSE, EXPECTED_OUTCOME,
CATALYST, RISK, INVALIDATION_CONDITION and EVIDENCE nodes. Preserve immutable node
and edge versions, company/segment scope, attribution, timestamp, baseline, target
fiscal identity, horizon/deadline, units/basis, thresholds, evidence policy and
adjudication version. A hypothesized cause is not a verified cause. See the
[qualitative design](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md).

Add packet references to BusinessQualityResearch dossiers, scoped moat hypotheses,
CompanyStrategicEvents, professional-report versions/rights metadata, corroboration
records and disagreements. Preserve a family vector—fundamentals, estimates,
market, valuation, business quality/moat, events, professional research and
alternative evidence—with original direction, confidence/reasons and missingness.
Do not immediately collapse the vector into one unexplained AI score.

Proposed outcome key: `(snapshot_id, node_id, node_version, outcome_definition_version,
horizon, outcome_build_id)`. Store adjudication status, value/unit where meaningful,
evaluation time, evidence availability, corroboration/contradiction links, reviewer
and correction lineage. Risk materialization and catalyst completion have their own
records even when the overall thesis succeeds. Unknown evidence remains indeterminate.

| Later question | Record / evaluation design |
| --- | --- |
| Which thesis claim was correct? | Match its frozen scope, threshold, fiscal target and deadline to independent outcome evidence |
| Which risk materialized? | Record occurrence/severity/time and impact evidence, without retroactively inserting the risk |
| Did professional research disagree? | Preserve original external stance/report version alongside internal stance, horizon and valuation assumptions; evaluate comparable assertions only |
| Did alternative evidence confirm management? | Compare frozen management claim and then-available proxy support with later direct evidence; do not let the proxy validate itself |
| Which evidence family was informative? | Matched-cohort diagnostics and out-of-sample ablations, reporting coverage, uncertainty, shared origins and all tried hypotheses |

A professional analyst may be right about overvaluation while management is right
about revenue growth. Market momentum measures another horizon. Use NOT_COMPARABLE
when predicates differ; do not award a single “winner.” Predictive association is
not causal credit for a family. If an external report supplied both an input and
the outcome opinion, flag circular evaluation rather than treating it as ground truth.

Newly discovered evidence, report edits, revised extraction and changed theses
create new versions/packets; sealed history remains unchanged. Assess learning
across thousands of predictions through existing maturity, walk-forward, multiple-
testing and champion/challenger gates. Evaluation can recommend new candidates;
it cannot silently change production formulas, weights, ratings or confidence rules.
