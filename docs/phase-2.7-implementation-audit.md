# Phase 2.7 factor implementation audit

Audit date: 2026-09-23. Starting checkpoint:
`5c186947ed3aa8f102dfe7c38831ae3f6c98781d` on `master`, equal to
`origin/master` with a clean worktree after checkpoint 2.7.9.

Result: **PASS after the correction below**. Phase 2.7 implementation is
complete and ready for independent review. It is **not yet frozen**, empirically
validated, calibrated, promoted, or production-provider-qualified. Phase 2.8 has
not started. Foundation and Phases 2.2, 2.4, 2.5, and 2.6 remain frozen.

This audit tested the implementation against the
[frozen Phase 2.7 contract](phase-2.7-factor-spec.md) and the
[architecture review](phase-2.7-factor-architecture-review.md), rather than
treating existing code or tests as the specification.

## Audit verdict by area

| Area | Result and concrete evidence |
| --- | --- |
| Scope | The immutable registry contains exactly the nine approved v1 keys, 19 named subfactors, and 47 components. There is no peer-leadership, operating-leverage, valuation, business-quality, sentiment, omnibus, buy, rank, signal, expected-return, or probability output. Eight factors are security-scoped; Industry Strength is stored once per exact industry node/group snapshot. |
| Definition identity | Every definition starts `CANDIDATE` and stores logical ID/version, public key, scope, orientation, ordered hierarchy, source expectations, normalization/weight/coverage/formula/quality versions, canonical payload, and SHA-256 hash. Registration is idempotent only for identical content; v2 is a separate definition and now has an explicit distinct-value regression. |
| Normalization | `factor_fixed_piecewise_v1` uses finite exact Decimal arithmetic at 38-digit `ROUND_HALF_EVEN` precision. Piecewise knots, saturation, non-zero neutral points, identity bounds, drawdown domain, and `2p-1` peer/industry percentile conversion match the contract. There is no learned threshold, clipping beyond the declared transform, z-score, winsorization, or orientation-based silent inversion. |
| Hierarchy and coverage | Calculation is subfactor-first, then fixed-share factor aggregation. Required anchors are explicit. Exactly 60% subfactor and 70% factor weighted coverage pass; one decimal unit below fails. Optional missingness reweights only inside its declared subfactor. Valid reduced coverage is `REDUCED`; missing is never numeric zero. |
| Orientation | Eight definitions are `POSITIVE_EVIDENCE`. `market_fragility_v1` is `RISK_HIGHER_WORSE`; `+1` means more observed fragility. Its drawdown is negated exactly once and a positive raw maximum drawdown is invalid. |
| Source isolation | The factor package imports registries and immutable result models, not frozen fundamental/market/estimate formulas. The runner never invokes those feature calculations. It reads exact `FeatureValue`, `PeerRelativeResult`, and `IndustrySnapshotMetric` artifacts. A diff from the approved `f51f174...` base changes no Phase 2.2, 2.4, 2.5, or 2.6 formula/runner file. |
| PIT and coherence | Fundamentals use the exact calculation build, newest measured eligible period at/before T, inclusive 135-day freshness, one factor anchor period, and no fallback. Estimate, market, peer, and industry sources require exact T/period/build identities. Estimate FQ1/FY1 targets, peer target sources, and industry node/group/statistic lineage are checked. Future, wrong-build, wrong-period, stale, or inadmissible rows remain explicit exclusions. |
| Build pinning | One canonical content-addressed `ResearchBuildManifest` pins T, schema/Git/dependencies/raw inventory, exact datasets and source calculation versions, source/factor admissions, all requested definition hashes and weight versions, normalization/coverage/formula/quality/runner versions, effective date, and snapshot origin. Factor values FK directly to that manifest. |
| Quality | Valid factor quality is the weakest factor-specific admission and every included input/group/statistic admission. Synthetic/limited/override evidence forces `DEVELOPMENT`. Excluded optional evidence affects coverage but not quality. Estimate, market, peer, and industry downgrades remain isolated to factors that consume them. No factor upgrades an input. |
| Storage | Seven dedicated tables at migration `p27001` store manifests, definitions, definition components, the future append-only promotion hook, factor values, subfactor values, and relational component decisions. Factors are not `ResearchFeature`/`FeatureValue` outputs. PostgreSQL constraints and insert guards enforce scope, definition count/weight, manifest timestamp, finite bounded values, status/value/coverage combinations, source identity, and peer/industry relationships. |
| Immutability | PostgreSQL triggers reject update/delete for manifests, definitions, definition components, promotion records, factors, subfactors, and component lineage. Source `FeatureValue` immutability remains unchanged. Exact reruns retain IDs, fingerprints, values, reasons, and execution timestamps; changed content fails closed. |
| Provenance | Every expected component has a relational included/excluded row. Included direct/peer/industry evidence carries exact source FKs, raw/normalized values, fixed/effective weights, contribution, quality/admission, period/availability/build, and selection evidence. Peer components also link their authoritative result; industry components link the exact metric/statistic/group. Subfactors remain independently queryable. |
| Atomicity and concurrency | One outer savepoint encloses all internal resolution/publication batches. A disposable two-transaction proof holds the first complete factor graph uncommitted, confirms an independent reader sees zero factor/subfactor/component rows, then proves both writers return the same immutable identity and one complete 1/1/3 graph remains. |
| Batching and scale | Source/lineage graphs are bounded to 250 security subjects, inserts to 2,000 rows, and industry publication to one node result. The accepted 10,000-security proof produced exact expected cardinalities in 667 statements with a 549.125 MiB Python peak. There is no security/component N+1 loop or industry Cartesian multiplication. |
| Migrations | The disposable verifier passes fresh base → `p27001` → `p26008` → `p27001`, all seven factor tables, `alembic check` metadata parity, 209 factor tests, immutability/FK checks, and concurrent one-winner publication. Persistent current/head remains `p27001`; no pushed migration was rewritten. |

## Exact v1 registry

| Registry key | Scope / orientation | Subfactors / components | Required anchors | Definition hash |
| --- | --- | ---: | --- | --- |
| `growth_v1` | Security / positive | 2 / 4 | revenue growth; peer revenue-growth percentile | `95de44b31bb3f0896888e67fc89d513c3c005d5f40d9bd8e30e0977db7305941` |
| `growth_acceleration_v1` | Security / positive | 2 / 3 | revenue acceleration; peer revenue-acceleration percentile | `f9a7c563cee357fb53041dbc09b9e2e57a45b809938957e11c4cee1bee9c2de1` |
| `profitability_v1` | Security / positive | 2 / 4 | operating margin; peer operating-margin percentile | `05aa757adf1311bde787d90e95281a8cc8883b3b6730e963b1c5b988576d3cb0` |
| `margin_expansion_v1` | Security / positive | 2 / 3 | operating-margin change; peer operating-margin-change percentile | `edff3b9c30d473a7b25a3d640d05043d63d29f06439f81e56e07baddfc130f39` |
| `cash_flow_quality_v1` | Security / positive | 2 / 5 | FCF margin; FCF conversion; peer FCF-margin percentile | `dd891e8be8360aa09ac1d15fe3ca6ecf8fe94cc567cfd3cfbfebdf6125dd15af` |
| `estimate_revision_v1` | Security / positive | 2 / 9 | FQ1 EPS/revenue magnitude; both peer magnitude percentiles | `ad966b08f2e161fd819e4646db3c33fcd9b49ce1cbe253caefe2143ba21ce008` |
| `market_leadership_v1` | Security / positive | 3 / 8 | price/SMA200; market-relative return; peer return percentile; 126-day return | `5bac2cf0ca678caa7e6ea95ade90c663ae6005bd8912550588c4d76842c37a7f` |
| `industry_strength_v1` | Industry node / positive | 3 / 8 | acceleration; EPS median/breadth; return median/breadth | `c6dc0e38d97336625894a8ea987760551438563c3f47af3a9d65f0e703d1d88a` |
| `market_fragility_v1` | Security / higher risk worse | 1 / 3 | 63-day realized volatility | `a97aaab20a01f15d1a1eb44b21536607a9ef3154fc8fb865ed5543c8cdbc5470` |

The component keys, weights, knots, directions, source versions/units/frequencies,
and required flags are asserted exhaustively in definition tests; the authoritative
human-readable table remains in the frozen contract.

## Correction made during this audit

### Deferred payloads were not independently compared on large reruns

Checkpoint 2.7.9 deferred `FactorValue.admission` and `FactorValue.provenance` for
large requests to keep 80,001 returned ORM rows inside the development host's
memory envelope. Scalar fields and the factor fingerprint were compared, but the
two deferred JSON objects were trusted through that fingerprint rather than read
and compared independently.

The defect was reproduced by publishing a complete valid factor graph, explicitly
disabling the database immutability trigger inside a rollback-only test, changing
only persisted provenance while leaving the original fingerprint, re-enabling the
trigger, and forcing a multi-batch replay. The pre-fix runner incorrectly accepted
the inconsistent row.

The accepted correction keeps returned JSON fields deferred and:

1. compares every scalar field and fingerprint as before;
2. recognizes rows inserted from the exact expected payload in the current
   statement, avoiding redundant JSON readback;
3. for a pre-existing or concurrently won PostgreSQL row, sends the expected
   batch as one JSONB recordset and lets PostgreSQL compare admission/provenance
   with `IS DISTINCT FROM`; and
4. retains a repository-compatible direct comparison fallback for other dialects.

The tamper regression now fails closed with
`PUBLISHED_FACTOR_CONFLICT_USE_NEW_BUILD_OR_VERSION`. A forced two-batch test
asserts zero JSONB checks for fresh inserts, one check per existing replay batch,
identical result IDs, and still-deferred returned payloads.

Two candidate fixes were rejected: eager cast projections and streamed Python
readback both accumulated too much memory during a 10,000-security run. Every
random database left by an OS kill was identified by its exact
`irs_factor_scale_*` name and force-dropped; no configured research database was
altered. The database-side equality design passed a maximum internal-batch replay:
250 securities plus one industry node, all nine definitions, 2,001 factor values,
one payload-equality statement, and identical IDs/fingerprints/execution times.

## Verification

- `python -m pytest -q tests/factors tests/peers/test_factor_runner_peer.py
  tests/peers/test_factor_runner_industry.py tests/peers/test_factor_lifecycle.py`
  — **212 passed**, zero failures.
- `python -m pytest -q` — **725 passed, 8 existing warnings**, zero failures.
- `python scripts/verify_factor_migrations.py` — PASS on a fresh disposable
  PostgreSQL database: base/head lifecycle, `p27001` downgrade/upgrade boundary,
  seven tables, metadata parity, 209 factor tests, and concurrent atomic
  publication.
- Checkpoint 2.7.8 lifecycle — two timestamps, exact nine golden values, PIT
  future/wrong-build exclusions, exact source/peer/industry lineage, full
  idempotency, build isolation, and factor-local development quality.
- Checkpoint 2.7.9 scale proof — 10,000 securities, 80,001 factors, 160,003
  subfactors, 390,008 component decisions, 667 SQL statements, 1,337.533 seconds,
  549.125 MiB peak RSS, and 120,815,616 PostgreSQL temp bytes. See
  [the scale report](phase-2.7-scale-sanity.md).
- Final maximum-batch replay proof — 250 securities plus one industry node, all
  nine factors, 2,001 identical rerun identities, one JSONB equality query.
- Persistent Alembic current/head: **`p27001`**. `git diff --check` passes.

The full 10,000-security first-publication path does not execute the new
pre-existing-row JSONB comparison, which is asserted directly. A repeat attempted
late in this long audit session was not accepted as new measurement evidence:
only about 437 MiB host memory remained while the already-frozen successful run
had measured a 549.125 MiB Python peak before PostgreSQL overhead. The OS killed
the process and its exact disposable database was cleaned. This environmental
retry does not supersede the successful checkpoint measurement or the maximum-
batch replay proof, and no threshold was weakened to make it pass.

## External limitations that remain

- No commercial historical estimate, market/action, classification, universe,
  identity/delisting, or broad fundamental source has completed the existing
  production qualification and licensing gates.
- Synthetic estimate/peer/industry evidence remains `DEVELOPMENT`; deterministic
  correctness is not empirical predictive validity.
- The nine definitions are candidates. There is no calibration, outcome study,
  rank, signal, probability, portfolio rule, buy recommendation, or promotion.
- The 22-minute synthetic development-host run is not a production SLA.
- The eight existing UTC/SQLAlchemy deprecation warnings are outside Phase 2.7.

Audit commit locator after publication:

```bash
git log -1 --format='%H' --grep='^Complete Phase 2.7 implementation audit$'
```

Next action: independent Phase 2.7 QC/architecture review. Do not freeze by
implication, activate candidate factors for production, or begin Phase 2.8 during
that review.
