# EC2 development machine setup

This guide reconstructs the development environment without changing research
semantics. Phase 1, 2.1 and 2.2 are complete; Phase 2.4 is complete/frozen.
Foundation and Phase 2.5 are frozen for implemented semantics after independent
review. Phase 2.6 is ready to start and has not yet begun.

## EC2 assumptions

Prefer Ubuntu 24.04 LTS, which provides Python 3.12 through Ubuntu packages.
The replacement machine validated on 2026-09-20 runs Ubuntu 26.04.1 LTS,
whose system Python is 3.14.4; use the separate source-build fallback below.
A practical development allocation is 2 vCPU, 4 GiB RAM and at least 30 GiB gp3
storage, increasing disk capacity for raw data. This bootstrap ran on 2 vCPU,
2 GiB RAM and a 28 GiB root filesystem; restrict compilation to two jobs.

Only inbound SSH (TCP 22) from your administrator IP is needed, or use AWS
Systems Manager with no inbound SSH. Use SSH keys and keep private keys private.
Never allow public inbound TCP 5432 or Docker TCP ports 2375/2376. Compose binds
PostgreSQL to loopback only. No AWS security groups are modified by setup.
Docker uses its local Unix socket. Docker group membership grants root-equivalent
host access; grant it only to trusted development users.

## Base packages and GitHub CLI

```bash
sudo apt-get update
sudo apt-get install -y git gh ca-certificates curl build-essential pkg-config xz-utils
# Authentication requires the operator's own GitHub identity; never paste tokens into Git.
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
```

The replacement host already had GitHub CLI 2.101.0 from the official GitHub
package repository; the Ubuntu package is sufficient for the commands here.

Existing authenticated machines can use `gh auth status` instead of logging in
again. Never put token output in build logs or documentation.

## Clone or update

```bash
mkdir -p ~/Document
cd ~/Document
gh repo clone Chandresh007/investment-research-system
cd investment-research-system
git status --short
git fetch origin
git checkout master
git pull --ff-only origin master
git rev-parse HEAD
git rev-parse origin/master
git log --oneline -10
git diff --check
```

For an existing clone, start with `cd ~/Document/investment-research-system` and
omit cloning. Inspect unexpected changes before proceeding; never reset them
blindly. Read README, PROJECT_STATE, the blueprint, learning roadmap and Phase 2.4
specification. PROJECT_STATE records the accepted freeze more accurately than
older planned-status text in README/specification.

## Python 3.12

`pyproject.toml` declares `requires-python = ">=3.12"`. This is a lower bound,
not a guarantee that every later Python version was validated. Use 3.12 to match
the previous machine. There is no CI Python matrix, Docker application runtime,
lockfile or alternate dependency manager in the original checkpoint.

On Ubuntu 24.04:

```bash
sudo apt-get install -y python3.12 python3.12-venv python3.12-dev
python3.12 --version
```

On the validated Ubuntu 26.04 host, `apt-cache policy python3.12
python3.12-venv python3.12-dev` returned no candidates. Do not mix Ubuntu release
repositories or replace `/usr/bin/python3`. Build the official source release
under a versioned prefix instead. This provides venv, ensurepip and development
headers, the equivalents of the three packages above.

```bash
sudo apt-get install -y libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
  libsqlite3-dev libffi-dev liblzma-dev libncurses-dev uuid-dev libgdbm-dev libexpat1-dev
build_dir=$(mktemp -d /tmp/irs-python.XXXXXX)
cd "$build_dir"
curl -fLO https://www.python.org/ftp/python/3.12.14/Python-3.12.14.tar.xz
echo '5c8462af5790baf43a321a1559dbe0db06d1be4300fb85fb53c40060668e548a  Python-3.12.14.tar.xz' | sha256sum -c -
tar -xf Python-3.12.14.tar.xz
cd Python-3.12.14
./configure --prefix=/opt/python/3.12.14 --with-ensurepip=install
make -j2
sudo make altinstall
# Optional version-specific command on the normal PATH; never replace python3.
if ! test -e /usr/local/bin/python3.12; then
  sudo ln -s /opt/python/3.12.14/bin/python3.12 /usr/local/bin/python3.12
fi
export PATH="/opt/python/3.12.14/bin:$PATH"
python3.12 --version
python3.12 -c 'import ssl, sqlite3, bz2, lzma, ctypes, venv; print("runtime modules OK")'
cd ~/Document/investment-research-system
```

Version/checksum source: [official Python 3.12.14 release](https://www.python.org/downloads/release/python-31214/).
The source installation is not maintained by apt: review official 3.12 security
releases, install upgrades in a new prefix and recreate the venv deliberately.

## Virtual environment and authoritative dependencies

```bash
cd ~/Document/investment-research-system
# Inspect an existing environment before replacing it.
python3 --version
if test -x .venv/bin/python; then .venv/bin/python --version; fi
# Only when replacing an incorrect/broken venv, preserve it outside the repository:
if test -d .venv; then mv .venv "../irs-venv-backup-$(date -u +%Y%m%dT%H%M%SZ)"; fi
python3.12 -m venv .venv
source .venv/bin/activate
python --version
which python
python -m pip install -e .
python -m pip check
```

The setuptools project in `pyproject.toml` is authoritative. Pytest is already a
project dependency; no `dev` extra exists. Bootstrap adds previously omitted
runtime dependencies NumPy (`research/calculators.py`), feedparser (RSS connector)
and python-dateutil (news date normalization) to that same declaration. Do not
chase missing imports with one-off installations. Version bounds are not a lock;
a later installation may resolve newer versions. The validation record below
identifies the versions tested here.

Editable installation resolves `investment_research` to this checkout even
without `PYTHONPATH`. Pytest also sets `pythonpath = ["src"]`. For the historical
project convention and explicit precedence over stale installs:

```bash
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
python - <<'PY'
import numpy, feedparser, sqlalchemy, alembic, polars, typer, structlog
import investment_research
print('numpy', numpy.__version__)
print('feedparser', feedparser.__version__)
print('sqlalchemy', sqlalchemy.__version__)
print('source', investment_research.__file__)
PY
```

The source path must end in this checkout's `src/investment_research/__init__.py`.
Run commands from the repository root. No global PYTHONPATH modification is needed.

## Environment variables

Create a private local file only if absent:

```bash
if ! test -e .env; then (umask 077; cp .env.example .env); fi
chmod 600 .env
```

Edit `.env` privately; it is ignored by Git. Never store configured identities or
keys in `.env.example`. `database/config.py` loads dotenv without overriding
existing process environment; Alembic imports that module too. Do not print
`.env` or `docker compose config` into shared logs.

| Name | Purpose and requirement |
| --- | --- |
| `DATABASE_URL` | PostgreSQL SQLAlchemy URL; required for nondefault DBs. Code otherwise defaults to the local Compose development database. |
| `SEC_USER_AGENT` | Required monitored operator identity for live SEC commands; loaded through central project configuration. |
| `FRED_API_KEY` | Required for live FRED requests; optional for local tests, which use mocks. |
| `DATA_ROOT` | Optional raw-storage root, defaults to `data` relative to the working directory. |
| `LOG_LEVEL` | Example-file convention only; current logger accepts a function argument and does not read this variable. |
| `PYTHONPATH` | Optional explicit local-source precedence, as above. |

Example identity/key placeholders, not real credentials:

```dotenv
DATABASE_URL=postgresql://postgres:postgres@127.0.0.1:5432/investment_research
SEC_USER_AGENT="investment-research-system your-real-email@example.com"
FRED_API_KEY="..."
DATA_ROOT=data
LOG_LEVEL=INFO
```

Compose's postgres/postgres pair is a local development default, not a production
credential. No additional provider keys are currently consumed by Yahoo/RSS.
Missing live-provider credentials must not block unit tests.

## Docker and PostgreSQL 16

The validated machine already had Ubuntu's Docker packages. On fresh Ubuntu,
use its repository packages rather than mixing Docker distributions:

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
# For an existing shell after adding the group:
newgrp docker
```

A new login also refreshes group membership. Continue from the repository root
and reactivate the venv if necessary. On hosts deliberately using Docker CE,
follow [Docker's official Ubuntu instructions](https://docs.docker.com/engine/install/ubuntu/)
instead; do not install conflicting Ubuntu packages over CE.

```bash
docker --version
docker compose version
docker ps
cd ~/Document/investment-research-system
docker compose up -d db
docker compose exec -T db pg_isready -U postgres -d investment_research
docker compose exec -T db psql -U postgres -d investment_research -c 'SELECT version();'
docker ps
```

Wait for `pg_isready` to report accepting connections. The existing Compose
service is `db`, container `irs_postgres`, image `postgres:16-alpine`, persistent
volume `postgres_data`. Host publication must read `127.0.0.1:5432->5432/tcp`.
Never run `docker compose down -v` as a repair: that destroys the database volume.
Recreating the container to change its port binding preserves the named volume.
The image tag follows PostgreSQL 16 updates; it is not an immutable image digest.

## Alembic and connectivity

```bash
source .venv/bin/activate
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
alembic heads
alembic current
alembic upgrade head
alembic current
alembic history
python - <<'PY'
from sqlalchemy import text
from investment_research.database.config import engine
with engine.connect() as connection:
    print('DB connectivity:', connection.scalar(text('SELECT 1')))
PY
```

Expected current single head: `fnd006`; older validation records below retain their
historical revision numbers. Use the full Alembic
chain to create schema, never hand-written CREATE TABLE repairs. Migrations do
not restore companies, financial facts, market prices, or raw response files.

## SEC configuration and current limitations

[SEC developer guidance](https://www.sec.gov/about/developer-resources) requires
an identifying User-Agent and limits aggregate automated access to 10 requests
per second. Use a real monitored email supplied by the operator.

`investment_research.config` loads private dotenv settings without overriding
process environment. At SEC command execution, `get_sec_user_agent()` reads
`SEC_USER_AGENT`, strips surrounding whitespace and rejects missing/blank values.
The CLI passes that value to `SECConnector`; `BaseConnector` installs it as the
HTTP `User-Agent` header. The variable name matches `config/sources.yaml`.

Both `sec-universe` and `sec-company` fail with a configuration error (exit 2)
before constructing a connector or requesting data if identity is missing.
There is no placeholder fallback. Non-SEC commands and mocked unit tests do not
require operator identity. The operator remains responsible for supplying a real,
monitored contact; no email is hardcoded into application source.

Each SEC request path sleeps 0.11 seconds before sending, keeping sequential
requests below approximately 9.09 requests/second plus network overhead. This is
not a shared limiter across threads, processes or machines. Run only one SEC
worker from this host/IP; concurrent workers could exceed the aggregate ceiling.
No limiter behavior was changed and no live SEC request was needed for bootstrap.

## Tests and external dependencies

```bash
source .venv/bin/activate
export PYTHONPATH="$PWD/src${PYTHONPATH:+:$PYTHONPATH}"
python -m pytest -q
# Separately report the unit suite (some tests still depend on database state):
python -m pytest -q tests/unit
```

Historical baseline: **72 passed, 7 warnings**. After the reproducibility
cleanup, the suite contains 83 tests: 81 unit tests and two integration tests.
Expected results are **81 passed, 8 warnings** for the unit suite and
**83 passed, 8 warnings** for the full suite, with zero failures. Warnings are
existing datetime UTC and SQLAlchemy Query.get deprecations; importing the CLI
for the added configuration tests exposes one additional existing warning.

Some unit files use real PostgreSQL and commit synthetic data: fundamental
features, fundamental resolution, news pipeline, normalization and research
foundation. Market feature tests use in-memory SQLite. Connector HTTP calls are
mocked; live SEC/FRED/Yahoo/RSS network and real provider credentials are not
required. Use a dedicated development/test database: existing news fixtures
clear news tables, and some other fixtures commit test data.

`tests/integration/test_nvda_features.py` is now a deterministic synthetic
integration test, despite its historical filename. Its fixture creates a company,
security and explicit FinancialFact rows with generated IDs. It preserves the
original fiscal/PIT case: two 90-day Q1 facts with the same calendar period end,
different fiscal years, and different filing dates. It checks missing-current,
missing-prior, the exact filing boundary, valid growth/provenance, and rejection
of a 181-day Q2 YTD fact as a standalone quarter. The full-run test requires
persisted feature rows and checks numeric values and provenance; it cannot pass
vacuously on an empty dataset. An outer transaction and a savepoint-bound Session
roll back fixture data and runner commits after each test.

`test_freshness_triggers_fetch` and `test_freshness_skips_fetch` now explicitly
supply the runner's securities query through a mocked SessionLocal. Cache tests
also mock Yahoo's raw-object bookkeeping session. They neither require NVDA in
PostgreSQL nor depend on a news test running first. Both tests check that the
freshness query actually ran.

No historical database restoration, live ingestion or undocumented seeded rows
are required for deterministic correctness. A backup is useful for restoring
research data, not for making this test suite pass.

## Troubleshooting

| Symptom | Diagnosis and action |
| --- | --- |
| Wrong Python version | Inspect `.venv/bin/python --version` and `pyvenv.cfg`; recreate with Python 3.12, keeping system Python intact. |
| Missing numpy | Reinstall the complete current project with `.venv/bin/python -m pip install -e .`; NumPy is now declared. |
| Missing feedparser | Same authoritative install; do not fix only one import. |
| Stale site-packages import | Check `investment_research.__file__`, interpreter path and editable install; reinstall from this checkout. |
| PYTHONPATH issue | From repo root prepend `$PWD/src` using the command above; avoid pointing at an old checkout. |
| PostgreSQL unavailable | Check Docker permissions, `docker compose ps`, `pg_isready` and local port 5432. Preserve volumes. |
| Missing DATABASE_URL | Set it in ignored `.env`; the default works only with the local Compose defaults. Process environment overrides dotenv. |
| Empty new DB | Run Alembic to create schema; tests provide their own prerequisites. Restore backups only to recover actual research data. |
| Missing FRED key | Obtain your own FRED key for live ingestion. Mocked tests do not need it. |
| Missing SEC_USER_AGENT | Supply your real monitored email privately; SEC commands fail clearly without it. Non-SEC tests need no email. |
| Docker permission denied | Ensure daemon is running and ubuntu belongs to docker; refresh login or use `newgrp docker`. Do not chmod the socket world-writable. |

## Git validation and scope

```bash
git status --short
git diff
git diff --check
git check-ignore .env .venv/
```

Review each intended configuration, test or documentation change before staging.
Never stage `.env`, credentials, raw data, logs, database volumes, or venv contents.
Migration scripts and frozen Phase 2.2/2.4 calculation semantics are unchanged.

## Bootstrap automation assessment

System installation varies between Ubuntu releases and requires operator-owned
GitHub/SEC/FRED identities; automatic venv replacement could also discard useful
state. The explicit commands above are the bootstrap interface for this change.
No automated setup script is added. This avoids hidden global changes, accidental
`.env` overwrites and accidental database destruction.

## Original bootstrap record (2026-09-20, before reproducibility cleanup)

Starting checkpoint: `73c2e94881f366faba44f3de20bf705a0bbdda20`.
The initial working tree had identity/key edits in `.env.example`. They were
preserved in ignored, mode-600 `.env`; no real identity or key was committed.
The tracked example now remains portable and describes the current wiring.

| Check | Observed result |
| --- | --- |
| System Python | `/usr/bin/python3`: 3.14.4, unchanged |
| Old venv | 3.14.4; preserved outside checkout at `~/Document/irs-venv-backup-python314-20260920` |
| New venv | Python 3.12.14 from `/opt/python/3.12.14` |
| Install | `python -m pip install -e .`; `pip check` passed |
| Imports | All requested imports passed; editable source path correct with and without PYTHONPATH |
| Docker / Compose | 29.1.3 / 2.40.3+ds1-0ubuntu1; ubuntu can use Docker without sudo |
| PostgreSQL | 16.15, running; loopback-only TCP 5432, SELECT 1 passed |
| Alembic | Single head/current `1f4b7c9d2a31`; existing DB upgrade successful/no-op |
| Fresh schema | All 11 migrations successfully replayed in a newly created disposable validation DB; only that disposable DB was removed afterwards |
| First full test run on empty research DB | 70 passed, 2 failed, 5 warnings |
| Subsequent unit suite | 70 passed, 0 failed, 7 warnings |
| Final full suite | 71 passed, 1 failed, 7 warnings |
| Provider configuration | DATABASE_URL, SEC_USER_AGENT, FRED_API_KEY, DATA_ROOT and LOG_LEVEL present in private dotenv; external key validity and monitored-email ownership not verified |

Installed versions: NumPy 2.5.3, feedparser 6.0.14, python-dateutil 2.9.0.post0,
SQLAlchemy 2.0.54, Alembic 1.20.0, Polars 1.44.2, Typer 0.27.2,
structlog 26.1.0, pytest 9.1.1, HTTPX 0.28.1, Pydantic 2.13.5,
PyArrow 25.0.1, DuckDB 1.5.5 and psycopg2-binary 2.9.13.

At the original bootstrap checkpoint, the outstanding failure was:
`tests/integration/test_nvda_features.py::test_nvda_fundamental_pit_correctness`,
line 50: expected `missing_prior_period`, received `missing_current_period`.
The first run failed at line 20 (`NVDA security not found in database`); later
synthetic tests populated database IDs without restoring the old NVDA facts.
The first run also failed
`tests/unit/test_market_data_cache_freshness.py::test_freshness_triggers_fetch`
with an expected-one-call/actual-zero assertion because ticker NVDA was absent.
No tests were skipped or changed, and no historical financial facts were invented.
Warnings are existing datetime UTC deprecations and SQLAlchemy legacy Query.get.

Those failures and the SEC CLI wiring limitation were resolved by the subsequent
reproducibility cleanup described above. No old database was imported. The
original failed counts are retained here as historical context, not current
acceptance criteria.

The editable install command was inspected at checkpoint
`fbb4e7ea97cef8ab03d7beea363d0ef3f645ba7e`: it already read
`python -m pip install -e .`. The guide never instructed installation from the
parent directory; no install-command correction was necessary.

## Reproducibility cleanup verification (2026-09-20)

Validation used separate newly created PostgreSQL databases for the unit suite
and full suite, each migrated from base to `1f4b7c9d2a31` before its first test.
Only the test process's `DATABASE_URL` was overridden; private `.env` was unchanged.

- `python -m pytest -q tests/unit`: **81 passed, 0 failures, 8 warnings**.
- `python -m pytest -q`: **83 passed, 0 failures, 8 warnings**.
- `python -m pytest tests/unit/test_market_data_cache_freshness.py::test_freshness_triggers_fetch -q`:
  **1 passed** in isolation against a third empty migrated database.
- `python -m pytest tests/integration/test_nvda_features.py -q`: **2 passed**.
- Explicitly reordered freshness and integration tests: **4 passed**.
- After isolated/reordered tests, company, security, financial fact, feature
  registry, feature value and ingestion-run tables all still contained zero rows.

Only these disposable validation databases were removed afterwards. No previous
research database or raw data was restored, replaced or destroyed. No live SEC
HTTP requests were made. Calculation code and migrations remain unchanged.


## Rebuild machine plus research data after remediation

1. Clone the repository and check out the backup/build manifest's exact Git commit.
   Install the documented Python environment and dependencies; compare dependency
   versions to the ResearchBuildManifest rather than assuming an unconstrained
   installation recreates the old environment.
2. Configure PostgreSQL and private credentials, and install matching pg_dump and
   pg_restore clients (or use the documented local Docker tool mode).
3. Retrieve the complete DB + raw backup directory from controlled storage and run
   `python scripts/research_backup.py verify-files BACKUP_DIR`.
4. Follow [durability commands](DATA_DURABILITY_AND_RECOVERY.md) to restore into a
   newly generated database and an exclusive raw root. No existing DB is overwritten.
   The restore performs Alembic/current, row-count/content, raw-hash and selected
   historical-query checks. Set DATA_ROOT to the restored raw root for later research.
5. Compare restored Alembic revision with the backup manifest before any upgrade.
   Test `alembic upgrade head` on a separate disposable clone if moving to newer code.
6. Load the hash-named ResearchBuildManifest with `ResearchBuildManifest.read(path)`;
   verify code/schema, datasets/raw inventory, providers, normalization, feature and
   calculation versions, dependencies, configuration and research timestamp. Invoke
   its historical-replay qualification policy before claiming PIT reconstruction.
7. Run `python -m pytest -q` against a separate disposable test database. Some legacy
   tests delete/commit fixture rows; do not aim the full suite at recovered research.
   Run `IRS_PG_TOOLS_CONTAINER=irs_postgres python scripts/verify_foundation_recovery.py`
   for the local Docker migration/recovery drill. Native client installations omit
   the container variable.
8. Review the restore report and remaining provider/coverage limitations before
   manually choosing any production cutover. A fresh install with no backup can
   run synthetic tests, but cannot reconstruct missing historical raw data.

No off-host archive is configured by these commands. Existing local metadata with
missing raw files must be reconciled from trusted originals; passing a synthetic
fresh-host test does not certify persistent-data recovery.
