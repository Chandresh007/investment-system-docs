# Phase 2.8 signal/detector implementation audit

Audit date: 2026-09-23. Starting checkpoint:
`8a859963c2eef80b1a0b4bb75b6d678b419b2939` on `master`, equal to
`origin/master` with a clean worktree after checkpoint 2.8.8.

Result: **PASS after the two corrections below**. Phase 2.8 implementation is
complete and ready for independent review. It is **not yet frozen**, empirically
validated, calibrated, promoted, or production-provider-qualified. Phase 2.9 has
not started. Foundation and Phases 2.2, 2.4, 2.5, 2.6, and 2.7 remain frozen.

This audit treated the
[Phase 2.8 implementation contract](phase-2.8-signal-detector-spec.md) and
[architecture review](phase-2.8-signal-detector-architecture-review.md) as the
authority rather than treating existing code or tests as the specification.

## Audit verdict by area

| Area | Result and concrete evidence |
| --- | --- |
| Scope | The immutable registry contains exactly nine v1 candidate signals, 24 ordered condition leaves, and two detector identities. `factor_pattern_detector_v1` emits exactly the first eight definitions by matching subject scope; `early_growth_detector_v1` emits only `early_growth_candidate_v1`. There is no buy/sell output, overall score, rank, probability, expected return, portfolio weight, valuation/news/AI signal, prediction/outcome ledger, backtester, or Phase 2.9 implementation. |
| Frozen upstream boundary | Phase 2.8 imports factor definitions, exact immutable `FactorValue` artifacts, Decimal context, and the frozen weakest-quality ordering. It does not invoke `FactorRunner`, read a `FeatureValue`, or reproduce a Phase 2.2/2.4/2.5/2.6/2.7 formula. The diff from approved checkpoint `84066fa...` changes no frozen formula, calculator, feature runner, factor calculator, factor registry, normalization, peer runner, or estimate/market implementation file. |
| Definitions | Every definition is `CANDIDATE`, version 1, code-owned, canonical, content-hashed, and relationally mirrored. The exact classes/scopes/detectors, factor IDs/versions/hashes, operators, thresholds, grouping, diagnostics, required flags, and order match the frozen contract. Registration is exact-idempotent; changed content conflicts, while an explicit v2 gets a separate identity. |
| Rule engine | The closed tree supports only `FactorCondition(GTE/LTE)`, ordered `ALL`, and ordered `AT_LEAST(k)`, with diagnostic roles outside the decision tree. There is no parser, expression language, `eval`, dynamic Python, arbitrary SQL rule, or user-supplied code. Inclusive comparisons use exact finite Decimal values. |
| Atomic signals | Fundamental Acceleration requires Growth and Acceleration `>= +0.50`; Margin Inflection requires Margin Expansion `>= +0.50`; Estimate and Market Confirmation use `>= +0.50`; node-scoped Tailwind/Headwind use `>= +0.50` / `<= -0.50`; Cash Flow Contradiction requires Growth `>= +0.50` with Cash Flow Quality `<= -0.10`; Fragility Warning uses Market Fragility `>= +0.50`, retaining `+1 = more fragile`. Exact one-unit neighbors pass their boundary tests, and Tailwind/Headwind cannot both fire from one exact node factor. |
| Early Growth | Growth and Acceleration must both be `>= +0.50`, plus one independent Expectations, Market, Industry, or Fundamental Economics family. The Economics family is itself one-of Profitability, Margin Expansion, or Cash Flow Quality `>= +0.10`, so multiple members still count as one family. Missing estimates can remain unavailable while another proof fires. No warning or contradiction is an economic veto. |
| Falsification | An audit-only oracle enumerated all 4,096 pass/fail/missing/invalid combinations across the two required core leaves and four confirmation families, plus all 64 nested Economics-member combinations. All 4,160 cases matched frozen `ALL`/`AT_LEAST(1)` determinacy. An additional 18 exact Decimal edge cases covered below/at/above each Early Growth confirmation threshold. One combined case fired all five diagnostics—margin deterioration, cash weakness, weak profitability, industry headwind, and fragility—while Early Growth still fired through Market proof. |
| Missing and invalid semantics | `VALID/fired=false` remains distinct from `MISSING` and `INVALID`; only valid results carry a Boolean fired state. `ALL` cannot claim a partial non-fire when a required child is unavailable. `AT_LEAST(k)` is determinate when enough valid passes exist, and unavailable only when missing/invalid alternatives could change the decision. Integrity/invalid, definition/subject/timestamp, build, quality, industry-context, then generic reason precedence now matches the contract. |
| Strength and severity | A passed `GTE` leaf uses `(x-theta)/(1-theta)` and a passed `LTE` leaf uses `(theta-x)/(theta+1)`. `ALL` uses its weakest required margin; `AT_LEAST(k)` uses the kth-strongest selected witness. Only fired valid results receive `[0,1]` evidence strength. Exact threshold has strength zero/`LOW`; exact rational thirds select `LOW`, `MEDIUM`, and `HIGH` as frozen. The field is consistently called evidence strength, never probability. |
| Proof and quality | Passing `ALL` selects every required child; `AT_LEAST(k)` selects the strongest k branches and breaks exact ties by canonical child ordinal, including nested Economics ties. Valid non-fires retain a deterministic failure proof. Signal quality is the weakest signal admission and distinct factor evidence in the selected proof only. Selected synthetic estimates propagate `DEVELOPMENT`; an unselected development estimate or diagnostic cannot contaminate a qualified proof. |
| Coverage | Coverage counts distinct declared/admitted/missing/invalid factors, preserves every condition, and stores minimum selected upstream weight coverage. A determinate result is `FULL` only when all declared factors are admitted and every selected factor is upstream `FULL`; non-decisive missing/invalid evidence or selected reduced evidence yields `REDUCED`. Unavailable results are `NOT_APPLICABLE`. |
| PIT and build pinning | The runner validates the canonical signal manifest, exact registry/hashes/versions, both detector versions, runner and five policy versions, T/effective date/origin, signal admission, context mapping policy, exact upstream factor manifest hash, factor runner, and factor hashes. Retrieval filters exact definition, subject, T, effective date, origin, and pinned factor build; future/wrong-build evidence is never substituted. Security use of Industry Strength requires its exact included historical group membership. |
| Persistence | Dedicated `research_signals`, `research_signal_conditions`, `signal_results`, and `signal_result_conditions` tables are authoritative; there is no `DetectorResult`. Migration `p28001` supplies definitions/results/lineage, and `p28002` permits deterministic failed-condition proof witnesses and requires exact included membership for security use of node evidence. Checks and insert guards enforce scope, manifest/T/build, factor-definition/value identity, finite domains, status/fired/strength/quality combinations, snapshots, relations, and exact operator/threshold copies. |
| Provenance | Every result pins signal/detector/runner/policy identities, both build IDs, subject and exact industry mapping, T/effective date/origin, all definition-condition and factor-value IDs, the recursive decision tree, selected proof, result/status/reason, strength/severity, quality, coverage, admission, and fingerprint. Every declared leaf has a relational condition row retaining pass/fail/unavailable relation and exact factor snapshot or absence reason. |
| Immutability/idempotency | PostgreSQL triggers reject update/delete of all four signal tables. Exact replay keeps result/condition IDs, fingerprints, values, reasons, and calculation times. Changed stored content fails closed; signal/factor build changes create separate artifacts. Parent and all condition rows publish inside one savepoint, later-batch failure rolls back earlier batches, and a real two-writer test converges on one complete graph. |
| Lifecycle | The full Semiconductor Infrastructure T1/T2 test publishes real Phase 2.6 peer/industry artifacts, runs the production Phase 2.7 factor runner for all nine factors, then runs the production signal runner. T1 fires Early Growth and Fragility together; T2 transitions opportunities to valid non-fires while Fragility persists. T2 cannot mutate T1, and T1 replay preserves all identities. No final signal row is manually injected. |
| Scale | The accepted 10,000-security proof publishes 70,002 results and 220,002 relational conditions from 80,001 factor artifacts in 544 SQL statements and 313.169 seconds on the 2-vCPU development host. Signal-run RSS growth is 275.438 MiB after bounded payload deferral; PostgreSQL temp-file/byte deltas and captured-plan temp blocks are zero. SQL grows by 36 statements per 1,000 securities, matching 250-security batches rather than an N+1 pattern. |
| Migrations and regression | The disposable verifier passes fresh base → `p28002` → `p27001` → `p28002`, all four tables, `alembic check` metadata parity, and 150 signal tests on fresh PostgreSQL. Persistent current/head is `p28002`. Focused frozen-factor plus signal/lifecycle regression is 367 passed; the complete repository suite is 887 passed with the same 8 existing warnings and zero failures. |

## Exact v1 registry

| Registry key | Scope / class | Detector | Conditions | Definition hash |
| --- | --- | --- | ---: | --- |
| `fundamental_acceleration_v1` | Security / Opportunity | `factor_pattern_detector_v1` | 2 | `19ca3c1a52889dc0c2e583231977b2c9db591eeb304109e5f77ba885da37eaf2` |
| `margin_inflection_v1` | Security / Opportunity | `factor_pattern_detector_v1` | 2 | `1e014f08118ff39aae2092f2fd619bf513763edc40d3d71e0479d7c601f867af` |
| `estimate_confirmation_v1` | Security / Confirmation | `factor_pattern_detector_v1` | 1 | `756f39069c3495bcbe877a897e17d2d84c0dab3d2109e48887c79443481135b7` |
| `market_confirmation_v1` | Security / Confirmation | `factor_pattern_detector_v1` | 1 | `68803f17a8a4db3994d4fd4baf791ee0e7582967d81bb1a056ade6d165b9cc85` |
| `industry_tailwind_v1` | Industry node / Context | `factor_pattern_detector_v1` | 1 | `d0ae18c295a88b3ee6d25a852869b2bc8a7fca350131fc931fc8938390048171` |
| `industry_headwind_v1` | Industry node / Context | `factor_pattern_detector_v1` | 1 | `117407a382b8ecc260afedc6c30825032ca3cbd6a62ff5adb2cbc3fc802d32f4` |
| `cash_flow_contradiction_v1` | Security / Contradiction | `factor_pattern_detector_v1` | 2 | `7706a7e2812d75c88226dfb0d8b50aeead0c3477d9a3ad51844a75dcf3ee7b46` |
| `fragility_warning_v1` | Security / Warning | `factor_pattern_detector_v1` | 1 | `00d9d504b9b3788fbea915278781c509b7020e1a89e701407f4420217a3d3b2f` |
| `early_growth_candidate_v1` | Security / Opportunity | `early_growth_detector_v1` | 13 | `6dfdee3545c2ffbd1b8c05238c140daff6a050b953f9d25caf9d958556384a2d` |

The 24 conditions comprise 18 decision comparisons and 6 diagnostics. The
canonical JSON trees, not this table, remain authoritative for nested grouping.

## Corrections made during this audit

### 1. Specific unavailable-confirmation reasons were hidden by a generic reason

The Early Growth independent-confirmation group correctly became `MISSING` when
no family passed and one potentially decisive family was unavailable. However,
it always emitted `INSUFFICIENT_CONFIRMATION_COVERAGE`, even when the unavailable
condition's exact reason was a higher-precedence `FACTOR_TIMESTAMP_MISMATCH`,
`BUILD_MISMATCH`, `QUALITY_NOT_ADMITTED`, or `INDUSTRY_CONTEXT_MISSING`.

The defect was reproduced first in the side-effect-free evaluator and then with
the production runner by placing Estimate Revision only in a non-pinned factor
build while all admitted confirmations failed. Before correction the persisted
result hid the build mismatch behind generic coverage.

The evaluator now uses the generic reason only for ordinary missing alternatives.
It preserves the frozen higher-precedence specific reason while every condition
still retains its own reason. A valid alternate proof continues to make the
result `VALID/REDUCED`, so this correction does not turn warnings or optional
absence into vetoes. No threshold, grouping, proof selection, factor formula,
quality policy, or definition hash changed.

### 2. Malformed manifest collection types escaped the controlled error boundary

A canonical `ResearchBuildManifest` can contain a syntactically valid but wrongly
typed nested value. Supplying a list for `calculation_versions` or
`configuration` caused the runner to call `.get()` and leak `AttributeError`
instead of producing the documented request-level build error.

The runner now validates both nested objects as mappings before reading signal or
factor pins. Parameterized tests cover the signal manifest and persisted upstream
factor manifest for both malformed fields. Every case rolls back with
`BUILD_MANIFEST_INVALID` and publishes no result. This is a fail-closed API/error
boundary correction; it changes no accepted manifest or signal semantics.

## Verification

- `python -m pytest -q tests/signals` — **150 passed**, zero failures.
- `python -m pytest -q tests/factors tests/signals
  tests/peers/test_signal_lifecycle.py` — **367 passed**, zero failures.
- `python -m pytest -q` — **887 passed, 8 existing warnings**, zero failures.
- `python scripts/verify_signal_migrations.py` — PASS on a fresh disposable
  PostgreSQL database: base/head lifecycle, `p28002` downgrade/upgrade boundary,
  four tables, metadata parity, and all 150 signal tests.
- Persistent `alembic current`, `alembic heads`, and `alembic check` identify
  **`p28002`** with no pending model operation.
- Checkpoint 2.8.7 lifecycle — exact T1/T2 production-factor artifacts, nine
  results and 24 conditions per T, warning/opportunity coexistence, transition,
  PIT/build isolation, and immutable T1 replay.
- Checkpoint 2.8.8 scale proof — exact 1k/5k/10k artifact counts; 544 statements,
  313.169 seconds, 356.000 MiB process peak, 275.438 MiB run delta, and zero
  PostgreSQL temp I/O at 10k. See
  [the scale report](phase-2.8-signal-scale-sanity.md).
- `git diff --check` and the approved-base frozen-file audit pass.

## External limitations that remain

- No commercial historical estimate, market/action, classification, universe,
  identity/delisting, or broad fundamental source has completed the existing
  production qualification and licensing gates.
- Synthetic estimate/peer/industry evidence remains `DEVELOPMENT`; deterministic
  correctness is not empirical predictive validity.
- All nine signals remain candidates. There is no outcome label, prediction or
  outcome ledger, historical backtester, calibration, rank, probability,
  expected-return model, portfolio rule, buy/sell recommendation, or promotion.
- The 10,000-security result is a structural development-host sanity check, not
  a production SLA or distributed-capacity claim.
- The eight existing UTC/SQLAlchemy deprecation warnings are outside Phase 2.8.

Audit commit locator after publication:

```bash
git log -1 --format='%H' --grep='^Complete Phase 2.8 implementation audit$'
```

Next action: independent Phase 2.8 QC/architecture review. Do not freeze by
implication, promote candidate signals, weaken provider gates, or begin Phase 2.9
during that review.
