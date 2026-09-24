# Final foundation qualification review

Review date: 2026-09-22. Starting checkpoint:
`e4b3c603384b7d61d31e9ee5daa6b71cb0830ccf`. Phase 2.6 was not started and no
investment feature family was added. Frozen Phase 2.4 mathematics did not change.
Phase 2.2 formulas did not change; only the concrete filing-availability defect in
fact selection and derived publication time was repaired.

## A. Verdict

**PASS for foundation risk closure and enforcement.** This is not a claim that all
live inputs are institutional-quality. Every remaining provider, license, legacy,
coverage, or operational dependency is now denied the corresponding historical,
survivorship, raw-reproducibility, or production claim unless an explicit
development override is persisted. No live provider currently qualifies an end-to-
end production or survivorship-safe backtest.

Recommendation: **READY FOR INDEPENDENT FOUNDATION / PHASE 2.5 FREEZE REVIEW**.
Phase 2.6 remains **NOT STARTED**.

## Risk classification

| Area | Classification | Result |
| --- | --- | --- |
| Provider capability and replay admission | A — fixable in our code | Resolved with catalog, scoped qualification, hard gates, quality provenance |
| Yahoo historical economics and corrections | B/C — live qualification and likely commercial data | Externally blocked; development only |
| Yahoo/legacy market observations already stored without versions | D — irrecoverable legacy semantics | Excluded from qualified replay |
| Live ALFRED retrieval contract | A — fixable in our code | Connector and mocked contract implemented |
| Live ALFRED series coverage/timing/rights | B/C — live qualification and rights | `LIVE_ALFRED_CONNECTOR_PENDING`; blocked from replay |
| Existing raw objects without bytes/hashes | D — irrecoverable legacy data | Reconciled as test-only missing evidence; blocked |
| Raw recovery representation and admission | A — fixable in our code | Original/reconstructed/missing/mismatch states and gate implemented |
| Permanent identity and lifecycle representation | A — fixable in our code | Minimal bitemporal identity history implemented |
| Complete failed/delisted universe and terminal returns | B/C — provider coverage/license | Externally blocked; survivorship claim rejected |
| SEC intraday/date-only availability | A — fixable in our code | Exact acceptance or next-day conservative policy implemented |
| Company Facts aggregate history/fiscal reconciliation | B/D/E — live validation, missing original evidence, development limitation | Accession scope only; aggregate blocked from replay |
| S3-compatible upload workflow | A — fixable in our code | Safe dry run, encryption/checksum, manifest-last upload implemented |
| Live off-host credentials, retention lock, restore drill | B/C — deployment/account | Externally blocked; no live durability claim |
| Production analyst estimates | B/C/E — provider/license, development limitation | Synthetic-only gate retained |

## B. Market provider qualification

The canonical provider catalog grants Yahoo only `CURRENT_RESEARCH`. It does not
grant historical replay, raw reproducibility, action PIT, historical-security,
delisted, lifecycle, survivorship, or verified-license capabilities. A direct
historical request fails with `SOURCE_NOT_ELIGIBLE_FOR_HISTORICAL_REPLAY`. A
survivorship-safe request fails with
`BACKTEST_DATA_NOT_SURVIVORSHIP_QUALIFIED`.

The Yahoo adapter archives the original response and exact request without secrets.
The normalizer accepts `open`, `high`, `low`, `close`, and `volume` as provider-
reported unqualified fields. `indicators.adjclose[0].adjclose` is stored separately
as provider-derived, non-PIT-safe metadata. The Phase 2.4 engine continues to use
`close`; an adversarial test makes adjusted close differ by 100x and proves it is
not consumed.

Calculation correctness remains separate from evidence quality. Split, reverse-
split, and dividend logic remains mathematically frozen and tested. No production
corporate-action feed is qualified. Ticker changes, exchange changes, mergers,
acquisitions, spinoffs, delisting events/proceeds, corrections, announcement time,
and compatible share basis remain provider requirements.

## C. Macro vintage qualification

The canonical `EconomicVintage` model and resolver still prove initial GDP 2.0
before a revision and 2.5 after it. Ordinary FRED remains latest-view context and
cannot enter historical replay.

`ALFREDConnector` now requests the official observations endpoint with explicit
real-time and observation bounds, `output_type=1`, count/offset pagination, and
archived sanitized raw pages. `ALFREDMacroVintageNormalizer` preserves series,
observation date, exact value/missingness, real-time start, conservative availability,
receipt, raw evidence, dataset, and normalization version. Mocked official-shape
tests require no network. Live credentials, representative series coverage,
intraday release evidence, pagination at scale, and rights remain unverified, so
the catalog status is exactly `LIVE_ALFRED_CONNECTOR_PENDING` and historical replay
is rejected.

## D. SEC PIT qualification

The accession-level profile is authoritative only when the run preserves original
filing bytes, accession identity, acceptance evidence, version, and coverage. The
Company Facts profile remains unqualified for replay because today's aggregate
response alone cannot establish every earlier API state.

The schema now stores filing/fact `available_at`, availability policy, fact-to-
filing link, and reporting fiscal context. Aware acceptance timestamps retain exact
UTC instants. Date-only evidence is unavailable until the next New York midnight.
The fact resolver and derived feature time use this policy instead of filing-date
midnight. A 16:30 ET filing is absent before 16:30 and eligible at that instant
under the historical acceptance policy; operational receipt policy may be later.

The comparative golden case has an original prior-year quarter, the same period
reported comparatively in a later filing, a later amendment/restated value, and a
current quarter. Earlier research retains the original. Each later value becomes
visible only at its own filing time. An earnings release before the filing is a
different source and cannot backdate EDGAR evidence.

## E. Legacy raw reconciliation

The read-only reconciliation found **34**, not 32, current metadata rows. The prior
audit counted 32 before additional required full-suite executions; the legacy
`test_provenance` case committed one new `/tmp/raw_facts.json` row per run. That test
now removes its own receipt, so the count no longer grows. Existing metadata was
not deleted.

All 34 rows are test-only: 33 SEC `api/facts` rows point to absent
`/tmp/raw_facts.json`; one RSS row points to absent `/tmp/raw_100`. All lack expected
hash and size. They are `UNVERIFIABLE_LEGACY` / `MISSING`; the RSS row is referenced
only by a legacy test news article and none support a financial fact or feature.
They cannot pass raw reproducibility. The checked-in aggregate
manifest is [`LEGACY_RAW_RECONCILIATION.json`](LEGACY_RAW_RECONCILIATION.json); the
script emits every row.

New receipts are `ORIGINAL_ARCHIVE`. A refetch creates a new
`RECONSTRUCTED_COPY` with `recovered_at`, `recovery_source`, new content hash, and
`original_raw_object_id`. It never rewrites the old hash or certifies the original
historical bytes. Runtime distinguishes `VERIFIED_ORIGINAL`,
`VERIFIED_RECONSTRUCTED`, `MISSING`, `HASH_MISMATCH`, and
`UNVERIFIABLE_LEGACY`. Raw-reproducible research accepts originals by default;
reconstructed bytes require a separate explicit option and still do not prove prior
PIT content.

## F. Survivorship qualification

Historical universe knowledge timing remains resolved. A new minimal
`SecurityIdentityHistory` keeps permanent internal `security_id` while versioning
ticker, exchange, lifecycle status, effective interval, knowledge time, source,
and optional successor. The PIT resolver selects the known version before the
effective interval and never consults today's `Security.is_active`. Tests cover a
ticker/exchange change and later delisting under one permanent ID.

This representation does not invent coverage. No live provider supplies verified
bankrupt, acquired, merged, delisted, ticker-reuse, multiple-share-class, exchange-
change, successor, or terminal-return history. A historical universe can retain a
known internal ID even when Yahoo cannot retrieve it today, but the resulting run
cannot claim complete historical market coverage. The survivorship gate requires
all of historical replay, historical universe, historical security coverage,
PIT corporate actions, delisted history, lifecycle, and explicit survivorship-safe
qualification. No live profile passes.

## G. Off-host data durability

Local backup still captures a consistent PostgreSQL snapshot plus every referenced
raw byte. Its raw inventory now includes source, path/object key, SHA-256, size,
archive kind, and recovery lineage. Restore continues to validate full DB table
digests, complete raw hashes, schema, and an as-of research query.

`scripts/upload_backup.sh` / `upload_backup.py` add S3-compatible operation. The
destination is supplied only by `IRS_BACKUP_S3_URI`; credentials stay in the AWS
credential chain. HTTPS is required for custom endpoints. Server-side AES-256 or
KMS encryption is mandatory. Local artifacts are verified before planning; remote
objects are checked before upload, matching objects are reused, conflicting objects
fail rather than overwrite, and absent objects use atomic conditional writes.
SHA-256 is requested and verified, and `manifest.sha256` uploads last. `--dry-run`
performs no external write. Files above the safe 5 GiB single-object path fail
before transfer until a conditional multipart implementation is qualified.

No cloud credentials were available or required. Therefore a live off-host copy,
bucket versioning/object lock, scheduler, alert, key recovery, and remote restore
remain deployment gates. Retention is deployment configuration: daily recent,
weekly longer, monthly archive, plus backups before major migrations, provider
migrations, and schema rewrites. A practical starting policy is 14 daily, 8 weekly,
and 12 monthly copies, subject to license and storage policy.

## H. Runtime historical-replay gates

`config/provider_qualifications.yaml` is canonical. `assert_provider_eligible` and
`require_historical_replay` validate trust, exact version, evidence record, aware
coverage, timestamp, capability set, and synthetic permission. Manifest provider
versions must match the qualification set exactly.

An override is not a boolean. It requires `requested_by`, `reason`, and `reference`.
The returned admission record includes every provider capability and limitation,
the override, purpose, timestamp, and `data_quality_level=DEVELOPMENT`. This is the
minimum future backtester/research-run provenance contract. Qualified PIT,
survivorship, and production labels cannot be assigned through an override.

## I. Provider qualification records

Versioned records live under [`provider-qualifications/`](provider-qualifications/README.md).
They cover purpose, endpoint, PIT and revision semantics, coverage, delisted support,
rights, gaps, replay eligibility, production status, date, and version. The runtime
catalog carries the exact trust and capabilities, preventing prose from silently
granting a stronger status.

## J. Tests

Final full regression: **428 passed, 8 existing warnings, zero failures**. New
adversarial coverage includes Yahoo replay refusal and recorded override,
survivorship refusal/fixture admission, raw state distinctions, identity history,
adjusted-close exclusion, SEC same-day and comparative/amendment behavior, mocked
ALFRED revisions, S3 configuration/dry-run/conflict behavior, and existing backup
corruption checks. No test uses live internet or cloud credentials.

Alembic current/head: **fnd005**. `git diff --check` passes. On clean committed
checkpoint `0e6977d338ee13240876e1f0766dffe4c752799a`, the final disposable drill
passed fresh base→head, downgrade to `25e001`, re-upgrade, 50 foundation tests,
PostgreSQL plus raw-byte backup, isolated restore, exact table/query/hash checks,
and tamper rejection. The full publication evidence is in the project-state entry.

## K. Documentation

README, project state, source trust, provider register, durability/recovery,
foundation audit, external handoff, provider qualification records, legacy raw
manifest, and this review state the same boundary: code safeguards are resolved;
unproved provider and deployment capabilities remain externally blocked.

## L. Git

Implementation and qualification records were published in
`798b8d3c5a7e04cf59252dccd640eef4b4a13056`; atomic off-host protection is in
`0e6977d338ee13240876e1f0766dffe4c752799a`. A final documentation checkpoint
records the latter commit's clean-checkout drill. Publication requires
`HEAD == origin/master` with a clean working tree; that terminal state is reported
to the owner after the final push rather than embedded into a self-referential commit.

## M. Remaining externally blocked items

- A licensed/versioned raw market and corporate-action source with corrections,
  stable IDs, failed/delisted securities, terminal proceeds, and verified rights.
- Live ALFRED credentialed qualification by series, timing, coverage, pagination,
  retention, and rights.
- Company Facts accession-by-accession reconciliation and broad historical
  fiscal-context coverage; separate earnings-release evidence where needed.
- A production estimate event provider and independent Phase 2.5 freeze review.
- Live encrypted off-host upload, retention/versioning/lock configuration,
  scheduling/alerting, key recovery, and a remote restore drill.
- Operational least-privilege database roles and a future sealed prediction ledger.

These are provider, license, coverage, or deployment dependencies. The runtime
refuses the stronger claim until evidence is added to a new qualification version.

## N. Recommendation

**READY FOR INDEPENDENT FOUNDATION / PHASE 2.5 FREEZE REVIEW**

Do not start Phase 2.6 as part of this checkpoint.
