# Data Model

## Core Tables

### companies
- Primary entity. Links ticker, CIK, and corporate metadata.

### securities
- Handles different security types for a company (e.g., Common Stock, Preferred).

### filings
- Tracks every SEC filing. Primary key is `accession_number` + `company_id`.

### financial_facts
- Normalized XBRL facts. Tracks `concept`, `value`, `period_end`, and `filed_date`.

### market_prices
- Daily OHLCV data. Uniqueness: `security_id` + `date` + `source`.

### economic_series & economic_observations
- FRED economic data.

### raw_data_objects
- The audit trail for all external requests. Stores `content_hash` and `file_path`.

### ingestion_runs
- Metadata about every execution of the data pipeline.

### data_quality_events
- Logs validation failures and anomalies.

## Point-in-Time Strategy
To prevent look-ahead bias, we distinguish between:
- `period_end`: The end of the financial period the data describes.
- `filed_date`: When the data became public (the most critical date for backtesting).
- `retrieved_at`: When our system downloaded the data.
