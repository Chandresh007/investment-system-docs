# Phase 2.7 Independent Freeze Review

Review date: 2026-09-23

Baseline reviewed: `76eab0897a6d0d09b4bb6f79c35e32d0f7d86337`

Branch: `master`

Migration head: `p27001`

## Verdict

**PASS — FREEZE APPROVED**

Phase 2.7 is frozen for factor definitions, normalization, hierarchy, coverage,
required anchors, quality propagation, PIT/build selection, storage, provenance,
immutability, runner behavior, and scale architecture. Phase 2.8 has not started
and is ready only for design.

This is a semantic and architectural freeze. It does not claim predictive power,
alpha, calibrated probabilities, production-provider qualification, or a buy/sell
recommendation.

## Scope and invariants

- The registry contains exactly nine v1 factors, 19 subfactors, and 47 components.
  There is no omnibus, valuation, business-quality, news, signal, probability,
  recommendation, or universe-ranking output.
- All source IDs, grouping, required anchors, fixed weights, orientations, and
  normalization knots match `phase-2.7-factor-spec.md`. Eight factors are
  `POSITIVE_EVIDENCE`; `market_fragility_v1` is `RISK_HIGHER_WORSE`, where `+1`
  means more fragility.
- Normalization is deterministic finite Decimal arithmetic. Peer percentiles are
  exactly `2p-1`. Exactly 60% subfactor and 70% total weighted coverage pass;
  reduced valid evidence is explicitly `REDUCED`; missing is never zero.
- Weight renormalization is local to a subfactor. A partially observed subfactor
  cannot consume another subfactor's fixed share.
- The runner reads frozen upstream artifacts and does not recalculate Phase 2.2,
  2.4, 2.5, or 2.6 formulas. Git history and the full regression show those
  formula/runner bodies remain unchanged.
- Exact T, period/target, source build/dataset, peer group, industry node, and
  authoritative source linkage are enforced. Exact reruns are idempotent;
  changed content at a published identity fails closed.
- Weakest-input quality is factor-local. Synthetic/limited evidence cannot be
  upgraded or contaminate an unrelated factor.
- Seven dedicated factor tables retain immutable definitions, manifests, values,
  subfactors, every included/excluded component decision, and the append-only
  promotion/version hook. Factors are not stored as `FeatureValue` outputs.

## Defects reproduced and fixed

No factor economics changed. The review reproduced and corrected these
fail-closed defects in the Phase 2.7 runner:

1. A valid embedded row admission caused the manifest-pinned source admission to
   be ignored instead of participating in weakest-input quality. Malformed row or
   component admissions could also fall through to a stronger fallback, and a
   malformed provider collection could claim production quality.
2. A source row that explicitly declared a dataset identity conflicting with the
   manifest was accepted when its calculation version matched. Estimate manifests
   also did not enforce the exact `estimates_v1:<profile>:<dataset>` identity.
3. Authoritative peer lineage did not recheck the group's effective date or the
   statistic's exact pinned dataset/source-build identity.
4. An industry metric could link a same-version/same-unit statistic for the wrong
   source feature, and the statistic's exact dataset/build identity was not
   rechecked.

Regression tests now cover each case. A separate rollback-only corruption test
changes an immutable upstream `FeatureValue` after publication and confirms replay
fails with `PUBLISHED_FACTOR_CONFLICT_USE_NEW_BUILD_OR_VERSION`. The internal
audit's deferred JSON replay fix was also independently exercised and held.

## Storage, migration, concurrency, and indexes

The disposable verifier passed fresh base → `p27001` → `p26008` → `p27001`, all
seven factor tables, metadata parity, factor tests, immutable/FK guards, and two
real concurrent publishers converging on one complete graph. `alembic current`,
`heads`, and `check` pass at `p27001`.

Existing indexes support definition/version lookup, security/industry factor
lookup by research timestamp, exact factor identity/build, forward component
lineage, and upstream snapshot retrieval. No observed plan justified another
index or a migration change.

## Scale assessment

Current corrected code was measured on disposable PostgreSQL 16.15 databases:

| Securities | SQL statements | Peak RSS | Temp I/O | Wall time |
| ---: | ---: | ---: | ---: | ---: |
| 1,000 | 128 | 353.258 MiB | 38,805,504 bytes | 124.286 s |
| 5,000 | 368 | 453.484 MiB | 32,653,312 bytes | 647.026 s |
| 10,000 | 668 | 560.969 MiB | 36,913,152 bytes | 1,345.070 s |

The 10,000-security run persisted exactly 80,001 factors, 160,003 subfactors,
and 390,008 component decisions. Statement growth is 15 statements per fixed
250-security batch plus fixed overhead, not per security or component. The one
statement increase from the implementation-audit proof is a fixed industry
source-feature lineage lookup.

With PostgreSQL `work_mem=4MB`, only the wide ordered exact-source selection sort
spilled in the captured 10k plans (521 temp-read / 522 temp-written blocks).
Source-presence aggregation and component-lineage verification did not spill.
Memory growth remained bounded, and one industry result/eight decisions were not
multiplied across the security universe. The architecture is structurally healthy
enough to freeze; the development-host runtime is not a production SLA.

## Verification

- `python -m pytest -q tests/factors` — **216 passed**.
- Focused factors plus peer/industry/lifecycle paths — **223 passed**.
- Full repository suite — **736 passed, 8 existing warnings, 0 failures** after
  the final documentation update.
- Disposable migration/concurrency verifier — **PASS**.
- Corrected-code 10,000-security scale verifier — **PASS**.
- `git diff --check` — **PASS**.

## External limitations

Production providers, licensing, broad retained historical coverage, complete
delisted/security-identity history, empirical predictive validation, calibration,
and promotion remain future gates. Synthetic estimate evidence remains
`DEVELOPMENT`. A frozen factor value is an evidence summary, not a probability or
recommendation.

## Freeze decision

The exact v1 factor semantics may now be depended on by later phases. Any change
to components, weights, anchors, normalization economics, orientation, coverage,
or quality semantics requires a new factor version rather than mutation in place.

**Phase 2.7: FROZEN**

**Phase 2.8: READY TO DESIGN**
