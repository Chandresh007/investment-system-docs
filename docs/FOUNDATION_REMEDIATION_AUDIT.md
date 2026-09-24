# Foundation remediation audit

## 2026-09-22 qualification closure

This document records the preceding remediation checkpoint. The subsequent
[final foundation qualification review](FINAL_FOUNDATION_QUALIFICATION_REVIEW.md)
closes the remaining accidental-trust paths. Runtime capabilities now distinguish
current research, historical replay, raw reproducibility, action PIT, universe and
security coverage, delisted/lifecycle support, survivorship safety, macro vintages,
SEC filing PIT, and verified rights. Yahoo, ordinary FRED, Company Facts aggregate,
and live ALFRED cannot obtain historical or production labels from documentation
alone. Development overrides are explicit provenance and force `DEVELOPMENT`.

Additional implemented boundaries: SEC acceptance/next-day availability and
comparative/amendment tests; original versus reconstructed raw lineage; read-only
reconciliation of 34 missing test receipts; permanent bitemporal security identity;
S3-compatible encrypted/checksummed manifest-last upload; versioned provider
records. Current regression: 428 passed, zero failures; Alembic fnd005. The updated
checkpoint verdict is **PASS for enforced foundation risk closure**, with live
provider/license/deployment qualification externally blocked. Phase 2.6 remains
not started.

Baseline: clean master/origin `125e780c2731dd831ebbdd69abf92332e5578dde`.
Baseline suite: 378 passed, 27 warnings. Phase 2.6 NOT STARTED. No new feature
families; frozen formulas remain unchanged.

## Membership semantics

`HistoricalUniverse.start_date/end_date` are economic [start, end) boundaries,
aliased as effective_start/effective_end. `available_at` is justified knowledge
of a particular version, `retrieved_at` is receipt, and `timing_policy` distinguishes
AS_OBSERVED (availability cannot precede receipt), CERTIFIED_PUBLICATION (qualified
historical reconstruction), and SYNTHETIC (tests only). No observation date or
legacy created_at is promoted to historical knowledge. Legacy rows stay nullable
and are excluded from PIT readers until new evidenced rows are appended.

An evidence key identifies one membership episode/classification segment within
source + universe + stable security. A revision increments membership_version and
replaces that entire segment. Removal appends a version with an exclusive end;
re-entry is a new episode. Classification changes close the old segment with a
new version and introduce a new segment. Publication must insert all related
segments in one transaction. A correction retains its segment key and a later
knowledge time. Conflicting overlaps fail explicitly, never duplicate basket weight.

`HistoricalUniverseResolver.known_at(T)` can expose announced future membership.
`effective_on(session, T)` first resolves versions known by T, then filters economic
intervals, then classifications. Reversing that order resurrects removed members.
The market sector/industry lookup and each daily industry basket use this resolver.
The mean-log-return calculation and every frozen market formula are unchanged.

Stable security IDs, not today's is_active, drive membership. Delisted names remain
usable; FK references prevent deleting referenced securities. Current ticker
lookup for benchmark ETFs is still a coverage/identifier qualification limitation.
No complete historical universe provider, ticker crosswalk, merger proceeds or
failed-company coverage is claimed. Historical evidence must be pinned in a build;
certified backfills are new research builds, not proof of operational receipt at T.

Checkpoint validation: 20 membership + existing market tests passed. Migration
fnd001 applied successfully; PostgreSQL blocks membership UPDATE, retaining
original evidence. Deletes remain administrative operations under the legacy
owner-role deployment; operational least-privilege roles are still required.

## Raw identity, publication and builds

RawStorage now atomically publishes SHA256-addressed bytes using a same-directory
fsynced temporary file and exclusive hard link. Identical bytes share storage;
different bytes have distinct keys regardless of second/source/identifier. Each
legacy receipt remains an audit row, including repeated identical downloads;
estimates retain their stronger exact-receipt row deduplication. Reads verify
SHA256, size and root containment; missing/tampered objects fail, never self-repair.
SEC, Yahoo, FRED and RSS receipts now include reconstructable public request
metadata; credentials are excluded. Existing historical receipts are not repaired
or retroactively supplied with invented request information.

Fundamental and market FeatureValue publication compares an existing identity's
value/status/evidence instead of UPDATE or silent conflict-ignore. Conflicts require
a new calculation_version (which may include a manifest build ID). PostgreSQL also
rejects UPDATE and duplicate nullable-period identities. Exact reruns reuse rows.
Estimates already enforce build-specific conflicts and their formulas are unchanged.
Administrative DELETE/owner access is not an immutable prediction ledger; production
roles and a future sealed publication ledger remain separate operational gates.

ResearchBuildManifest provides canonical JSON + SHA256 identity, including full
Git commit, schema, datasets/raw inventory, provider/normalization/feature/calculation
and dependency versions, configuration and UTC research timestamp. Wall-clock
execution time is excluded. The manifest is portable and hash-verified on reload;
legacy runner callers must explicitly use a build-qualified calculation_version.
This is reusable infrastructure, not a claim that old runs were retrospectively
pinned or that every existing ingestion table is a versioned dataset.

## Market and macro checkpoint

Yahoo chart is used directly; yfinance is not installed as the connector. The
normalizer previously searched for adjusted close inside quote instead of the
sibling indicators.adjclose array. It now preserves that field correctly, records
PROVIDER_REPORTED_UNQUALIFIED price/volume basis and yahoo_chart_v2 normalization,
and uses the original raw receipt timestamp. Epoch/request conversions use UTC.
The frozen engine still consumes close + known actions, never adjusted_close.
Legacy price rows remain unqualified, not relabeled raw. Price revisions remain
in raw receipts; the legacy canonical first-observation table is not a versioned
institutional price archive. That is an explicit provider-replacement prerequisite.

EconomicVintage is a separate canonical append-only table with source/build,
series, observation date, realtime_start, exact Decimal or missing value,
availability, receipt, timing policy, normalization version and raw FK. The
SyntheticMacroVintageConnector validates archived bytes and preserves successive
releases; the resolver pins source/build and selects the latest known vintage.
January 15 sees 2.0; February 15 sees 2.5; later receipt does not impersonate past
system receipt. SYNTHETIC is explicit. Live FRED remains latest-view context;
a qualified ALFRED adapter, release timing and coverage checks are next provider work.

SourceQualification/require_historical_replay rejects current-only,
historical-as-seen-today, unqualified and unproven archived sources. Only scoped
PIT qualifications with version/evidence/coverage and explicit synthetic permission
pass. All existing live adapters fail admission by default. This is the required
future replay boundary; development calculators remain available and are not a
certified backtester.

Focused validation: 36 foundation/market tests passed; 22 market, cache,
fundamental integration and full synthetic estimates integration tests passed.
Migrations fnd002/fnd003 applied. Live provider qualification remains incomplete.

## Corporate actions and survivorship

Reviewed CorporateActionResolver and MarketAdjustmentEngine directly. Only actions
with coalesce(available_at, created_at)<=T enter normalization. Effective dates
control split basis and dividend sessions; current Security.is_active is not a
historical price or membership filter. Existing split-announced-before-effective,
late-known split, reverse-split/dividend, delisted-history and stale-session tests
remain unchanged in their assertions. Only membership fixture evidence was made
explicit; arithmetic was not altered.

Historical companies/securities remain relational IDs; referenced rows cannot be
deleted through existing FKs. Current collection may restrict active securities,
which does not establish historical coverage. Current ticker-based benchmark
lookup and metadata ingestion do not implement a bitemporal identifier crosswalk.
Ticker reuse/renaming, merger successor identity, spinoff allocations and delisting
proceeds therefore remain unqualified; silently stitching by ticker is prohibited.
The historical resolver handles industry changes but does not invent missing firms.

## Recovery implementation and evidence

Scripts: backup_research_data.sh, restore_research_data.sh, verify_backup.sh;
shared implementation research_backup.py; disposable lifecycle verifier
verify_foundation_recovery.py. Credentials remain in environment/private config.
Local destinations must be new and outside the checkout. Compressed pg_dump and
row hashes/counts/raw inventory share one exported REPEATABLE READ snapshot.
Every raw DB reference must validate before COMPLETE. Restore accepts no existing
DB name: it always generates a new database and requires a fresh raw root.
Verification restores, compares all table contents/counts, raw bytes and schema,
runs Alembic current and an as-of feature query with a nonempty FCF=80 fixture,
and drops only its generated DB. Missing/tampered dump/raw/manifest and path escape
are tested before database creation. No persistent database was downgraded/restored.

Two intermediate disposable drills passed through fnd003 then fnd004, including
base→head→25e001→head and exact restored feature queries. Final checks are recorded
in PROJECT_STATE. The drill intentionally corrupts its disposable dump to test
rejection; its temporary artifacts are not backups of actual research data.
No cloud copy, encryption configuration, schedule or persistent-data restore is
claimed. At this checkpoint an existing-store audit found 32 raw metadata rows
(SEC/RSS) unverifiable under the configured data root. The later final audit found
34 after two more legacy test receipts and stopped future growth. They were retained. Some legacy tests leave raw
metadata referencing temporary/nonexistent files, so fixture residue and genuine
research receipts must be reconciled before a persistent backup can be trusted.

## Risk verdicts

| Original risk | Verdict | Scope and remaining work |
| --- | --- | --- |
| Historical universe knowledge time (R1) | RESOLVED for evidence/query semantics | Separate effective/knowledge/receipt times, monotone immutable versions, PIT readers and adversarial tests; real coverage is separate |
| Yahoo/market PIT (R4) | PARTIALLY RESOLVED | Explicit unqualified basis and fixed boundary; frozen formulas correct conditional on inputs; qualify versioned prices/actions, dividends' share basis and terminal outcomes |
| FRED vintage history (R5) | PARTIALLY RESOLVED | Canonical immutable vintage + synthetic raw-to-replay architecture; live ALFRED/release/coverage adapter remains next provider task |
| Raw storage reproducibility (R7) | PARTIALLY RESOLVED overall; new writes RESOLVED | Atomic content keys, verified reads and immutable metadata; missing legacy bytes cannot be recreated honestly |
| Feature history immutability (R6) | RESOLVED for recalculation | Identical reruns reuse rows, conflicting content fails, new builds coexist, SQL UPDATE blocked and ambiguous reads require version pin; owner-role deletion and future prediction ledger remain outside this guarantee |
| Survivorship bias (R1 coverage) | PARTIALLY RESOLVED | Delisted stable IDs retained and effective/PIT selection repaired; complete failed-firm coverage/crosswalk/merger economics unproven |
| Data durability (R9) | PARTIALLY RESOLVED | Consistent local backup and disposable restore proven; actual persistent archive reconciliation and encrypted off-host operation remain open |

R2 SEC same-day midnight availability and R3 comparative fiscal-context qualification
remain **UNRESOLVED**. This task did not change frozen fundamental formulas or
pretend to qualify their source extraction. R8 production estimates remains
**PARTIALLY RESOLVED** from the earlier synthetic implementation; real provider,
coverage/rights, publication orchestration and independent freeze review remain.
The source-admission boundary prevents claiming these paths are certified replay.

Overall verdict: **CONDITIONAL PASS** for implemented foundation safeguards, not
production historical research. Recommendation: **MORE FOUNDATION REMEDIATION
REQUIRED** before feature expansion. Prioritize SEC timing/context, persistent raw
inventory reconciliation and verified off-host backup; qualify a survivorship-safe
membership/identifier archive and raw-price/action contract. Live ALFRED can remain
excluded until qualified. Independent Phase 2.5 freeze review is still a separate
useful review, but this remediation does not grant that freeze or start Phase 2.6.

## Review corrections during implementation

The first full regression caught the new raw-integrity exception escaping the
estimate archive's established RAW_CONTENT_HASH_MISMATCH contract. The archive now
translates the shared integrity exception, preserving all original assertions;
no estimate formula changed. Request identifiers are retained in estimate receipt
metadata for reconstruction. Request hashes now canonicalize parameter ordering
and exclude credentials, so cache identity is stable across key rotation; old
cache entries simply miss and are not rewritten.

Membership evidence requires raw provenance for non-synthetic policies. Database
append guards reject reused/backwards version/knowledge times; related removal
and classification segments remain an atomic publisher responsibility. Market
relative-feature provenance now includes selected membership IDs and knowledge
cutoff. Feature SQL UPDATE and raw metadata UPDATE are rejected. Explicit build
selection prevents a new calculation version becoming an arbitrary earlier result.

Existing tests using owner-level fixture cleanup still delete legacy rows. These
DB guards are not protection from a database administrator or disk administrator;
production writer roles must deny DELETE/TRUNCATE and schema modification. A future
sealed research ledger should preserve its own evidence/build references. Frozen
formula source paths and existing numerical expectations remain unchanged.


## Final validation

Full suite: **411 passed, 8 existing warnings, zero failures**, 103.45s. Final
foundation suite on disposable PostgreSQL: **33 passed**. Alembic current/head:
**fnd004**. Final lifecycle/recovery drill ran from committed 36980d9 with the full
migration cycle and FCF=80 as-of query; original/restored content hashes matched,
and deliberately corrupted dump rejection passed. Final report:
`/tmp/irs-foundation-drill-7lmfyqq7/restored_raw/restore_report.json` (synthetic).

`git diff --check` and changed Markdown local links passed. Scoped model/migration
inspection found only the pre-existing sector/industry index omissions on
HistoricalUniverse, not newly introduced schema drift. Frozen calculator,
adjustment engine, price/action resolver and all research/estimates code have no
diff from the starting checkpoint. There is no investment performance claim.

The final documentation checkpoint is discoverable with:
`git log -1 --format='%H' --grep='^Record foundation remediation validation and remaining gates$'`.
