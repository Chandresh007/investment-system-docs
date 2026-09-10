# System Architecture

## Overview
The Investment Research System is designed as a long-term investment research operating system. Phase 1 focuses on the **Free Data Pipeline**.

## Pipeline Flow
`DATA SOURCES` $\rightarrow$ `CONNECTORS` $\rightarrow$ `INGESTION ENGINE` $\rightarrow$ `RAW DATA STORAGE` $\rightarrow$ `VALIDATION` $\rightarrow$ `NORMALIZATION` $\rightarrow$ `PostgreSQL` / `Parquet` / `DuckDB`.

## Core Architectural Decisions
1. **Raw Data Immutability**: Every raw response is saved as a file and indexed in `raw_data_objects`.
2. **Provenance**: Every normalized record references its `raw_object_id`.
3. **Point-in-Time Correctness**: We track `filed_date`, `publication_date`, and `retrieved_at` separately to avoid look-ahead bias.
4. **Idempotency**: Ingestion jobs use natural keys and content hashes to prevent duplicates.
5. **Separation of Concerns**:
   - `Connectors`: Handle HTTP communication and rate limiting.
   - `Ingestion`: Orchestrate download, raw storage, and trigger normalization.
   - `Normalization`: Transform raw JSON/XML into structured models.
   - `Storage`: Manage PostgreSQL and Parquet persistence.

## Storage Strategy
- **PostgreSQL**: System of record for structured, normalized data.
- **Parquet/DuckDB**: Optimized for analytical queries and historical backtesting.
- **Local Filesystem**: Immutable store for raw API responses.
