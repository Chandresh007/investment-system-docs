# Phase 2.7 Factor Scale / Performance Sanity

Date: 2026-09-23

Status: PASS for Phase 2.7 development-scale structural sanity
Scope: deterministic synthetic PostgreSQL workload, not an investment-performance
or production-provider claim

## 1. Workload

`scripts/verify_factor_scale.py` creates a random disposable PostgreSQL database,
migrates base to `p27001`, and drops only that database after the proof. The final
run used PostgreSQL 16.15 and:

- 10,000 security subjects;
- the exact nine Phase 2.7 factor definitions (eight security-scoped definitions
  plus one shared industry-node definition);
- 29 direct source feature IDs at the exact manifest-pinned build;
- all ten authoritative peer outputs for one formal-node target;
- all eight authoritative industry metrics for the target's node;
- 80,001 persisted factor values;
- 160,003 persisted subfactor values; and
- 390,008 persisted component decisions.

The 9,999 non-target securities deliberately lack projected peer artifacts. Their
seven peer-anchored factors are explicit `MISSING`; Market Fragility remains
`VALID`. The target has all eight security factors valid and the one industry
factor is valid. This produces 10,008 `VALID`, 69,993 `MISSING`, zero `INVALID`,
and 80,001 `DEVELOPMENT` results. Synthetic qualification is never presented as
production quality.

## 2. Final measurements

| Measurement | Result |
| --- | ---: |
| Factor-run wall time | 1,337.533 seconds |
| SQL statements | 667 |
| `SELECT` | 304 |
| `INSERT` | 341 |
| `SAVEPOINT` / `RELEASE` | 11 / 11 |
| Baseline process RSS | 81.227 MiB |
| Peak sampled process RSS | 549.125 MiB |
| Factor-run RSS increase | 467.898 MiB |
| PostgreSQL temporary files | 20 |
| PostgreSQL temporary bytes | 120,815,616 |

Statement count is approximately 0.067 per security and grows by fixed 250-
security batches, not per security or component. It includes immutable definition
resolution, source selection/presence, all bulk inserts, publication conflict
checks, and complete relational lineage verification.

## 3. Query plans and indexes

Three representative statements were rerun with
`EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` after publication.

### Exact source selection (one 250-security batch)

- 7,260 actual rows;
- 429.409 ms execution;
- plan nodes: `Gather Merge`, `Sort`, `Bitmap Heap Scan`, `Bitmap Index Scan`;
- 547 shared-hit and 1,380 shared-read blocks; and
- 525 temp-read / 526 temp-written blocks.

The source query uses an index-backed bitmap path. The wide ordered source rows
spill a small sort under this host's PostgreSQL work-memory setting. Across the
whole factor run the measured temporary write volume was about 115.2 MiB, which is
bounded and not an unbounded materialization.

### Source-presence aggregate (one batch)

- 7,260 actual rows;
- 111.722 ms execution;
- plan nodes: `Aggregate`, `Bitmap Heap Scan`, `BitmapAnd`, two
  `Bitmap Index Scan` nodes;
- 1,924 shared-hit and 231 shared-read blocks; and
- no temporary I/O.

### Component-lineage verification (one batch)

- 9,758 actual rows;
- 1,457.763 ms execution;
- plan nodes: `Gather`, `Nested Loop`, `Seq Scan`, `Bitmap Heap Scan`,
  `Bitmap Index Scan`;
- 9,124 shared-hit and 14,379 shared-read blocks; and
- no temporary I/O.

The mixed plan scans one bounded side and uses the component FK/index path for the
large relation. Exact row counts prove that no security-by-industry Cartesian
expansion occurred: one industry factor and eight industry components are stored,
not 10,000 copies.

## 4. Structural defects reproduced and corrected

The first measured attempt used an unbounded publication statement and whole-run
source/lineage materialization. Review before execution identified PostgreSQL bind
limit and memory risks. Publication now uses parameter-safe 2,000-row inserts and
join-scoped lineage reads.

A 1,000-security internal batch under `tracemalloc` was killed by the host memory
limit. Replacing allocation tracing with an RSS sampler removed measurement
inflation but exposed real accumulation from the returned ORM result payload.
The final implementation therefore:

1. validates the manifest and definitions once;
2. retains one outer savepoint/atomic request;
3. resolves and publishes fixed 250-security batches;
4. publishes the node-scoped industry factor only once;
5. preserves registry-major and caller subject ordering after batching;
6. defers the two large `FactorValue` JSON payloads only for large requests while
   retaining all scalar result fields and immutable fingerprints;
7. leaves those JSON fields lazy-accessible and keeps authoritative evidence in
   relational component rows; and
8. retains the v1 top-level component-ID provenance so performance changes do not
   alter historical fingerprints under the frozen runner identity.

Two killed disposable attempts left random scale databases because the OS could
not execute Python `finally`; each exact database name was resolved read-only and
force-dropped before the next run. The successful run executed its normal cleanup.
No configured research database was populated, downgraded, or dropped.

### Final-audit replay hardening

Checkpoint 2.7.10 reproduced one integrity gap: a large rerun compared deferred
admission/provenance through its fingerprint but did not independently compare the
two JSON objects. Pre-existing or concurrently won rows now use one batch-level
PostgreSQL JSONB equality query; rows inserted from the exact expected payload in
the current statement do not incur redundant readback. The return payloads remain
deferred.

The fresh 10,000-security path measured above is therefore unchanged and a forced
regression asserts it performs zero payload-equality queries. The corrected replay
path passed at the maximum internal batch size: 250 securities plus one industry
node, all nine definitions, 2,001 factor values, one JSONB equality statement, and
identical IDs, fingerprints, and execution timestamps. See the
[final implementation audit](phase-2.7-implementation-audit.md) for the reproduced
tamper case and memory-safe correction.

## 5. Independent freeze-review rerun

The independent review reran the same production runner after its fail-closed
quality/build/lineage corrections. Disposable 1,000-, 5,000-, and 10,000-security
runs produced:

| Securities | SQL statements | Peak RSS | Temp bytes | Wall time |
| ---: | ---: | ---: | ---: | ---: |
| 1,000 | 128 | 353.258 MiB | 38,805,504 | 124.286 s |
| 5,000 | 368 | 453.484 MiB | 32,653,312 | 647.026 s |
| 10,000 | 668 | 560.969 MiB | 36,913,152 | 1,345.070 s |

The corrected 10,000-security run retained the exact 80,001 / 160,003 / 390,008
factor, subfactor, and component cardinalities. Its 305 `SELECT`, 341 `INSERT`,
11 `SAVEPOINT`, and 11 `RELEASE` statements differ from the prior proof by one
fixed `SELECT` that validates the industry statistic's source-feature identity.
The slope remains exactly 15 statements per 250-security batch between 1k and 5k
and approximately the same through 10k; there is no per-security query loop.

At the unchanged PostgreSQL `work_mem=4MB`, the 10k exact-source selection plan
used an index-backed bitmap scan plus `Gather Merge`/`Sort` and recorded 521
temp-read / 522 temp-written blocks. The source-presence aggregate and component-
lineage verification plans recorded zero temp blocks. All random review databases
were dropped by their normal cleanup paths.

## 6. Verdict and limitations

The final shape is structurally healthy for Phase 2.7:

- no per-security or per-component SQL loop;
- no unbounded source/lineage ORM graph;
- parameter-safe bulk publication;
- bounded temporary I/O;
- index-backed source and component access;
- exact expected row cardinalities;
- no industry Cartesian duplication; and
- bounded memory on the approximately 2 GiB development host.

The 22-minute development runtime is candidly not a production SLA. Most volume
comes from append-only publication plus relational integrity/immutability checks
for roughly 630,000 result and explanation rows. Future operational tuning may
measure faster hardware, PostgreSQL work memory, COPY/staging, or trigger-aware
bulk strategies, but must not weaken atomicity, immutability, PIT, quality,
coverage, or provenance. A distributed system is not justified by this result.
