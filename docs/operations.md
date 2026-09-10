# Operations Manual

## Setup
1. `docker compose up -d` - Start PostgreSQL.
2. `pip install -e .` - Install the system.
3. `alembic upgrade head` - Initialize database schema.

## Daily Pipeline
The primary entry point for data updates is:
`investment-research pipeline daily`

This command:
1. Checks `refresh_policy.yaml`.
2. Identifies stale datasets.
3. Fetches data via connectors.
4. Validates and normalizes.
5. Updates PostgreSQL.

## Manual Data Retrieval
To fetch data for a specific company:
`investment-research sec company --ticker NVDA`

To fetch economic data:
`investment-research fred-sync FEDFUNDS`

## Monitoring
- Check `ingestion_runs` table for job status.
- Check `data_quality_events` for validation errors.
- Logs are available via structured logging.
