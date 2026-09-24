# Phase 2.8 Independent Freeze Review

Review date: 2026-09-23

Implementation commit reviewed: `9e40f75c83c5179d919a10315c322487a85c01e8`

Branch: `master`

Migration head: `p28002`

## Verdict

**PASS — FREEZE APPROVED**

Phase 2.8 is frozen for signal and detector definitions, typed-rule semantics,
thresholds, deterministic proof selection, evidence strength, severity,
missingness, quality propagation, PIT/build pinning, storage, provenance,
immutability, runner behavior, and scale architecture. Phase 2.9 has not started
and is ready only for design.

This is a semantic and architectural freeze. It does not claim that a signal is
predictive, produces alpha, is a calibrated probability, or is an investment
recommendation. Phase 2.9 must begin the historical prediction-replay,
outcome-ledger, and backtesting layer that can test those separate questions.

## Scope and invariants

- The code-owned registry contains exactly nine versioned signals:
  `fundamental_acceleration_v1`, `margin_inflection_v1`,
  `estimate_confirmation_v1`, `market_confirmation_v1`,
  `industry_tailwind_v1`, `industry_headwind_v1`,
  `cash_flow_contradiction_v1`, `fragility_warning_v1`, and
  `early_growth_candidate_v1`. There is no omnibus score, rank, recommendation,
  portfolio output, or hidden prediction layer.
- The detector registry is exactly `factor_pattern_detector_v1` for the first
  eight atomic/context definitions and `early_growth_detector_v1` for the one
  compound candidate. Detectors interpret persisted factors; they do not
  recalculate source features or factor economics.
- The executable rule surface is closed to typed `Comparison`, `ALL`, and
  `AT_LEAST_N` nodes. Approved `ANY` behavior is represented by
  `AT_LEAST_N(1)`. There is no parser, `eval`, arbitrary expression, dynamic
  Python, or runtime code execution path.
- Strong positive/negative thresholds are exactly `+0.50`/`-0.50`; ordinary
  positive/negative thresholds are exactly `+0.10`/`-0.10`. Inclusive Decimal
  comparisons are covered one representable Decimal unit below, at, and above
  each applicable boundary.
- Early Growth is exactly strong Growth and strong Growth Acceleration plus one
  independent Expectations, Market, Industry, or Fundamental Economics family.
  Profitability, Margin Expansion, and Cash Flow Quality are nested inside the
  single Fundamental Economics family and cannot count as independent families.
- Industry Tailwind and Headwind are node-scoped, use opposite inclusive strong
  thresholds, and cannot both fire for one valid value. Market Fragility retains
  `+1 = more fragile`. Cash Flow Contradiction remains strong Growth plus weak
  Cash Flow Quality; it does not cancel the Growth evidence.
- Missing Estimate Revision remains explicitly unavailable and may be bypassed
  by another passing independent family. `VALID/fired=false`, `MISSING`, and
  `INVALID` remain distinct. Warnings, contradictions, and headwinds can coexist
  with an independently proven opportunity and never act as hidden vetoes.
- Passing `AT_LEAST_N` proofs are selected by descending strength and then
  canonical child ordinal. Quality follows only the selected recursive proof;
  unused low-quality alternatives and diagnostics remain visible without
  contaminating result quality.
- Evidence strength is exact rule-margin magnitude in `[0,1]`: comparison
  margin at a threshold is zero, `ALL` takes the minimum, and `AT_LEAST_N` takes
  the kth-largest passing child. Severity uses exact thirds: `LOW` below `1/3`,
  `MEDIUM` from `1/3` below `2/3`, and `HIGH` from `2/3` through `1`.
- Every run pins the exact signal definitions, detector/runner/policy versions,
  signal build, upstream factor build and hashes, T, effective date, origin, and
  exact historical industry mapping. Factor resolution has no global-latest or
  future-value fallback.

## Adversarial findings and fixes

No economic threshold, grouping, proof policy, quality policy, definition hash,
factor formula, or migration changed. The review reproduced and corrected two
fail-closed request-boundary defects:

1. Canonical manifests could contain wrongly typed collection fields beyond the
   two fields already checked. Unhashable nested `signal_admission` quality or
   `snapshot_origin` values could also leak raw `TypeError`. The runner now
   validates all seven required manifest mapping fields and validates those
   nested scalar types before membership checks. Signal and upstream variants
   fail with controlled `BUILD_MANIFEST_INVALID` or `QUALITY_NOT_ADMITTED` and
   publish no result.
2. A non-iterable registry request or a collection containing non-text or
   unhashable keys could leak `TypeError`. Registry requests now require an
   iterable of non-empty strings before duplicate/unknown-key evaluation and
   fail with `SIGNAL_REQUEST_INVALID`.

Eighteen new regression cases cover these paths. A further required-scope probe
confirmed the existing `SIGNAL_SUBJECT_INVALID` behavior without a code change.

## Provenance, storage, and immutability

The four authoritative tables remain `research_signals`,
`research_signal_conditions`, `signal_results`, and
`signal_result_conditions`. Every declared leaf is persisted, including failed,
missing, invalid, supplemental, contradictory, and warning evidence. Condition
rows retain definition/factor identities, exact FactorValue ID when present,
actual status/value/reason/quality/coverage, operator/threshold, outcome,
selection state, and proof provenance, so both firing and non-firing decisions
are reconstructible.

Live-schema inspection confirmed the definition, subject/T/build, fingerprint,
lookup, and forward-lineage indexes and all four immutable-table triggers. Result
and condition guards reject scope, definition, factor-build, factor-family,
subject, timestamp, and industry-context mismatches. The supported publication
path inserts and verifies the complete parent/condition graph in one savepoint;
injected failures roll back every row, exact replay reuses the graph, changed
content fails closed, and two real PostgreSQL publishers converge on one complete
graph.

Definition fingerprints cover signal/detector versions, policy versions, rule
shape, factors, operators, and thresholds. Result and condition fingerprints add
the exact factor artifact/value/quality, evaluation, selected proof, subject,
builds, admission, and provenance. Direct mutation probes confirmed that a
threshold, detector version, or signal version changes the definition hash;
runner tests cover factor/proof/quality content conflicts and isolated new builds.

## Runner, PIT, and frozen-phase regression

`SignalRunner` queries immutable `FactorValue` artifacts only. It neither imports
nor invokes fundamental, market, estimate, peer, feature, or factor calculators.
The full synthetic T1/T2 lifecycle uses production feature, peer, factor, and
signal runners and persisted FactorValues; it manually injects no SignalResult.
T2 creates new pinned artifacts and an exact T1 replay retains the original IDs,
fingerprints, and lineage.

The diff from the approved Phase 2.7 checkpoint changes no Phase 2.2, 2.4, 2.5,
2.6, or 2.7 formula, registry, normalization, calculator, or runner economics.
The full repository regression independently exercises those frozen phases.

## Migration and indexes

The disposable verifier passed fresh base → `p28002` → `p27001` → `p28002`,
four-table presence/absence, ORM/Alembic metadata parity, immutability and FK
guards, all signal tests, and real concurrent publication. Persistent
`alembic current`, `heads`, and `check` pass at the single head `p28002`.

Existing indexes support signal definition/version lookup, security and node
result lookup by T/build, idempotent fingerprints, exact result identity, and
condition lineage. No observed lookup or scale plan justifies another index or a
historical migration edit.

## Scale assessment

The reviewed production runner has this disposable PostgreSQL scale evidence:

| Securities | Factor artifacts | Signal results | Condition rows | SQL statements | RSS growth | Temp I/O | Wall time |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1,000 | 8,001 | 7,002 | 22,002 | 220 | 111.871 MiB | 0 | 31.676 s |
| 5,000 | 40,001 | 35,002 | 110,002 | 364 | 185.055 MiB | 0 | 154.791 s |
| 10,000 | 80,001 | 70,002 | 220,002 | 544 | 275.438 MiB | 0 | 313.169 s |

After fixed setup, query count grows by 36 statements per 1,000 securities,
matching four bounded 250-security batches rather than per-security work. The
single node results are not multiplied across the universe, plans use no temp
I/O, and memory growth is bounded by the batched architecture. The review fixes
only constant-size request validation before factor resolution, so rerunning the
expensive scale proof would not add performance evidence.

## Verification

- `python -m pytest -q tests/signals` — **168 passed**.
- Full repository suite — **905 passed, 8 existing warnings, 0 failures**.
- Early Growth adversarial lattice — **4,160 cases passed**.
- Disposable migration/concurrency verifier — **PASS**.
- `alembic current`, `alembic heads`, `alembic check` — **PASS at `p28002`**.
- `git diff --check` — **PASS**.

## Test-quality assessment

Database tests use rollback-isolated sessions; the concurrency and migration
proofs create and force-drop only random disposable databases. The lattice has an
independent expected-value oracle over production evaluator types, while the T1/T2
lifecycle and runner tests exercise real persistence. Failure injection verifies
atomic rollback. No production rule implementation is mocked into the expected
answers, and no shared committed database state or order dependency was found.

## External limitations

Signals remain uncalibrated `CANDIDATE` interpretations. Production-provider
coverage is incomplete; synthetic estimates remain development quality; outcome
labels, historical predictive validation, ranking, portfolio construction, and
live investment recommendations do not exist. These are explicit future gates,
not reasons to weaken or postpone the semantic freeze.

## Freeze decision

The exact Phase 2.8 v1 definitions and semantics may now be depended on by later
phases. Any change to a threshold, rule grouping, factor family, detector behavior,
proof ordering, strength/severity arithmetic, quality/missingness policy, or PIT
identity requires a new version rather than mutation in place.

**Phase 2.8: FROZEN**

**Phase 2.9: READY TO DESIGN**
