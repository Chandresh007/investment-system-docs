# External independent review handoff

> **Current review target:** Phase 2.6 implementation is complete and ready for
> independent review, but is **not yet frozen**. Foundation and Phase 2.5 remain
> frozen. Phase 2.7 has not started. The older foundation/Phase 2.5 challenge brief
> later in this file is retained as a clearly historical review record.

Audience: a skeptical independent reviewer, including Claude or another model.
Do not assume the specification, implementation-audit PASS, or prior test results
are correct. Try to falsify them. Distinguish reproducible defects from hypotheses
and unqualified production assumptions. Do not silently change frozen formulas,
declare a freeze by implication, or begin Phase 2.7.

## Phase 2.6 review handoff

Review current `master` and begin with:

- [Phase 2.6 frozen specification](phase-2.6-peer-industry-spec.md)
- [Phase 2.6 architecture review](phase-2.6-peer-industry-architecture-review.md)
- [Phase 2.6 implementation audit](phase-2.6-implementation-audit.md)
- [Current project state](PROJECT_STATE.md)

The implementation checkpoint can be located after publication with:

```bash
git log -1 --format='%H' --grep='^Complete Phase 2.6 implementation audit$'
```

Important code is under `src/investment_research/research/peers/`,
`src/investment_research/models/peers.py`, migrations `p26001` through `p26007`,
and `tests/peers/`. Review the actual manifest, provider catalog, frozen source
FeatureValue shapes, and database triggers; do not infer behavior from docs alone.

Reproduce the current gates in a dedicated development environment:

```bash
cd ~/Document/investment-research-system-master
source .venv/bin/activate
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
python -m pytest -q tests/peers
python -m pytest -q
alembic current
alembic heads
alembic check
git diff --check
```

Expected checkpoint results are **77 peer tests**, **508 full tests**, **8 existing
warnings**, zero failures, and Alembic current/head `p26007`. Inspect the scripts
before running them: `scripts/verify_peer_migrations.py` and
`scripts/verify_peer_scale.py` each create and force-drop only a random disposable
PostgreSQL database. Neither should downgrade or populate the configured research
database.

Try especially to falsify these claims:

1. Evidence heads are selected within the exact sealed dataset and knowledge
   cutoff before effective intervals are applied; future corrections and later
   sealed builds cannot leak backward.
2. Active taxonomy release, release-specific hierarchy, exact node, target
   exclusion, permanent-ID order, five-peer threshold, and no-fallback policy are
   deterministic at every boundary.
3. One-primary-security ambiguity is global to the historical eligibility corpus,
   not only the selected taxonomy node, and never relies on current ticker flags.
4. Membership is preserved when a feature is absent, invalid, stale, wrong-unit,
   wrong-build, incompatible, or unqualified. Coverage uses membership peers, not
   only convenient valid observations.
5. Fundamentals use latest measured quarter then latest known value; estimates
   and market require exact T/D; missing/invalid latest rows never fall back.
6. Every included source row has its own catalog-consistent admission covering the
   selected artifact. Synthetic, limited, or overridden evidence can never be
   upgraded above `DEVELOPMENT`.
7. Median, target-minus-median, and ascending midrank use finite exact Decimals;
   ties, zero, negatives, 5/5 counts, and the 60% boundary match the specification.
8. Every member, observation, result, industry metric, and projected FeatureValue
   can be walked through relational evidence to the pinned build, and cross-security
   evidence substitution is rejected at the database boundary.
9. Exact reruns reuse IDs/fingerprints/timestamps, conflicts require a new build or
   policy, UPDATE/DELETE remains prohibited, and an outer failure leaves no partial
   publication.
10. The 10,001-security run truly uses window/DISTINCT-style set operations and
    bounded publication rather than hidden per-security reads or a daily Cartesian
    materialization. Evaluate its plans and memory, not just its query count.
11. Industry metrics include all eligible members (including the anchor), use
    valid-only breadth denominators, keep zero non-positive, and do not confuse
    their threshold with target-relative five-peer semantics.
12. No Phase 2.2, 2.4, or 2.5 formula was altered, no z-score/winsorization/factor/
    signal/ranking/backtest was introduced, and inactive peer definitions cannot
    enter legacy feature scans.

Production qualification is explicitly outside the PASS. No commercial historical
taxonomy, primary-common-equity universe/security master, source-feature corpus,
or associated rights have been approved. A frozen legacy source row without
adequate row-level admission must fail closed. The scale fixture is synthetic and
does not prove an operational SLO or investment usefulness.

Return PASS / CONDITIONAL PASS / FAIL with prioritized findings, exact source
locations, counterexamples, reproduction commands, affected-output scope, required
gates, and remaining uncertainty. A separate explicit decision is required to
freeze Phase 2.6.

## Historical foundation and Phase 2.5 handoff

The challenge brief below led to the 2026-09-22 independent
[foundation and Phase 2.5 freeze review](PHASE_2_5_AND_FOUNDATION_FREEZE_REVIEW.md).
It is retained for provenance; its old phase status and test counts are not current.

## Foundation remediation handoff

Final qualification checkpoint: first read
[FINAL_FOUNDATION_QUALIFICATION_REVIEW.md](FINAL_FOUNDATION_QUALIFICATION_REVIEW.md),
the machine-readable provider catalog, and the provider records. The claimed result
is PASS for enforceable risk closure, not live-data production qualification.

Review the latest master and [foundation audit](FOUNDATION_REMEDIATION_AUDIT.md)
before interpreting the historical checkpoint below. Bitemporal membership, raw
integrity, conflict-safe feature publication, canonical macro vintages, source policy,
build manifests and disposable restore tooling now exist. The later independent
review froze foundation and Phase 2.5 semantics; external provider gates remain.

New falsification targets: version selection before effective/classification filters;
legacy unknown knowledge exclusion; delayed receipt versus certified reconstruction;
raw hash/size/path checks and same-second publication; exact feature reruns versus
conflicts; ambiguous unpinned build reads; macro initial/revision selection; source
admission coverage; and exported-snapshot dump/inventory consistency. Also try to
bypass provider capabilities with Yahoo/current FRED, obtain a survivorship label
without delisted/lifecycle coverage, admit reconstructed/missing raw lineage, use a
later SEC comparative fact too early, omit the research-quality record, or overwrite
a conflicting remote backup object.

Run `python scripts/verify_foundation_recovery.py` with native PostgreSQL clients or
`IRS_PG_TOOLS_CONTAINER=irs_postgres` for the local Docker DB. It creates/drops only
random disposable DBs. Expected schema head/current fnd006; use PROJECT_STATE for
current test counts. No end-to-end live-provider qualification, persistent-archive
restore, live off-host backup or backtest result is claimed. Legacy raw inventory
gaps are reconciled as missing test-only evidence and rejected, not hidden by the
synthetic drill.

## Goal and checkpoint

Goal: a deterministic PIT investment research system scanning thousands of companies,
detecting early growth opportunities, preserving every judgment, evaluating future
outcomes and improving versions through controlled validation rather than hindsight.

Historical implementation checkpoint: `a5a80ef8bc594500000b4f357192bc82513b5bb8` on master.
Phase 2.5 implementation audit PASS; implementation complete, ready for independent
review, NOT YET FROZEN. Strategic review verdict CONDITIONAL PASS. Phase 2.6 not
started. Documentation checkpoint can be located with:

```bash
git log -1 --format='%H' --grep='^Document closed-loop research and validation architecture$'
```

## Read and inspect

Read README and PROJECT_STATE, then:

- [Blueprint](INVESTMENT_RESEARCH_SYSTEM_BLUEPRINT.md) and [learning roadmap](LEARNING_ROADMAP.md).
- [Market contract](phase-2.4-market-features-spec.md), [estimate contract](phase-2.5-estimates-spec.md), [implementation audit](phase-2.5-implementation-audit.md).
- [Strategic review](STRATEGIC_ARCHITECTURE_REVIEW.md), [source trust](DATA_SOURCE_TRUST_AND_PIT.md), [provider register](DATA_PROVIDER_RISK_REGISTER.md).
- [Ledger](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md), [validation/calibration](VALIDATION_CALIBRATION_AND_LEARNING.md), [historical plan](HISTORICAL_RESEARCH_VALIDATION_PLAN.md).
- [Valuation/quality comparison](RESEARCH_PLATFORM_COMPARISON.md), [events](NEWS_AND_EVENT_INTELLIGENCE_PLAN.md), [durability](DATA_DURABILITY_AND_RECOVERY.md).

Important source paths below are relative to `src/investment_research/`:

| Area | Files |
| --- | --- |
| Canonical state | `models/research.py`, `financial_fact.py`, `filing.py`, `market_price.py`, `economic_data.py`, `estimates.py`, `audit.py` |
| Fundamentals | `normalization/sec.py`, `research/resolver.py`, `research/runner.py`, `research/calculators.py` |
| Market | `connectors/market_data/yahoo.py`, `normalization/market_price.py`, `research/adjustment_engine.py`, `research/market_runner.py`, `research/benchmark_resolver.py`, `research/trading_calendar.py` |
| Macro/news | `connectors/fred/connector.py`, `normalization/fred.py`, `normalization/news.py`, `connectors/news/rss.py` |
| Estimates | `connectors/estimates/base.py`, `normalization/estimates.py`, `ingestion/estimate_runner.py`, all `research/estimates/` modules |
| Raw durability | `storage/raw_storage.py`, `storage/hashing.py` |
| Persistence/tests | Repository `database/migrations/versions/`, `tests/estimates/`, `tests/unit/test_research_foundation.py`, fundamental/market unit suites, `tests/integration/test_nvda_features.py` |

## Verification

Use [machine setup](BUILD_MACHINE_SETUP.md) for local PostgreSQL/configuration.
Do not disclose `.env` or credentials. Inspect test fixtures before running against
any database; use a dedicated test environment, never an unreviewed production URL.

```bash
cd ~/Document/investment-research-system-master
git status --short
git rev-parse HEAD
git rev-parse origin/master
source .venv/bin/activate
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
python -m pytest -q
python -m pytest -q tests/estimates
alembic current
alembic heads
git diff --check
```

Expected current baseline: full 431, failures 0, 8 existing warnings; current/head
fnd006. Historical Phase 2.5 checkpoint counts below remain commit-specific. For migration lifecycle
review, inspect `scripts/verify_estimate_migrations.py` first; it creates/drops a
disposable database and requires suitable privileges. Do not downgrade the research DB.

## Falsification questions

1. Where can future information leak through values, metadata, universe selection,
   labels, imputation, provider corrections or AI pretrained knowledge?
2. Are any providers unsuitable for PIT reconstruction despite historical endpoints?
   Does Yahoo quote.close actually satisfy the raw-as-traded contract? Are FRED
   vintages preserved, or merely latest history plus realtime fields?
3. Are fiscal periods reconstructed safely for comparative facts, restatements,
   non-calendar years, Q1 ambiguity, missing quarters and calendar-regime changes?
4. Can historical-universe membership introduce survivorship bias? Effective dates
   exist; where is knowledge time? Are ticker reuse and failed/delisted names covered?
5. Are corporate actions safe across multiple splits/dividends and revisions? Are
   dividends on the same share basis as adjusted prices? Are terminal outcomes modeled?
6. Are estimate revisions historically reconstructable with a real vendor? Verify
   contributor identity, corrections versus economic activity, same absolute target,
   horizon rollover, basis compatibility, gaps and publication completeness.
7. Are backtest labels isolated from feature data? Is label maturity respected at
   training time? Can an outcome correction ever change a frozen prediction?
8. Can calibration overfit through overlapping labels, repeated trials, reused
   holdouts, sector selection or tuning to famous winners?
9. Are data licenses compatible with retention, backups, derived outputs, AI use
   and sharing? Unknown is not permission.
10. Does the planned ledger preserve exact evidence despite legacy FeatureValue
    updates? Are all abstentions/exclusions recorded?
11. Can raw paths collide outside estimates? Can a restore reproduce raw hashes,
    build identities, triggers, results and sealed research judgments?
12. Can the current replay scale to thousands of securities and long event history?
    Measure throughput; an indexed lookup alone proves little about full replay.
13. Does runtime provider qualification exactly match the versioned catalog, or can
    prose, a brand name, a version mismatch, missing coverage or a bare override pass?
14. Can current-survivor-only data obtain `SURVIVORSHIP_QUALIFIED` without historical
    universe/security coverage, actions, delistings and lifecycle evidence?
15. Does the SEC comparative fixture preserve the original prior-period value before
    the later filing and select amendments/restatements only after availability?
16. Does ordinary FRED remain blocked while mocked ALFRED vintages select GDP 2.0
    then 2.5? Challenge date-only timing and all live coverage claims.
17. Can a `RECONSTRUCTED_COPY` rewrite or certify an `ORIGINAL_ARCHIVE`, or can
    MISSING/HASH_MISMATCH/UNVERIFIABLE legacy evidence pass raw reproducibility?
18. Does off-host durability cover both PostgreSQL and raw bytes, refuse remote
    conflicts, require protection, verify integrity, and publish completion last?
19. Does every admitted or overridden mixed-source run state its exact quality level
    and per-provider limitations in durable provenance?

## Known concerns to challenge, not merely repeat

Strategic R1–R9 gates cover membership knowledge, filing midnight, comparative fiscal
identity, Yahoo/action basis, macro vintages, mutable legacy outputs, generic raw
collisions, production estimates and unproven recovery. Also inspect mean-log-return
industry benchmarks versus portfolio returns, market window spec/code discrepancies,
benchmark inception and timestamp conventions. Estimate provenance references in
JSON lack feature-to-evidence FKs; independent review must decide acceptability
against the specification. Consensus must be published through T before feature reads.

Request a verdict (PASS / CONDITIONAL PASS / FAIL), prioritized findings with source
locations and counterexamples, reproduction commands, scope of affected outputs,
required gates and remaining uncertainty. Do not weaken tests, invent provider
capabilities, or declare frozen status by implication. A separate explicit independent
review decision is needed before Phase 2.5 freeze.

## Qualitative / alternative extension review

Read [detailed architecture](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md),
[evidence hierarchy](RESEARCH_EVIDENCE_HIERARCHY.md), [alternative roadmap](ALTERNATIVE_DATA_ROADMAP.md)
and the extended prediction ledger. Locate the documentation checkpoint with:

```bash
git log -1 --format='%H' --grep='^Document qualitative and alternative research architecture$'
```

Try to falsify these additional claims:

1. Does the architecture distinguish evidence from opinion, forecast and scenario?
2. Can every qualitative assertion be traced to its precise source version and scope?
3. Does professional research leak hindsight through updated reports, current fair
   values, later extraction or an LLM's pretrained knowledge?
4. Are alternative-data timestamps actually PIT, including panel/mapping revisions,
   deleted observations, release delays and normalized historical series?
5. Can company claims be independently corroborated, or are all sources repeating
   the same management announcement? Is common-origin dependence recorded?
6. Could future moat assessments be audited from evidence, counter-evidence, rubric
   and historical versions? Does UNKNOWN remain distinct from stable/no moat?
7. Could persuasive LLM synthesis overpower deterministic evidence or hide disagreement?
8. Are professional, internal, consensus and market views compared on the same
   claim, basis and horizon? Can NOT_COMPARABLE prevent false conflicts?
9. Do rights distinguish reading from extraction, AI processing, training and sharing?
10. Do outcome labels test each claim/risk without changing its original threshold
    or treating the same external opinion as both predictor and ground truth?

No production qualitative ratings or alternative feeds exist. This extension does
not close R1–R9 or independently freeze Phase 2.5. Phase 2.6 remains NOT STARTED.
