# Historical research validation plan

Status: FUTURE DESIGN. No backtester or historical performance result exists yet.
Foundation seeds now implemented: bitemporal membership, canonical macro vintages,
verified raw replay, immutable recalculation, ResearchBuildManifest and fail-closed
source qualification, raw trust states, security identity history, and research-
quality provenance. See the
[final qualification review](FINAL_FOUNDATION_QUALIFICATION_REVIEW.md).
The small deterministic feature/restore tests are mechanical reproducibility proofs,
not empirical backtests. Future runners must call manifest source admission over
all required lookback coverage before using legacy development calculators.

## Historical reality rather than hindsight

Retrospective analysis often starts with a known winner and today's revised history.
A replay starts with a decision timestamp, qualified then-available evidence and a
historical universe, without access to later outcomes. It records every decision,
then joins future outcomes through a separate evaluator.

Worked schedule (dates illustrate calendar horizons, not an executed experiment):

```text
Research date: 2021-06-30
Research cutoff: explicit UTC instant corresponding to chosen market-close policy
Inputs: only evidence with justified available_at <= cutoff
Generate and seal snapshot + prediction + eligible universe manifest
Reveal outcomes separately:
  2021-09-30 (3m)
  2021-12-31 (6m)
  2022-06-30 (12m)
  2023-06-30 (24m)
```

The actual observation/execution dates follow the versioned session policy in the
[ledger design](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md). A June 30 close cannot
be used to execute a decision requiring processing after that close. Accounting
labels may mature later than these dates. Downloading 2021 prices in 2026 does not
make them knowledge from 2021. A qualified historical archive can support a new
reconstructed build, but it cannot create a record of what this system actually saw.

## Replay contract

1. Pin code, dependencies, availability policies, data builds and calendar version.
   Invoke `require_historical_replay` for every provider. Persist the returned
   purpose, quality level, provider capabilities/coverage/evidence, raw status,
   limitations, and any named development override with the run.
2. Reconstruct eligible securities and classifications, with effective AND knowledge
   times. Include delisted/acquired/bankrupt names and historical ticker mappings.
3. Verify raw hashes and coverage manifests. Quarantine sources that cannot prove
   timestamp, adjustment or revision semantics. Missing history stays missing.
   Raw-reproducible claims require verified originals by default; reconstructed
   copies are explicit and cannot certify prior PIT content.
4. Build features only through cutoff T. Publish estimate consensus through T and
   lookback boundaries before feature calculations. Resolve the same absolute
   target for revisions. All fitted transforms come from pre-T training artifacts.
5. Freeze all predictions, exclusions, benchmark policy, thesis and expected horizon.
6. Let an isolated outcome worker read later data and produce versioned labels.
7. Compare performance, business outcomes and coverage against preregistered baselines.
8. Rerun from pinned artifacts; exact deterministic payload hashes must match.

A historical LLM may know future events from pretraining. Initially omit it from
historical signal generation entirely. If used for explanatory replay, label its
output retrospective, restrict citations to the packet and keep it outside scored
predictions. Prompt instructions alone do not erase future knowledge.

## Golden case library

| Case family | Educational question |
| --- | --- |
| Successful early compounders | Did improvement become visible before recognition? |
| Failed growth stories | Which apparent confirmations failed? |
| Value traps | Did cheapness mask deterioration? |
| Bubble stocks | Did valuation/risk contradict acceleration? |
| Cyclical peaks | Did unsustainable margins mimic durable growth? |
| Post-earnings acceleration | Did release timing and next-session execution matter? |
| Margin inflections | Did profitability improvement survive working-capital effects? |
| Estimate-revision inflections | Did contributor activity agree with consensus movement? |

Each case manifest records selection rationale, visibility cutoff, legal source
rights, original and corrected evidence, expected features/statuses, label version,
independent hand calculations, and known contamination through prior inspection.
Include winners, failures, near-misses, false positives and missing-data cases.
Famous historical stocks may explain behavior; they may NOT be the sole optimization
target. Existing NVDA and synthetic Helios tests validate mechanics, not generality.

Keep small synthetic or explicitly redistributable golden data in Git. Larger or
licensed source packets belong in controlled object storage with permitted manifests.
No real corpus is downloaded or committed by this design review.

## Generalization matrix

Predefine a broad historical universe and chronological folds before seeing labels.
Report results and uncertainty across:

- Earlier-year training and later-year validation, untouched final years, then prospective shadow runs.
- Different sectors and leave-sector-out stress tests, including AI and non-AI industries.
- Bull and bear markets; high- and low-rate regimes; recessions and recoveries.
- Small, mid and large caps; liquidity and coverage buckets; new listings and delistings.
- Company-held-out cohorts and rolling/expanding training histories.

Use retrospective regime labels only for diagnosis unless then-known evidence
supports them as features. Sparse slices are inconclusive, not a pass. A rule need
not outperform everywhere, but any claimed scope must survive unseen periods and
show honest weak regimes. Cross-company coverage changes must be reported alongside
returns; dropping difficult names can manufacture apparent generalization.

## Adversarial gates

Before performance research, demonstrate that a T+1 filing, action, membership
correction, analyst alias, macro vintage or event cannot change a sealed T snapshot.
Test exact-boundary visibility, after-hours releases, amended/comparative facts,
calendar changes, multiple splits/dividends, unknown action basis, delisting proceeds,
missing benchmark history, provider corrections and corrupted raw objects.

Test broad-universe batching and resource usage at realistic history depths;
per-security estimate prefix replay and serialized build locks are not yet scale
qualified. Prefer measured SQL/index and incremental-replay improvements over new
distributed infrastructure. Revalidate outputs against the correctness reference.

Promotion evidence must distinguish golden regression PASS, source qualification
PASS, statistical validation PASS, and production readiness PASS. None implies the others.
