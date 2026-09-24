# Phase 2.6 Independent Freeze Review

## Verdict

**PASS — FREEZE APPROVED**

Phase 2.6 is frozen for its implemented taxonomy, historical membership, peer,
peer-statistic, industry-snapshot, provenance, quality, and migration semantics.
Phase 2.7 is ready to design and has not started.

## Repository reviewed

- Baseline commit: `7c689f4a44d3f3b66d97ad09363ee8458661c56f`
- Baseline branch: `master`, equal to `origin/master`, initially clean
- Review/freeze changes: this review's `Freeze Phase 2.6 peer and industry foundation`
  commit
- Authoritative specification, architecture review, internal audit, prior freeze
  review, trust contract, implementation, migrations, tests, and scale verifier were
  reviewed directly.

## Scope and critical invariants

- Membership selects the latest evidence head known at `T` before applying the
  end-exclusive effective interval at `D`. Sealed builds prevent later corrections
  and backfills from changing an older snapshot.
- Taxonomy family, release/evidence version, stable node IDs, release-specific
  hierarchy, and classification evidence remain distinct and reconstructable.
- Canonical resolution is exact same historical sub-industry, deterministic by
  permanent security ID, excludes the target, and never falls back implicitly.
- Five non-target members, five valid observations, and 60% feature coverage are
  separate gates. Feature exclusions never rewrite membership.
- Source selection is exact-build and PIT-safe. Fundamentals select the newest
  eligible measured quarter without fallback; estimate and market inputs require
  exact `T`/`D` snapshots.
- V1 arithmetic is finite-Decimal median, target minus peer median, and ascending
  midrank `(L + 0.5E) / N`. There is no winsorization, z-score, or desirability
  inversion.
- Output quality is the weakest admitted required/included input. Synthetic or
  overridden evidence remains `DEVELOPMENT`; survivorship claims fail closed.
- The exact ten inactive `peer_v1` features and eight `industry_v1` metrics read
  frozen upstream FeatureValues and do not alter Phase 2.2, 2.4, or 2.5 formulas.

## Defects reproduced and fixed

1. A source FeatureValue could claim provider coverage wider than the runtime
   qualification. Row coverage must now be contained by the qualified provider
   interval; an adversarial row is retained as `PROVIDER_NOT_QUALIFIED` rather than
   admitted.
2. Same-security classification, eligibility, or identity evidence could be linked
   from outside the snapshot's sealed datasets. Migration `p26008` adds relational
   dataset IDs and composite foreign keys binding every member to both its exact
   group datasets and dataset/evidence pairs. The populated migration preflight and
   downgrade/upgrade lifecycle pass.
3. Persisted peer artifacts carried the manifest hash but no reconstructable
   canonical manifest payload or location. Group provenance now stores the exact
   canonical payload, and the lifecycle walks a projected FeatureValue through the
   complete relational chain.

## Validation

- Peer suite: **82 passed**
- Full suite: **513 passed, 0 failed, 8 existing deprecation warnings**
- Alembic current/head: **`p26008`**
- `alembic check`: no new upgrade operations
- Disposable lifecycle: fresh base through `p26008`, every Phase 2.6 boundary,
  populated backfill, metadata parity, concurrent one-winner publication, and peer
  tests: **PASS**
- Final 10,001-security proof: **52 SQL statements**, **10,000 valid non-target
  observations**, **69.282 seconds**, **248.163 MiB** peak traced Python memory,
  four bounded window plans, and **zero PostgreSQL temporary I/O**
- `git diff --check`: clean

The scale proof is a correctness and query-shape gate on the small development
host, not a production latency benchmark. It shows bounded queries, batched
publication, reasonable memory, indexed relational guards, and no obvious N+1 or
spill design.

## External limitations

Licensed historical GICS/classification data, a qualified historical
primary-common-equity universe, complete delisted-security coverage, production
source-feature providers, and strict accounting-framework provenance remain
external gates. Synthetic acceptance evidence remains `DEVELOPMENT`; runtime
admission fails closed for stronger claims. This freeze is not live-data or
investment-performance certification, and peer percentiles are descriptive—not
buy/sell signals.

## Freeze decision

Phase 2.6 is safe to freeze. Phase 2.7 may proceed to design only; no Phase 2.7
implementation is included in this review.
