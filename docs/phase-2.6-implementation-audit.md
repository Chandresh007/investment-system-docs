# Phase 2.6 peer / industry implementation audit

Audit date: 2026-09-22. Starting checkpoint:
`e80111241c0866d6777aa34536504a7c8327aa5d` on `master`, equal to
`origin/master` with a clean worktree after checkpoint 2.6.6.

Result: **PASS after corrections below**. Phase 2.6 implementation is complete
and ready for independent review. It is **not yet frozen**. Phase 2.7 has not
started. Foundation and Phase 2.5 behavior remains frozen.

This audit tested the implementation against
[the frozen Phase 2.6 specification](phase-2.6-peer-industry-spec.md), rather
than treating the existing tests as the specification.

## Evidence map

| Area | Result and concrete evidence |
| --- | --- |
| PIT membership | `PeerResolver` restricts classification, eligibility, and identity evidence to the exact sealed datasets and `available_at <= T`, ranks evidence heads with PostgreSQL `row_number()`, and only then applies effective intervals. Overlaps fail closed. The two-release lifecycle covers future announcements, exits, delisting, correction, and re-entry boundaries. |
| Taxonomy identity | Resolution binds the manifest taxonomy, exact effective/known release, release-specific node versions, full four-level path, and exact-node policy. There is no name matching or automatic hierarchy fallback. |
| Eligibility | Membership requires active historical eligibility and identity plus a company-backed security. The one-primary-security rule is evaluated across the whole active eligibility corpus, including a second alleged primary classified outside the target node. Both ambiguous securities remain evidenced and are excluded. |
| Membership versus feature eligibility | Every exact-node classification candidate is persisted first. Missing, invalid, stale, incompatible, wrong-build, or unqualified source values become observation exclusions and remain in the coverage denominator; no test deletes a historical `FeatureValue` to simulate absence. |
| Source snapshot selection | Fundamentals select greatest measured `period_end <= D` and then newest `available_at <= T` in the exact calculation build. Estimates and market require exact T/D. Newer missing/invalid rows do not fall back. `PeerFeatureRunner.get_snapshot` now also binds effective D, so a same-build/same-T row for another period cannot leak into the result. |
| Dataset/build isolation | Feature selection filters the exact manifest calculation identity. Provenance-bound families validate the exact dataset key/version/hash. Wrong versions, datasets, future rows, and non-exact snapshots have separate exclusions. |
| Provider admission | Each selected source row must carry its own provider admission; group-level admission is no longer substituted for missing row evidence. Provider identity, catalog qualification version, trust, capabilities, domain, evidence reference, synthetic flag, coverage bounds, limitations, override, and quality are checked. The admitted interval must contain the selected artifact. Missing or mismatched evidence fails closed. |
| Quality | Output quality is the weakest group and included source quality. Synthetic evidence, any limitation, or a development override forces `DEVELOPMENT`; an asserted stronger row quality cannot upgrade it. Excluded observations affect coverage but do not masquerade as inputs. |
| Numeric semantics | All contributing values must be finite exact `Decimal` values. Canonical outputs remain median, target minus peer median, and ascending midrank `(L + 0.5E) / N`. Ties, negatives, zero, odd/even medians, and non-finite inputs are covered. No z-score or winsorization was introduced. |
| Thresholds | Relative results require five non-target membership peers, five valid non-target observations, and 60% coverage. All-member industry distributions independently require five valid members and 60% coverage. Database checks now prevent a row marked `VALID` from violating those frozen thresholds. |
| Registry and projection | The exact ten inactive `peer_v1` feature definitions and eight direct `industry_v1` metrics remain isolated from legacy scanners. Peer projection references the authoritative relative result and exact group/statistic/source build. Exact reruns reuse immutable identities. |
| Relational provenance | Database guards bind each peer member to classification/eligibility/identity evidence for the same security and bind each observation/result to the correct member, security, source feature, calculation build, and `FeatureValue`. Cross-security substitutions are rejected independently of application code. |
| Immutability and atomicity | Existing foundation UPDATE/DELETE protections remain unchanged. Group, observation, result, metric, and projected-feature publication uses nested transactional scope; injected failure rolls the complete batch back. Concurrent writers converge on one immutable identity. |
| Scale | A disposable 10,001-security PostgreSQL fixture includes two versions per classification, eligibility, and identity key. It resolves 10,001 group members and publishes 10,000 valid non-target observations in 52 SQL statements. Resolver head selection and feature selection use four window queries, and inserts are safely batched below PostgreSQL's bind-parameter ceiling. |
| Migrations | `p26006` hardens observation/result source links. `p26007` adds member-evidence guards, finite-number checks, and valid-threshold constraints. The disposable verifier exercises fresh base through head and every Phase 2.6 downgrade/upgrade boundary, a populated industry-count backfill, concurrent publication, metadata parity, and all peer tests. |

## Corrections made during this audit

1. **Primary-security ambiguity was node-local.** A second active eligible
   security for the same company escaped the ambiguity rule when classified
   outside the target node. The resolver now computes ambiguous companies from
   the complete active eligibility corpus before applying the decision to
   candidates. Regression:
   `test_primary_security_ambiguity_is_global_not_limited_to_the_target_node`.

2. **Feature admission could be borrowed from the peer group.** A source
   `FeatureValue` without row-level provider admission inherited group admission,
   allowing unproven source quality. Rows now fail closed, the selected artifact
   must be inside the recorded admission interval, and catalog-backed provider
   fields/coverage are checked. Regressions:
   `test_missing_row_level_provider_admission_fails_closed` and
   `test_source_admission_must_cover_the_selected_artifact`.

3. **Exact projected reads omitted effective date.** A same-security,
   same-feature, same-T, same-build row with another `period_end` could enter
   `get_snapshot`. The runner now derives D from the pinned timezone/session
   policy and filters it exactly. Regression:
   `test_get_snapshot_rejects_same_build_and_timestamp_with_wrong_effective_date`.

4. **Relational source FKs were not content-bound.** The original persistence
   trigger proved group/member association but allowed an observation or result
   to reference another security's `FeatureValue`. Migration `p26006` binds the
   link to security, feature, calculation build, included member, and target
   observation. Two direct-database regressions prove rejection.

5. **Member evidence FKs could point at another security.** Migration `p26007`
   validates classification security/taxonomy/release, security/company,
   eligibility security/policy, identity security, and the complete included-row
   conditions. It preflights existing rows before installing the trigger.
   Regression: `test_database_rejects_cross_security_membership_evidence`.

6. **PostgreSQL `NUMERIC` admits non-finite values.** A `NaN` source reached
   Decimal ordering and could raise instead of producing explicit evidence.
   Non-finite inputs are now `FEATURE_INVALID`; database constraints also reject
   non-finite published observations, distributions, relative outputs, and
   industry metrics. Application and direct-database regressions cover both paths.

7. **Representative resolution was not actually set-based.** The Phase 2.6
   resolver reused foundation helpers that loaded all dataset rows and reduced
   evidence heads in Python. The Phase 2.6 path now ranks heads in SQL. Bulk
   member and observation publication is chunked at 1,000 rows, avoiding the
   PostgreSQL parameter ceiling exposed by the 10,001-security proof. Frozen
   foundation resolver semantics and Phase 2.2/2.4/2.5 formulas were not changed.

The interrupted statistics failures were not product defects: one fixture used a
non-64-character manifest hash, and another published then deleted a historical
`FeatureValue`. The latter violates the deliberately frozen append-only guard.
Missing-feature cases now model the correct history: the row was never published.
No immutability protection was weakened.

## Scale evidence

`python scripts/verify_peer_scale.py` creates and force-drops only a random
disposable database. The verified 2026-09-22 run produced:

```text
securities                       10,001
included group members           10,001
non-target membership peers      10,000
valid non-target observations    10,000
result status                    VALID
SQL statement count              52
wall time                        63.914 seconds
peak traced Python memory        245.017 MiB
PostgreSQL                       16.15
window-plan execution times      66.862, 65.006, 56.669, 28.556 ms
window-plan actual rows          10,001 each
temporary read/write blocks      0 / 0 for every plan
```

The plans used bounded window aggregation and ordinary hash/sort/index paths.
This is a representative correctness/query-plan gate, not a production SLO or a
claim of arbitrary-universe capacity. No index or infrastructure was added
without measured need.

## Verification

- `python -m pytest -q tests/peers` — **77 passed**, zero failures.
- `python -m pytest -q` — **508 passed, 8 existing warnings**, zero failures.
- `python scripts/verify_peer_migrations.py` — PASS on fresh PostgreSQL at
  `p26007`, all Phase 2.6 downgrade/upgrade boundaries, populated backfill,
  concurrency, metadata check, and 77 peer tests.
- `python scripts/verify_peer_scale.py` — PASS with the measurements above.
- Persistent Alembic current/head: **p26007**; `alembic check`: no new upgrade
  operations. The read-only `p26007` member-integrity preflight returned zero
  inconsistent existing rows.
- `git diff --check`: clean.

## Remaining gates

- No live/commercial historical taxonomy, primary-security universe,
  security-master, market, estimate, or fundamental provider has been qualified
  for production peer claims. Licensing, retained history, delisted coverage,
  corrections, and derived-data rights remain external gates.
- Frozen legacy fundamental and market `FeatureValue` publishers do not invent
  Phase 2.6 admission. A source row without adequate row-level admission is
  deliberately excluded. The peer registries remain inactive pending qualified
  orchestration and independent review.
- The scale proof is synthetic and single-calculation. It does not certify a
  production service-level objective, broad concurrent workload, or investment
  usefulness.
- Existing UTC and SQLAlchemy deprecation warnings remain outside this phase.

Audit commit locator after publication:

```bash
git log -1 --format='%H' --grep='^Complete Phase 2.6 implementation audit$'
```

Next action: independent Phase 2.6 QC/architecture review. Do not freeze by
implication, activate production outputs, or begin Phase 2.7 during that review.
