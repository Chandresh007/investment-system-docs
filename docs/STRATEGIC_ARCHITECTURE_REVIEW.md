# Strategic research architecture review

Date: 2026-09-21. Reviewed checkpoint:
`a5a80ef8bc594500000b4f357192bc82513b5bb8`, master = origin/master, initially clean.
Reconstructed with fetch, checkout master, ff-only pull and 20-commit history.
Read README, project state, complete blueprint/curriculum, frozen market/estimate
specifications and estimate implementation audit; inspected relevant source and
primary provider documentation. This is a strategic review, not an exhaustive new
implementation audit or a Phase 2.5 independent freeze decision.

## Remediation follow-up

This document preserves the original review evidence. Implementation and current
risk verdicts now live in [FOUNDATION_REMEDIATION_AUDIT.md](FOUNDATION_REMEDIATION_AUDIT.md).
The original table is not a claim that the repaired code defects still exist.
Production coverage, source qualification and recovery conditions remain distinct.

## Verdict: CONDITIONAL PASS

The architectural direction can support deterministic, broad, explainable research
and controlled empirical improvement. The current repository is a tested feature
foundation, not yet an end-to-end historically certified investment research system.
An unconditional PASS would confuse synthetic correctness with qualified source
history, survivorship-safe universes, immutable predictions and predictive validity.

No application code, migrations or frozen financial formulas changed. Phase 2.5
remains IMPLEMENTATION COMPLETE / READY FOR INDEPENDENT REVIEW / NOT YET FROZEN.
Phase 2.6 has not started. The documentation review is complete; the conditions
below remain future engineering/review gates, not completed fixes.

## Assessment by layer

| Area | Assessment | Evidence and limit |
| --- | --- | --- |
| PIT foundation | Conditional | Timestamp-aware resolvers exist; knowledge-time semantics are uneven across sources |
| Fundamentals | Useful deterministic foundation | 15 metrics; explicit missingness and fiscal matching; filing-date timing and comparative-context qualification remain |
| Market | Frozen math, conditional source validity | 22 features with raw→split→TRI intent; Yahoo raw basis/actions not qualified |
| Estimates | Strongest versioned subsystem | Pinned builds, immutable events, replay, corrections, same-target revisions; only synthetic provider qualified |
| Historical universe | Blocking gap before peer expansion | Effective intervals exist; no knowledge-time/versioned supersession or broad historical coverage proof |
| Data-source quality | Not production qualified overall | Current Yahoo/FRED/RSS data do not establish historical-as-known archives |
| Backtester | Designed here, absent in code | Must isolate inputs from outcomes and handle execution, delistings and coverage |
| Calibration | Designed here, absent in code | Rules first; chronological validation and independent promotion required |
| Qualitative/news | Planned with basic ingestion | Evidence-backed typed assertions, valuation and neutral external-event context |
| Durability | Design exists, operation unproven | Local raw bytes and DB require off-host backups and restore drills |
| Generalization | No empirical claim possible yet | Golden tests are not broad-universe unseen-period investment tests |

## Conditions and priorities

Paths in this table are relative to `src/investment_research/`.
Severity is architectural priority, not a claim that every historical output is affected.

| ID / gate | Observed evidence | Consequence | Required future closure |
| --- | --- | --- | --- |
| R1 — before 2.6 peer features | `models/research.py:HistoricalUniverse` has start/end and created_at, no available_at or correction chain; `research/benchmark_resolver.py` filters effective intervals only | A retrospectively inserted/reclassified member can alter a past basket; schema presence does not cure survivorship bias | Versioned knowledge + effective-time membership, stable identities, complete failed/delisted coverage; adversarial late-correction tests |
| R2 — before historical fundamental claims | `normalization/sec.py` parses filed date at midnight; `research/resolver.py` filters it directly; no fundamental join to Filing acceptance | Same-day pre-release visibility can leak; current assumptions overstate intraday PIT | Qualified accession dissemination/timezone or conservative date-only availability, versioned policy; test release-minus/at boundaries |
| R3 — before broad fundamental expansion | Normalizer copies `fy/fp` for each Company Facts entry; comparative contexts need accession-level interpretation | Source labels alone may identify reporting context rather than the comparative period intended by a calculation | Independent comparative/restatement/fiscal-change corpus; explicit context mapping, preserve ambiguity; no calendar-month shortcut |
| R4 — before market backtest claims | Yahoo chart quote.close treated as raw; action engine adds split/dividend adjustments; qualified live action ingestion not established | Retrospective normalization or double adjustment; incomplete distributions/terminal returns | Document/prove raw price/volume basis, action knowledge and corrections with independent examples, or qualify replacement provider |
| R5 — before macro enters historical signals | FRED request sends only series_id; economic observations unique by series/date with conflict-do-nothing | Latest/first-ingested values are not a vintage archive | Vintage-aware connector + versioned persistence/release availability + replay tests |
| R6 — before any frozen research judgments | `market_runner._persist_value` overwrites matching rows; generic FeatureValue is not a prediction ledger; estimates have stronger conflict checks | A later rebuild can change evidence referenced by an old judgment | Immutable copied/versioned snapshots, run manifests and outcome isolation; never rely only on legacy row IDs |
| R7 — before more durable live collection | `storage/raw_storage.py` writes same identifier/second path with wb; hash check absent in generic read; estimates add hash-based paths/checks | Two same-second payloads may overwrite raw evidence outside estimates | Content-addressed/exclusive raw publication, hash verification and collision tests across connectors; no existing data repair without provenance |
| R8 — before production estimates | Audit documents synthetic-only profile, manual consensus-through-T ordering and JSON evidence IDs | Missing publication can look stale; rights/coverage unknown; no feature-evidence FK guarantee | Independent 2.5 review, production profile contract, orchestration-completeness gate, evidence integrity decision, scale benchmark |
| R9 — before reliability claims | Local raw store; no checked-in backup/restore workflow found or executed | Host loss can destroy reproducibility | Encrypted off-host DB+raw backup and successful isolated restore with manifests |

Address R1–R4, R6–R7 at the next foundation review before building more feature
families on top. Macro may remain excluded until R5 closes; production estimate
and operations gates R8/R9 can be pursued independently. This task documents the
contracts rather than making an unreviewed shared-schema or frozen-calculation
change. No evidence established that production outputs are currently deployed;
all broad historical/production claims remain gated.

## Market-specific conclusions

The intended raw price + PIT-known actions → normalized price → TRI design is
correctly separated in code. `MarketPriceResolver` uses retrieval cutoff and actions
use availability/created_at cutoff. This blocks backdating today's download but
does not certify provider input basis. See [source review](DATA_SOURCE_TRUST_AND_PIT.md)
for Yahoo/yfinance primary-source evidence and limitations.

Additional independent-review questions:

- Dividend `amount_per_share` is added directly to split-normalized prices. Qualify
  its share basis across later splits; a pre-split dividend and post-split price
  cannot safely mix. No multi-action production qualification was established.
- Frozen industry return averages constituent log returns. `mean(log(1+r))` differs
  from `log(1+mean(r))`, the latter corresponding to a rebalanced equal-weight
  simple-return basket. Preserve market_v1 semantics; do not label it an executable
  portfolio without a separately versioned return methodology.
- Code `vol_trend_21` uses latest 21 / previous 220 observations (241 total). The
  spec's written denominator indices span 200 while its count says 241; registry
  description is also inconsistent. README describes actual code. Resolve the
  documentation contract deliberately in independent review, not by changing math.
- Code momentum acceleration requires 127 prices for two contiguous 63-interval
  returns; the spec table says 128. Existing blueprint already flagged this.
- Benchmark inception, ticker identity, missing constituents, suspensions and
  delisting proceeds require tests beyond the synthetic minimum-three basket.
- MarketPrice uniqueness omits provider/build/version; generic normalizer cannot
  preserve revisions alongside original observations. Future abstraction must
  include canonical versioned storage, not merely an interchangeable HTTP client.

## Closed-loop research recommendation

Freeze every eligible security's judgment, including abstentions, with input/build,
feature/factor/signal/model/calibration versions and thesis assertions. Record future
3/6/12/24m price, relative-return, risk and fundamental outcomes separately. Preserve
first-reported and restated labels with explicit versions. No future labels enter
features; no prediction is rewritten after outcomes.

Use rules and simple baselines, chronological walk-forward validation, maturity
checks and purging/embargo when label windows overlap. Reserve untouched periods,
log all trials, report failures and regime slices. Evaluate multi-objective families:
early growth, compounders, catalysts and risk. Champion/challenger promotion is a
recorded decision on predefined evidence, never an automatic formula rewrite.

Golden cases teach mechanics. They must include failed growth, value traps, bubbles,
cyclical peaks and false positives. Famous winners cannot be the sole optimization
set. Broad historical universes, unseen periods and prospective evidence are required.

## Sequencing decision

Keep existing 2.6 peers → 2.7 factors → 2.8 signals → 2.9 backtester → 2.10 calibration
→ 2.11 AI analyst numbering. The proposed alternative silently moves an existing AI
phase; it is unnecessary. Add explicit prerequisite workstreams instead:

1. Independent Phase 2.5 review plus foundation/data/durability qualification plan.
2. 2.6 historical membership knowledge/coverage before relative peer features.
3. 2.7 inspectable factors with versioning; valuation context may develop alongside.
4. 2.8 minimal immutable snapshot/prediction publication before shadow predictions.
5. 2.9 outcome ledger and replay before performance claims.
6. 2.10 calibration, generalization and champion/challenger gate.
7. 2.11 AI/deeper snapshots only with valuation, assertion/event contracts and citations.
8. Proposed 2.12 production event intelligence; 2.13 broader quality/source coverage.

Basic event contracts can be designed before the later production event phase.
DCF, valuation bands, peer multiples and quality evidence have named dependencies
in the blueprint/comparison; they must not disappear behind a growth-only ranking.
No next-phase implementation is authorized by this recommendation.

## Validation and review limits

Baseline full suite reproduced: **378 passed, 27 warnings, 0 failures, 101.75s**.
Warnings are existing datetime/SQLAlchemy deprecations. Alembic current/head both
**25e001**. Prior implementation audit records **295 focused estimate tests**;
this strategic review runs the full suite, which includes those tests, rather than
claiming a new standalone focused run. Final documentation validation is recorded
in PROJECT_STATE. No new empirical backtest, live-provider sample qualification,
license determination, restore drill, migration or scale test was performed.

All findings distinguish source-inspected behavior, external documented semantics,
and future recommendations. Detailed contracts are linked from README and
[external handoff](EXTERNAL_REVIEW_HANDOFF.md). A skeptical independent reviewer
should try to falsify both this review and the implementation assumptions.
