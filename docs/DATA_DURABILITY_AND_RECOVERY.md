# Data durability and recovery

Status: **local backup/restore and S3-compatible off-host upload tooling
implemented; disposable PostgreSQL recovery verified**. Live cloud credentials,
retention enforcement, scheduling, and a remote restore drill are not yet proven.
See the [final qualification review](FINAL_FOUNDATION_QUALIFICATION_REVIEW.md).

## What belongs in Git

Git contains code, migrations, documentation, small synthetic/golden fixtures,
verification tools and permitted redacted manifest summaries. This checkpoint's
fixtures are synthetic; no public-data redistribution right is assumed.
Git does not contain PostgreSQL dumps, large raw datasets, licensed proprietary
data, secrets or full restricted inventories. Backup destinations inside the
checkout are rejected. Dumps and backup directories are additionally ignored.

## Implemented commands

Activate the repository venv. Set DATABASE_URL privately using ignored dotenv or
process configuration. Credentials are not printed or included in manifests.
Install PostgreSQL client tools matching the server; the tested server is 16.
For the documented local Docker development DB, tools may instead run inside its
container with `export IRS_PG_TOOLS_CONTAINER=irs_postgres`. Named environment
variables carry credentials, not command-line values. Container tool mode assumes
the configured host/port are reachable from that container; SSL client paths and
remote libpq service configuration should use native client tools.

Choose fresh destinations outside Git and on a durable volume:

```bash
source .venv/bin/activate
scripts/backup_research_data.sh /mnt/research-backups/checkpoint-001 \
  --raw-root /mnt/research-data \
  --research-timestamp 2026-09-21T00:00:00+00:00

# This verifies bytes, creates a NEW random disposable DB, restores and checks it,
# then drops only that generated DB. The new raw directory/report is retained.
scripts/verify_backup.sh /mnt/research-backups/checkpoint-001 \
  --raw-destination /mnt/research-drills/checkpoint-001

# Restore for inspection/recovery: always creates a NEW random database,
# never accepts an existing DB name, and leaves it available for operator review.
scripts/restore_research_data.sh /mnt/research-backups/checkpoint-001 \
  --raw-destination /mnt/research-restores/checkpoint-001
```

No automatic persistent restore, overwrite, cutover or production drop exists.
Both commands refuse an existing raw destination and require CREATEDB privileges.
Verify drops only its own `irs_restore_<random>` database, including on failure;
restore retains its generated database on failure for diagnosis. Artifact hashes
are checked before any database is created. Restore only trusted backups: checksums
prove integrity against the manifest, not publisher authenticity or SQL safety.

`python scripts/research_backup.py verify-files BACKUP_DIR` checks artifact bytes
without connecting to PostgreSQL. `IRS_PYTHON` may override the shell wrappers'
venv interpreter. There is no default GitHub destination or embedded credential.

For a protected S3-compatible copy, configure the destination and use a dry run
before the live upload:

```bash
export IRS_BACKUP_S3_URI=s3://private-research-backups/foundation
export IRS_BACKUP_S3_SSE=AES256
# Optional: IRS_BACKUP_S3_ENDPOINT_URL=https://s3-compatible.example
# For KMS: IRS_BACKUP_S3_SSE=aws:kms and IRS_BACKUP_S3_KMS_KEY_ID=...

scripts/upload_backup.sh /mnt/research-backups/checkpoint-001 --dry-run
scripts/upload_backup.sh /mnt/research-backups/checkpoint-001
```

Credentials come from the normal AWS credential chain and are never command
arguments, manifests, or Git content. Custom endpoints must use HTTPS. The uploader
verifies the complete local artifact, requests SHA-256 and server-side encryption,
checks remote size/hash metadata plus API checksum when configured, reuses exact
existing objects, and creates absent objects with atomic `If-None-Match: *` writes,
so a concurrent writer cannot be overwritten. It performs no deletion and uploads
`manifest.sha256` last. The safe conditional path deliberately rejects any single
artifact above 5 GiB; qualify a conditional multipart implementation before the
archive reaches that size. Set `IRS_BACKUP_S3_REQUIRE_API_CHECKSUM=false` only for an
S3-compatible service that cannot return API checksums; size and immutable SHA-256
metadata are still verified, and that weaker live service must be documented.

## Snapshot and manifest guarantees

A PostgreSQL REPEATABLE READ transaction exports one snapshot. The compressed
custom-format pg_dump uses that snapshot while the same transaction captures all
public-table row counts/content hashes, schema revision, build IDs, raw inventory
and a deterministic as-of FeatureValue query pinned per calculation identity.
Each referenced raw file must match its DB SHA256 and size before copying. New raw
objects arriving after the snapshot are irrelevant. Old missing/corrupt objects
fail closed; backups never silently omit their metadata or invent replacement bytes.
The content-addressed writer publishes bytes before DB metadata, so concurrent
new receipts cannot expose half-written files through a committed raw row.

The small manifest records backup timestamp, Git commit/dirty-state flag,
PostgreSQL version, Alembic revision, dump SHA256, raw object count and canonical
inventory hash, per-object source/path/SHA256/size/archive/recovery lineage,
per-table counts/hashes, estimate/macro datasets and market source
versions, replay timestamp and query digest. Full inventories remain with backups.
Execution timestamps are operational metadata, not ResearchBuildManifest identity.
The dump/objects/manifests are flushed before the completion checksum is published.
Partial directories are not COMPLETE and must be inspected, not overwritten.

Restoration verifies all hashes, restores into a newly generated empty database
with pg_restore --exit-on-error --single-transaction, then compares schema revision,
every table count/content digest, every restored raw DB reference and the selected
as-of research query. It also runs `alembic current`. It does not upgrade a restored
DB before proving faithful restoration. Test an upgrade separately afterwards.

The manifest's Git commit and dirty flag describe backup tooling provenance.
Research artifacts need their own clean-code ResearchBuildManifest, dependency
versions and input inventory; a backup alone does not retrospectively qualify old
runs. Table content hashing sorts every row: suitable for this foundation, not yet
benchmarked for very large archives. Large systems will need streamed/versioned
object inventories and a qualified consistent incremental backup policy.

## Executed drill and remaining data gap

`IRS_PG_TOOLS_CONTAINER=irs_postgres python scripts/verify_foundation_recovery.py`
creates a disposable source DB, runs base→head→25e001→head migrations, foundation
tests, seeds a deterministic FCF=80 plus membership/macro/raw evidence, backs it up,
restores to another generated DB, verifies counts/hashes/query and tests corrupted
dump rejection. Only generated databases are dropped. Temporary synthetic reports
remain in `/tmp/irs-foundation-drill-*`; these are test artifacts, not disaster backups.

The final clean-checkout run at
`0e6977d338ee13240876e1f0766dffe4c752799a` passed through `fnd005`, including
50 foundation tests and an isolated restore. Its synthetic report is
`/tmp/irs-foundation-drill-w34u713u/restored_raw/restore_report.json`. The path is
ephemeral validation evidence; the retained, deliberately corrupted dump is not a
usable backup.

A read-only final audit found 34 existing raw metadata rows with missing files in
the configured local store. All are test-only `/tmp` receipts with no hash/size.
The preceding count of 32 grew because a legacy test committed a new row on each
full run; that test now removes its own receipt. Existing rows remain untouched and
are cataloged in [the reconciliation manifest](LEGACY_RAW_RECONCILIATION.json).
Passing the complete synthetic drill does not prove that persistent archive can
be recovered. Reconcile actual research receipts from trusted originals, distinguish
fixture residue, and take/verify a complete persistent backup before claiming recovery.
Do not silently delete evidence or rewrite stored hashes to make verification pass.

## Off-host live qualification still required

Use least-privilege managed credentials, bucket versioning, retention/object-lock
protection where permitted, and an independently recoverable key/account strategy.
The uploader supplies the transfer and integrity boundary; no live bucket was used
in tests and no off-host copy is yet claimed. Licenses determine retention, backup,
and redistribution permissions.

Deployment policy should retain at least 14 daily, 8 weekly, and 12 monthly verified
copies, adjusted for license/volume/cost. Also take a protected backup before a major
migration, data-provider migration, or schema/storage rewrite. Objectives remain
RPO ≤24 hours, RTO ≤8 hours, monthly restores, and an extra drill after storage or
schema changes. Retention belongs in bucket lifecycle/IaC rather than this script.
Scheduling, alerting, remote restore evidence, role separation, and key recovery are
the next operational tasks. The project owner is the recovery custodian until another
owner is explicitly assigned.

On a new host: clone the manifest's code version, install the environment, verify
backup bytes, restore into an isolated DB/raw root, run Alembic current, compare
counts and hashes and research build identity, then run deterministic tests in a
separate test database. Review any upgrade/cutover explicitly. Preserve old storage
for incident investigation. EBS snapshots can supplement, not replace, these checks.
