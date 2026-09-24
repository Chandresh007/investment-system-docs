# Phase 2.8 Signal Scale Sanity

Status: **PASSED** for the Phase 2.8 structural scale gate.

This is a development-machine structural check, not a production throughput
claim. It tests the production `SignalRunner` against immutable, build-pinned
Phase 2.7 factor artifacts in a disposable PostgreSQL database. It does not
change a signal rule, threshold, proof policy, quality policy, factor formula,
or economic interpretation.

## Method

[`scripts/verify_signal_scale.py`](../scripts/verify_signal_scale.py) creates a
random database, migrates it from an empty schema to Alembic head, and then
publishes all nine Phase 2.8 signal definitions from both detector identities.
The configured research database is never populated, downgraded, or dropped.
The random database is force-dropped in a `finally` block.

Each run seeds:

- eight exact security-scoped factor values per security;
- one exact node-scoped `industry_strength_v1` value;
- all nine signal definitions and both detector identities;
- `PIT_QUALIFIED`, `EDUCATIONAL` factor evidence at one research timestamp and
  one pinned factor build; and
- no security-to-industry context in the signal request, deliberately proving
  that unavailable Industry confirmation remains explicit while an alternate
  confirmation can prove Early Growth.

Expected cardinality is checked before a run can pass:

```text
factor values      = 8 * securities + 1
signal results     = 7 * securities + 2 node results
condition results  = 22 * securities + 2 node conditions
node results       = 2 exactly
```

The verifier counts every SQL statement issued during signal evaluation and
publication, samples process RSS every 50 ms, reads PostgreSQL database temp-I/O
counters before and after, and replays representative statements through
`EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)`.

Environment: the designated 2-vCPU, 8-GB development host and PostgreSQL
16.15. Measurements were taken on 2026-09-23. Wall times include result and
condition persistence plus integrity verification; database creation, migration,
and factor seeding are excluded.

## Results

| Securities | Factor values | Signal results | Conditions | SQL statements | Wall time | Peak RSS | Run RSS delta | Temp files / bytes |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1,000 | 8,001 | 7,002 | 22,002 | 220 | 31.676 s | 191.016 MiB | 111.871 MiB | 0 / 0 |
| 5,000 | 40,001 | 35,002 | 110,002 | 364 | 154.791 s | 264.785 MiB | 185.055 MiB | 0 / 0 |
| 10,000 | 80,001 | 70,002 | 220,002 | 544 | 313.169 s | 356.000 MiB | 275.438 MiB | 0 / 0 |

Every run produced only `VALID/PIT_QUALIFIED` results. At 10,000 securities:

```text
fired true       = 60,001
fired false      = 10,001
FULL coverage    = 60,002
REDUCED coverage = 10,000
```

The six expected security signals fire for every security, including Early
Growth and Fragility Warning together. Cash Flow Contradiction is a valid
non-fire. The node-scoped Tailwind fires once and Headwind is a valid non-fire
once. Industry evidence is unavailable to each security request and therefore
reduces Early Growth coverage; it is not converted to zero and does not block
the stronger Estimate/Market/Fundamental Economics confirmation proof.

## SQL structure

Statement growth is batch-linear, not security-linear:

| Securities | SELECT | INSERT | SAVEPOINT | RELEASE | Total |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 1,000 | 83 | 75 | 31 | 31 | 220 |
| 5,000 | 163 | 139 | 31 | 31 | 364 |
| 10,000 | 263 | 219 | 31 | 31 | 544 |

After fixed setup, the slope is 36 statements per additional 1,000 securities,
matching four 250-security batches. There are no per-security definition queries,
per-condition factor queries, or N+1 publication queries. Factor retrieval and
lineage publication remain set-based within bounded batches.

Representative 10,000-security plans were:

| Operation | Returned rows | Plan | Execution | Temp blocks |
| --- | ---: | --- | ---: | ---: |
| factor batch retrieval | 2,001 | Gather + sequential scan | 22.365 ms | 0 |
| result graph verification | 1,752 | bitmap heap/index scan | 40.749 ms | 0 |
| condition graph verification | 5,502 | sequential scan | 38.502 ms | 0 |

PostgreSQL selected sequential scans for two representative bounded-batch
queries at this synthetic table size. They completed without sort/hash spill,
and neither plan nor statement-count behavior indicates N+1 access. No index or
query-policy change is justified from this development fixture alone.

## Memory finding and correction

The first complete matrix found that the returned ORM collection retained the
large `admission` and `provenance` JSON documents for every signal result. It did
not corrupt results, but it was avoidable materialization:

| Securities | Initial run RSS delta | Final run RSS delta |
| ---: | ---: | ---: |
| 1,000 | 179.137 MiB | 111.871 MiB |
| 5,000 | 568.438 MiB | 185.055 MiB |
| 10,000 | 1,053.480 MiB | 275.438 MiB |

Large multi-batch runs now defer those two ORM attributes on returned result
objects. The JSON is still persisted in full. Exact replay independently compares
stored and expected JSON in a batched PostgreSQL `jsonb_to_recordset` integrity
query, so deferral cannot weaken immutable-content verification. Focused tests
prove requested order, lazy access, idempotent replay, and fail-closed detection
of tampered deferred payloads.

## Conclusion and limitations

The Phase 2.8 runner passes the requested 10,000-security structural gate:
bounded batch retrieval, no N+1 pattern, exact artifact cardinality, complete
relational lineage, bounded process memory, and no observed PostgreSQL temp I/O.

This fixture does not qualify real providers, establish predictive validity,
set a production latency SLO, exercise a distributed worker fleet, or test Phase
2.9 calibration/backtesting. Results are specific to one synthetic evidence
shape and this development host. Those limitations do not justify changing the
frozen signal semantics.

Reproduction:

```bash
source .venv/bin/activate
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
IRS_SIGNAL_SCALE_SECURITIES=10000 python scripts/verify_signal_scale.py
```
