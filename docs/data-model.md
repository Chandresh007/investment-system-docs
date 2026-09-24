# Data Model

## Core Tables

### companies
- Primary entity. Links ticker, CIK, and corporate metadata.

### securities
- Handles different security types for a company (e.g., Common Stock, Preferred).

### security_identity_history
- Bitemporal ticker, exchange, lifecycle, successor, and effective-interval evidence
  under a permanent internal `security_id`. Ticker is metadata, not identity.

### filings
- Tracks every SEC filing. Primary key is `accession_number` + `company_id`.

### financial_facts
- Normalized XBRL facts. Tracks `concept`, `value`, `period_end`, accession/filing
  provenance, exact or conservative `available_at`, and reporting fiscal context.

### market_prices
- Daily OHLCV data. Uniqueness: `security_id` + `date` + `source`.

### economic_series & economic_observations
- FRED economic data.

### raw_data_objects
- The audit trail for all external requests. Stores `content_hash` and `file_path`.
- `archive_kind` distinguishes original, reconstructed, and unverifiable legacy
  evidence. Reconstructed rows keep recovery time/source and the original row link.

### ingestion_runs
- Metadata about every execution of the data pipeline.

### data_quality_events
- Logs validation failures and anomalies.

## Point-in-Time Strategy
To prevent look-ahead bias, we distinguish between:
- `period_end`: The end of the financial period the data describes.
- `filed_date`: SEC date metadata; date-only values are not beginning-of-day knowledge.
- `available_at`: Exact evidenced acceptance when available, otherwise a documented
  conservative boundary such as next New York midnight.
- `retrieved_at`: When our system downloaded the data.
